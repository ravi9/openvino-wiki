#  Guide for contributing to C++/C/Python IE samples

The Inference Engine sample applications are simple console applications that show how to utilize specific Inference Engine capabilities within an application, assist developers in executing specific tasks such as loading a model, running inference, querying specific device capabilities and etc.

**Main Goal of Sample:** Illustrate the primary, basic use case of the Inference Engine API

**Requirements:**
1. Each sample should represent the [main flow](https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_Integrate_with_customer_application_new_API.html) to inference with OpenVINO(TM). Add commit blocks for each Integration step from the flow like in other samples.
2. Add explicit comments to your code if needed. It may help new users to understand your sample.
3. Create README.md for your sample with the following sections:
   * Description - a short description about the sample, used features, and model type
      * Table of API
      * Table of common values, like (validated model, validated input data, supported devices, other language realization)
   * How It Works
   * Building
   * Running
   * Sample Output
   * See Also - section with useful links to proceed
4. Please follow repo code style, we have linter checks for each PR.
5. Please look at the basic `hello` samples.
6. One sample - one type of model and no more than 2 API special features.
7. Each API feature should be documented with code snippets on our [OpenVINO Documentation site](https://docs.openvinotoolkit.org/)
8. Sample should be developed with a one-source approach, without explicit pre/post-processing steps.
9. Prefer C++/Python realizations for each sample (C is optional).
10. Use name for sample in format: <model_type> <feature> <lang> Sample.

Please take it in your mind before creating your sample.

 