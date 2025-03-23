# Adapt toward torchvision v2 
- Based on two most popular Vision Transformer implementations: [HuggingFace's implementation](https://github.com/huggingface/transformers/blob/v4.49.0/src/transformers/models/vit/image_processing_vit.py#L152-L283) and [Timm's implementation](https://github.com/huggingface/transformers/blob/main/examples/pytorch/image-classification/run_image_classification.py#L333-362), image preprocessing workflow before torchvision v2 is as follows:
  1. Convert input batch into numpy array (or keep it as PIL image for Timm's implementation)
  2. Resize (or RandomResizedCrop).
  3. RandomHorizontalFlip (only for Timm's implementation)
  4. Further augment images with TrivialAugmentWide, RandAugment (only for Timm's implementation)
  5. Convert to torch tensor then rescale and normalise (Timm's implementation); or rescale and normalise then convert to torch tensor (HuggingFace's implementation)
    
- Since torchvision v2, preprocessing tensor is much quicker than PIL images as stated [in torchvision documentation](https://pytorch.org/vision/main/transforms.html#performance-considerations). But not PIL images but also quicker than numpy array as someone had experimented [here](https://app.semanticdiff.com/gh/huggingface/transformers/pull/28847/overview). Comparing [torchvision v2's implementation](https://github.com/huggingface/transformers/blob/main/src/transformers/image_processing_utils_fast.py#L668-743) to either implementations above, the workflow has changed into:
  1. Convert input batch into tensor
  2. Do everything above with either torch.transforms.v2 or torch.transforms.v2.functionals
