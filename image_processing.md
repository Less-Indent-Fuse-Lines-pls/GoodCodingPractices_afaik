# Adapt toward torchvision v2 
- [Processing a batch of images before torchvision v2](https://github.com/bwconrad/vit-finetune/blob/main/src/data.py#L166-L189) is as follows
  1.
  2. Resize (or RandomResizedCrop) PIL images/numpy array.
  3. (Optional) Further augment images with extras RandomHorizontalFlip, TrivialAugmentWide, RandAugment
  4. Convert PIL images/numpy array into torch tensor.
  5. Normalise torch tensor
  6. Randomly erase some pixel in torch tensor.

- Since [torchvision v2, this workflow flips around](https://pytorch.org/vision/main/transforms.html#performance-considerations): Now you convert PIL images/numpy array into torch tensor because it is now quicker to preprocess image as torch tensor. Someone has done experiments [here](https://app.semanticdiff.com/gh/huggingface/transformers/pull/28847/overview). From comparing the source code between slow and fast, 
