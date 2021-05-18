Here is an overview of how to detect memory errors in OpenVINO using AddressSanitizer and LeakSanitizer tools.

Running OpenVINO tests and tools with sanitizers requires changing build options and environment variables.

# Supported platfroms

Linux
Mac OS

# Enabling AddressSanitizer and LeakSanitizer for OpenVINO

Use the ENABLE_SANITIZER flag while building OpenVINO to enable AddressSanitizer. LeakSanitizer is enabled automatically with AddressSanitizer.

cmake .. -DENABLE_SANITIZER=ON

Inside Intel, you can use pre-built "sanitizer" binaries from nightly artifacts.

# Running memory validation

Binaries compiled with -DENABLE_SANITIZER=ON report the runtime memory errors found to stdout. AddressSanitizer error messages start from the "AddressSanitizer" word.

AddressSanitizer and LeakSanitizer use ASAN_OPTIONS and LSAN_OPTIONS environment variables for their configuration.

Run OpenVINO tests and tools with the environment variables set to:
 
LSAN_OPTIONS=suppressions=<OpenVINO Repository>/tests/lsan/suppressions.txt NEOReadDebugKeys=1 DisableDeepBind=1

The default AddressSanitizer behavior is to halt execution on the first error found. To continue execution to see other errors add ASAN_OPTIONS=halt_on_error=0 environment variable. 

# Interpreting LeakSanitizer results

LeakSanitizer reports memory leaks to the stdout on the application exit.

LeakSanitizer distinguishes between directly leaked blocks (not reachable from anywhere) and indirectly leaked blocks (reachable from other leaked blocks). When fixing memory leaks start from "Direct leaks" first. Direct leaks can be a cause of indirect ones. If you have only indirect leaks those are the result of cyclic references.

# Known limitations

An old version of OpenVINO GPU plugin uses compute-runtime which is incompatible with AddressSanitizer and LeakSanitizer: https://github.com/intel/compute-runtime/issues/376
AddressSanitizer and LeakSanitizer may conflict with signal handlers such as libSegFault.so







 