# CPU plugin runtime parameters cache

## Table of Contents

- [Runtime cache application check list](#check_list)
- [Fused post ops handling](#post_ops)

## Runtime cache application check list<a name="check_list"></a>
1. Determine what data will be cached. We usually use the Executor concept that represents a junction of the executable code, usually JIT generated kernel, with some precomputed algorithm parameters.
2. Provide a key that uniquelly identifies the cached value as a funtion of dynamically changing parameters, i.e. shapes, dynamic input that determines the algorithm parameters, etc. To be used in a hash table, the key must have the following static interface:
   ```cpp
   struct KeyType {
       size_t hash() const;
       bool operator== () const;
   };
   ```
3. Provide a builder, that is, a callable object of the following signature: 
   ```cpp
   ValueType build(const KeyType& key);
   ```
   The `ValueType` is a type to be cached (e.g. shared pointer to Executor object). Remember that in the current cache implementation, a default constructed `ValueType()` object is considered empty, so it is better to use `std::shared_ptr` as the `ValueType`. The builder instantiates a specific type of cached entity from the `key`, thus the `key` completely defines the cached data. The builder is used to creat the `ValueType` object in case of cache miss.
4. Refactor the specific implementation of the `prepareParams()` method to extract the cached object construction logic (e.g. the algorithm parameters recalculation and JIT kernel generation) into the builder.
5. Add the key generation code into the `prepareParams()` method to query the cache.
6. Implement cache usage as the following:
   ```cpp
   void preapareParams() override {
        ... //code that prepares parameters for the key

        //key instantiation
        KeyType key = {param1, param2, ...};
        // get a reference to the cache
        auto cache = getRuntimeCache();
        //query cahce, buildExecutor is the builder descibed in 3
        auto result = cache->getOrCreate(key, buildExecutor); 
        // get the the cached value, in this example it is a pointer to an executor
        execPtr = result.first; 
   }
   ```
7. To provide smoke testing of these changes, add repeated shapes to the "target shapes" part of the corresponding single layer test definition:
    ```cpp
    { //dynamic case description each pair per each input has {{dynamic shape}, {{static shape case1}, {static shape case2}, ...}
        {{-1, -1, -1}, {{10, 10, 10}, {5, 5, 5}, {10, 10, 10}}}, // input 0
        {{-1, -1, 5}, {{10, 10, 5}, {5, 5, 5}, {10, 10, 5}}}  // input 1
    },
    ```

## Fused post ops handling<a name="post_ops"></a>
Post operations fusing mechanism in JIT kernels was designed bearing in mind static shapes and static data paradigm, which means that pointers to the working arrays of fused operations are part of the post operation description structure, which in turn is a part of the whole operation parameters list enclosed in the key. But obviously the data addresses themself are not a descripton of the operation and must be removed from the coressponding data structures in order to be passed through the rumtime parameters list of the JIT kernel. To address this problem, we need to update all the plugin JIT kernels simultaniously, that is a decent amount of work and will be done in the near future. Until then, we need to apply some workaround described below.

If the **quantization or depthwise** post ops are used through the post ops **injectors** mechanism, we need to update JIT kernels generation code in order to pass the scales and shifts data pointers at runtime:
   ```cpp
    size_t ptrs_table_off = quantization_post_op_idx * quantization_injectors[quantization_post_op_idx]->memoryStep();

    quantization_injectors[quantization_post_op_idx]->init_crop_ptrs(reg_post_op_ptrs + ptrs_table_off, reg_oc_off);

    quantization_injectors[quantization_post_op_idx]->init_input_scale_shift_ptrs(reg_post_op_ptrs + ptrs_table_off, reg_oc_off);

    quantization_injectors[quantization_post_op_idx]->init_output_scale_shift_ptrs(reg_post_op_ptrs + ptrs_table_off, reg_oc_off);

    quantization_post_op_idx++;
   ```
Since the post ops become a part of the key, we need to store their data pointers somwhere to use them at the execution stage, and zero them out inside the post ops to remove the data pointers from the key:
   ```cpp
   for (int i = 0; i < postOps.len(); ++i) {
        auto &data = postOps.get()->entry_[i].quantization.data;
        fqDataPtrs.insert(fqDataPtrs.end(), std::begin(data), std::end(data));
        memset(data, 0, sizeof(data));
    }
   ```