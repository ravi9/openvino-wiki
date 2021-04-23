#  Guide for contributing to C++/C/Python IE samples

The Inference Engine sample applications are simple console applications that show how to utilize specific Inference Engine capabilities within an application, assist developers in executing specific tasks such as loading a model, running inference, querying specific device capabilities and etc.

**Main Goal of Sample:** Illustrate the primary, basic use case of the Inference Engine API

**Requirements:**
1. Each sample should represent the [main flow](https://docs.openvinotoolkit.org/latest/openvino_docs_IE_DG_Integrate_with_customer_application_new_API.html) to inference with OpenVINO(TM). Add commit blocks for each Integration step from the flow like in other samples.
2. Add explicit comments to your code if needed. It may help new users to understand your sample.
3. Create README.md for your sample with the following sections and following our [[Documentation guidelines|CodingStyleGuideLinesDocumentation]]:
   * Description - a short description about the sample, used features, and model type
      * Table of API
      * Table of common values, like (validated model, validated input data, supported devices, other language realization)
   * How It Works
   * Building
   * Running
   * Sample Output
   * See Also - section with useful links to proceed
4. **REQUIRED:** The README title must contain a document label in a form: `{#openvino_inference_engine_<bridge>_sample_<name>}`. For example: `Style Transfer Python* Sample {#openvino_inference_engine_ie_bridges_python_sample_style_transfer_sample_README}`.
   * Add your file to the documentation structure. Open the documentation structure file [`docs/doxygen/openvino_docs.xml`] and add your file path and a sample name to the "IE Code Samples" section.
   * Add link to your sample to docs/IE_DG/Samples_Overview.md 
5. Please follow repo code style, we have linter checks for each PR.
6. Please use the basic `hello` sample as a template to create your sample.
7. One sample - one type of model and no more than 2 API special features.
8. Each API feature should be documented with code snippets on our [OpenVINO Documentation site](https://docs.openvinotoolkit.org/)
9. Prefer sample development with a one-source approach, without explicit pre/post-processing steps if it is possible.
10. Prefer C++/Python realizations for each sample (C is optional).
11. Set name for your sample in the format: "`model_type` `feature` `lang` Sample"
     - `feature` is optional in case your sample represents a new model.

Please take it in your mind before creating your sample.

 