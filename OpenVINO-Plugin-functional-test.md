# Plugin specific functional Test

## Description
Functional test is one of OpenVINO testing framework. It is used to verify public Inference Engine API. You can find high level description from https://github.com/openvinotoolkit/openvino/wiki/InferenceEngineTestsInfrastructure. The plugin specific functional tests are dependent on the corresponding plugin. The functional tests can be categorized to the mainly 5 types of tests as below.
* Behavior test: Test basic behavior of plugin related IE API
* Execution graph info: Test graph handling features of plugin
* Low precision transformation: Test low precision transformation(int8 quantization) features in plugin
* Single layer test: Test whether inference result of plugin about single op layer is same with result from ngraph reference function or not.
* Subgraph test: Test inference result of plugin about combinations of single op layer


## Source folder structure
The source codes needed for the functional test can be found under {OPENVINO_ROOT}/src/tests/functional folder except Template plugin(It is under {OPENVINO_ROOT}/doc/template_plugin/tests/functional).

> ```
> src/tests/
> └── functional
>     ├── plugin
>     │   ├── CMakeLists.txt
>     │   ├── conformance
>     │   ├── cpu
>     │   ├── gna
>     │   ├── gpu
>     │   ├── myriad
>     │   └── shared
>     └── shared_test_classes
>         ├── CMakeLists.txt
>         ├── include
>         └── src
> ```

* `shared_test_classes` folder has common layer test utilities, test classes of single layer and subgraph for all plugin tests. For example, `src/tests/functional/shared_test_classes/src/base/layer_test_utils.cpp` contains actual execution body(`LayerTestsCommon::Run()`) for single layer test under functional test.
> ```
> src/tests/functional/shared_test_classes
> ├── CMakeLists.txt
> ├── include
> │   └── shared_test_classes
> └── src
>     ├── base
>     ├── precomp.hpp
>     ├── report
>     ├── single_layer
>     └── subgraph
> ```
* `shared` folder has gtest main, test definitions and classes of plugin functional test for all plugins except `single_layer_tests` and `subgraph_test`. Only test definition of both `single_layer_tests` and `subgraph_test` are located here and test classes are in `shared_test_classes`.
> ```
> src/tests/functional/plugin/
> └── shared
>     ├── CMakeLists.txt
>     ├── include
>     │   ├── base
>     │   ├── behavior
>     │   ├── execution_graph_tests
>     │   ├── low_precision_transformations
>     │   ├── multi
>     │   ├── onnx
>     │   ├── single_layer_tests
>     │   ├── snippets
>     │   └── subgraph_tests
>     ├── models
>     └── src
>         ├── behavior
>         ├── execution_graph_tests
>         ├── low_precision_transformations
>         ├── main.cpp
>         ├── onnx
>         ├── precomp.hpp
>         ├── single_layer_tests
>         └── snippets
> ```

> ```
> // Example of test class
> namespace BehaviorTestsDefinitions {
> class ExecutableNetworkBaseTest : public testing::WithParamInterface<BehaviorTestsUtils::InferRequestParams>,
>                                   public CommonTestUtils::TestsCommon {
> public:
>     static std::string getTestCaseName(testing::TestParamInfo<BehaviorTestsUtils::InferRequestParams> obj) {
>         std::string targetDevice;
>         std::map<std::string, std::string> configuration;
>         std::tie(targetDevice, configuration) = obj.param;
>         std::ostringstream result;
>         result << "targetDevice=" << targetDevice << "_";
>         if (!configuration.empty()) {
>             using namespace CommonTestUtils;
>             result << "config=" << configuration;
>         }
>         return result.str();
>     }
> ......
> ```

> ```
> // Example of test definition
> TEST_P(ExecutableNetworkBaseTest, canLoadCorrectNetworkToGetExecutable) {
>     ASSERT_NO_THROW(auto execNet = ie->LoadNetwork(cnnNet, targetDevice, configuration));
> }
> ```

`gpu` folder has instances of common functional tests, single layer and subgraph tests.

> ```
> src/tests/functional/plugin/
> └── gpu
>     ├── behavior
>     ├── CMakeLists.txt
>     ├── concurrency
>     ├── dynamic_tests
>     ├── remote_blob_tests
>     ├── shared_tests_instances
>     ├── single_layer_tests
>     └── subgraph_tests
> ```

* It is test case instances of `shared_test_classes` for GPU plugin test. If you want to add more test cases of some op single layer test using `shared_test_classes`, you can add them here.
> ```
> src/tests/functional/plugin/gpu/shared_tests_instances
> ├── behavior
> ├── core_config.cpp
> ├── execution_graph_info
> ├── low_precision_transformations
> ├── multi
> ├── single_layer_tests
> ├── skip_tests_config.cpp
> └── subgraph_tests
> ```

> ```
> // Example of test case instance
> INSTANTIATE_TEST_SUITE_P(smoke_ReshapeCheck, ReshapeLayerTest,
>         ::testing::Combine(
>                 ::testing::Values(true),
>                 ::testing::ValuesIn(netPrecisions),
>                 ::testing::Values(InferenceEngine::Precision::UNSPECIFIED),
>                 ::testing::Values(InferenceEngine::Precision::UNSPECIFIED),
>                 ::testing::Values(InferenceEngine::Layout::ANY),
>                 ::testing::Values(InferenceEngine::Layout::ANY),
>                 ::testing::Values(std::vector<size_t>({10, 10, 10, 10})),
>                 ::testing::Values(std::vector<int64_t>({10, 0, 100})),
>                 ::testing::Values(CommonTestUtils::DEVICE_GPU),
>                 ::testing::Values(std::map<std::string, std::string>({}))),
>                 ReshapeLayerTest::getTestCaseName);
> ```

* `single_layer_tests` in specific plugin folder has plugin specific tests. If test case instantiated from `shared_test_classes` doesn't work or have enough coverage on specific plugin, the specific plugin has own single layer test classes in separated folder under plugin test. There are 3 single layer tests in the folder of OpenVINO 2022.2 branch. They have layer test class and instance together in the file. `convolution.cpp` has additional GPU convolution tests which provide more test coverage for GPU plugin in addition to convolution tests from `shared_test_classes`. `loop.cpp` and `tensor_iterator.cpp` are used instead of tests from `shared_test_classes`.
> ```
> src/tests/functional/plugin/gpu/single_layer_tests
> ├── convolution.cpp
> ├── loop.cpp
> └── tensor_iterator.cpp
> ```
* Conformance test executes single operation/subgraph tests(IR files) which are parsed from real models (OMZ, DL Benchmark scopes) and reports coverage of op on specific plugin. For more details, please refer https://github.com/openvinotoolkit/openvino/wiki/Operation-Conformance-tests
> ```
> src/tests/functional/plugin/conformance
> ```

## How to run
Functional test is using google gtest and you can use running options from gtest.
* Full test
> ```
> $./gpuFuncTests
> ```
* Test case filtering with word
> ```
> $./gpuFuncTests --gtest_filter=*smoke*
> ```
* Test case filtering with test suite name
> ```
> $./gpuFuncTests --gtest_filter=*smoke_Convolution2D_ExplicitPadding*
> ```
* Test case filtering with test class name
> ```
> $./gpuFuncTests --gtest_filter=*ConvolutionLayerTest*
> ```
* Test case filtering with exclusion. Minus(-) option means excluding the test cases of following keyword.
> ```
> $./gpuFuncTests --gtest_filter=*smoke*:-*smoke_Convolution2D_ExplicitPadding*
> ```
* Test result output to file
> ```
> $./gpuFuncTests --gtest_filter=*smoke* --gtest_output=xml:TEST-gpuFuncTests.xml
> ```


## How to add a new single layer test
When you implement new op support in the plugin, you need to a single layer test for the new op. Generally all operations supported by OpenVINO have test class in `src/tests/functional/shared_test_classes/include/shared_test_classes/single_layer`. You can use it or create new test class for specific plutin.

* `shared_test_classes/include/single_layer_tests/[op_name].hpp` defines the common test fixture classes for the operations under test, which can be shared by all plugins. 
* `plugin/shared/include/single_layer_tests/[op_name].hpp` defines the parameterized test cases which can be shared by all plugins. 
* `plugin/[plugin_name]/shared_tests_instances/single_layer_tests/[op_name].cpp` declares the plugin specific test case instances with actual parameter values such as input shapes and required attributes.


1. When you want to use test class from `shared_test_classes`, only test instance for new op under `src/tests/functional/plugin/gpu/shared_tests_instances/single_layer_tests` is required with file name of `_op_name_.cpp`.

> ```
> // Example of test instance - src/tests/functional/plugin/gpu/shared_tests_instances/single_layer_tests/concat.cpp
> #include <vector>
>
> #include "single_layer_tests/concat.hpp"
> #include "common_test_utils/test_constants.hpp"
>
> using namespace LayerTestsDefinitions;
>
> namespace {
>
> std::vector<int> axes = {-3, -2, -1, 0, 1, 2, 3};
> std::vector<std::vector<std::vector<size_t>>> inShapes = {
>         {{10, 10, 10, 10}},
>         {{10, 10, 10, 10}, {10, 10, 10, 10}},
>         {{10, 10, 10, 10}, {10, 10, 10, 10}, {10, 10, 10, 10}},
>         {{10, 10, 10, 10}, {10, 10, 10, 10}, {10, 10, 10, 10}, {10, 10, 10, 10}},
>         {{10, 10, 10, 10}, {10, 10, 10, 10}, {10, 10, 10, 10}, {10, 10, 10, 10}, {10, 10, 10, 10}}
> };
> std::vector<InferenceEngine::Precision> netPrecisions = {InferenceEngine::Precision::FP32,
>                                                          InferenceEngine::Precision::FP16,
>                                                          InferenceEngine::Precision::I64};
>
>
> INSTANTIATE_TEST_SUITE_P(smoke_NoReshape, ConcatLayerTest,
>                         ::testing::Combine(
>                                 ::testing::ValuesIn(axes),
>                                 ::testing::ValuesIn(inShapes),
>                                 ::testing::ValuesIn(netPrecisions),
>                                 ::testing::Values(InferenceEngine::Precision::UNSPECIFIED),
>                                 ::testing::Values(InferenceEngine::Precision::UNSPECIFIED),
>                                 ::testing::Values(InferenceEngine::Layout::ANY),
>                                 ::testing::Values(InferenceEngine::Layout::ANY),
>                                 ::testing::Values(CommonTestUtils::DEVICE_GPU)),
>                         ConcatLayerTest::getTestCaseName);
> }  // namespace
> ```

2. When you want to create new test class and definition for the plugin, you need to add new file with file name of `_op_name_.cpp` under `src/tests/functional/plugin/gpu/single_layer_tests`. It should have test definition, instance and class all together.

> ```
> // Example of plugin specific single layer test - src/tests/functional/plugin/gpu/single_layer_tests/convolution.cpp
> // using namespace LayerTestsDefinitions;
> using namespace InferenceEngine;
> using namespace ov::test;
>
> namespace GPULayerTestsDefinitions {
> using LayerTestsDefinitions::convSpecificParams;
>
> typedef std::tuple<
>         convSpecificParams,
>         ElementType,     // Net precision
>         ElementType,     // Input precision
>         ElementType,     // Output precision
>         InputShape,      // Input shape
>         LayerTestsUtils::TargetDevice   // Device name
> > convLayerTestParamsSet;
>
> class ConvolutionLayerGPUTest : public testing::WithParamInterface<convLayerTestParamsSet>,
>                              virtual public SubgraphBaseTest {
> public:
>     static std::string getTestCaseName(const testing::TestParamInfo<convLayerTestParamsSet>& obj) {
>         convSpecificParams convParams;
>         ElementType netType;
>         ElementType inType, outType;
>         InputShape inputShape;
>         std::string targetDevice;
>         std::tie(convParams, netType, inType, outType, inputShape, targetDevice) = obj.param;
>
> ......
>
>         return result.str();
>     }
>
> protected:
>     void SetUp() override {
>         convSpecificParams convParams;
>         InputShape inputShape;
>         auto netType = ElementType::undefined;
>         std::tie(convParams, netType, inType, outType, inputShape, targetDevice) = this->GetParam();
>
> ......
>
>         auto inputParams = ngraph::builder::makeDynamicParams(inType, inputDynamicShapes);
>         auto paramOuts = ngraph::helpers::convert2OutputVector(ngraph::helpers::castOps2Nodes<ngraph::op::Parameter>(inputParams));
>
>         auto convolutionNode = ngraph::builder::makeConvolution(paramOuts.front(), netType, kernel, stride, padBegin,
>                                                                 padEnd, dilation, padType, convOutChannels);
>
>         ngraph::ResultVector results;
>         for (int i = 0; i < convolutionNode->get_output_size(); i++)
>                 results.push_back(std::make_shared<ngraph::opset1::Result>(convolutionNode->output(i)));
>
>         function = std::make_shared<ngraph::Function>(results, inputParams, "Convolution");
>     }
> };
>
> TEST_P(ConvolutionLayerGPUTest, CompareWithRefs) {
>     SKIP_IF_CURRENT_TEST_IS_DISABLED()
>     run();
> }
>
> namespace {
> // Check 3D input tensor for convolution is handled properly and its output is correct comparing with ngraph runtime.
> INSTANTIATE_TEST_SUITE_P(smoke_ConvolutionLayerGPUTest_3D_tensor_basic, ConvolutionLayerGPUTest,
>         ::testing::Combine(
>                 ::testing::Combine(
>                         ::testing::Values(SizeVector{3}),
>                         ::testing::Values(SizeVector{1}),
>                         ::testing::Values(std::vector<ptrdiff_t>{0}),
>                         ::testing::Values(std::vector<ptrdiff_t>{0}),
>                         ::testing::Values(SizeVector{1}),
>                         ::testing::Values(13),
>                         ::testing::Values(ngraph::op::PadType::SAME_UPPER)),
>                 ::testing::Values(ElementType::f16),
>                 ::testing::Values(ElementType::f16),
>                 ::testing::Values(ElementType::undefined),
>                 ::testing::Values(InputShape{{}, {{1, 13, 30}}}),
>                 ::testing::Values<std::string>(CommonTestUtils::DEVICE_GPU)),
>                 ConvolutionLayerGPUTest::getTestCaseName);
> }  // namespace
>
> } // namespace GPULayerTestsDefinitions
> ```

3. Single layer test has mainly 4 steps - `LoadNetwork()`, `GenerateInputs()`, `Infer()` and `Validate()`.

> ```
> // src/tests/functional/shared_test_classes/src/base/layer_test_utils.cpp
> void LayerTestsCommon::Run() {
> ......
>     if (jmpRes == CommonTestUtils::JMP_STATUS::ok) {
>         crashHandler->StartTimer();
>         try {
>             LoadNetwork();
>             GenerateInputs();
>             Infer();
>             Validate();
>             s.updateOPsStats(functionRefs, PassRate::Statuses::PASSED);
>         }
>         catch (const std::runtime_error &re) {
>             s.updateOPsStats(functionRefs, PassRate::Statuses::FAILED);
>             GTEST_FATAL_FAILURE_(re.what());
> ......
> ```

* Single layer test is using ngraph function if target operation. It is defined in the test class and loaded to plugin in `LoadNetwork()`.
* Input tensor for the single layer test is generated randomly with op corresponding condition in `GenerateInputs()`.
* The test passes when inferred result from plugin should be almost same with reference value under given threshold. It executes in `Validate()`. Reference value is calculated by ngraph op reference code(`src/core/reference/include/ngraph/runtime/reference`). It is usually written with generic c++ math functions to get the result or combination of other ngraph op reference code.
* There are two threshold - relative threshold and absolute threshold. Relative threshold is about relative difference between inferred result and reference value. Default value is 0.02(2%). Absolute threshold is about absolute difference. It is default turned off. Those threshold could be configured per each test class.
>

4. Each test case could be listed in skip test config. For example, specific test case could fail until new feature implementation. Then we can add it into the skip test config and make overall test performed without failure. Rule of filtering test case is similar with gtest_filter and you need to add `.` before `*` to avoid ignoring the `*` in parsing.
> src/tests/functional/plugin/gpu/shared_tests_instances/skip_tests_config.cpp

> ```
> #include <vector>
> #include <string>
>
> #include "functional_test_utils/skip_tests_config.hpp"
>
> std::vector<std::string> disabledTestPatterns() {
>     return {
>             //TODO: Issue: 34748
>             R"(.*(ComparisonLayerTest).*)",
>             // TODO: Issue: 39612
>             R"(.*Interpolate.*cubic.*tf_half_pixel_for_nn.*FP16.*)",
>             // Expected behavior
>             R"(.*EltwiseLayerTest.*eltwiseOpType=Pow.*netPRC=I64.*)",
>             R"(.*EltwiseLayerTest.*IS=\(.*\..*\..*\..*\..*\).*eltwiseOpType=Pow.*secondaryInputType=CONSTANT.*)",
>             // TODO: Issue: 43794
>             R"(.*(PreprocessTest).*(SetScalePreProcessSetBlob).*)",
>             R"(.*(PreprocessTest).*(SetScalePreProcessGetBlob).*)",
>             R"(.*(PreprocessTest).*(SetMeanValuePreProcessSetBlob).*)",
>             R"(.*(PreprocessTest).*(SetMeanImagePreProcessSetBlob).*)",
>             R"(.*(PreprocessTest).*(ReverseInputChannelsPreProcessGetBlob).*)",
>             R"(.*(InferRequestPreprocessDynamicallyInSetBlobTest).*)",
> ```
