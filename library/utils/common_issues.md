
# torch
1. RuntimeError: CUDA error: device-side assert triggered CUDA kernel errors might be asynchronously reported at some other API call,so the stacktrace below might be incorrect. For debugging consider passing CUDA_LAUNCH_BLOCKING=1.
   check num_classes in data and model, see this stack overflow [thread](https://discuss.pytorch.org/t/runtimeerror-cuda-error-device-side-assert-triggered-cuda-kernel-errors-might-be-asynchronously-reported-at-some-other-api-call-so-the-stacktrace-below-might-be-incorrect/139735)