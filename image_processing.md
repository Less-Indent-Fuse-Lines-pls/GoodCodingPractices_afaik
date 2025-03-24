# List of coding practice that speedups image processing workflow:
- [ ] Use Dataset.map(batched=True). See performance boost [here](https://huggingface.co/learn/nlp-course/en/chapter5/3?fw=pt#the-map-methods-superpowers) (0scroll down to see table). But you need to use num_proc and set batched=True in map(), otherwise it will be slower than default set_transform() [as reported here](https://discuss.huggingface.co/t/using-map-take-7-2x-times-longer-than-set-transform/62285).
- [ ] Modify your transforms.Compose to use torchvision v2 instead of torchvision v1.
  
# Why adapt toward torchvision v2 
- From 2 most popular ViT implementations: [HuggingFace's implementation](https://github.com/huggingface/transformers/blob/v4.49.0/src/transformers/models/vit/image_processing_vit.py#L152-L283) and [Timm's implementation](https://github.com/huggingface/transformers/blob/main/examples/pytorch/image-classification/run_image_classification.py#L337-L362), image preprocessing workflow before torchvision v2 is as follows:
  1. Resize/RandomResizedCrop PIL images/numpy array.
  2. RandomHorizontalFlip for Timm's implementation; Rescale for Huggingface's implementation.
  3. Convert PIL image to torch tensor then rescale and normalise for Timm's implementation; rescale and normalise numpy array then convert to torch tensor for HuggingFace's implementation.
    
- Since torchvision v2, preprocessing tensor is much quicker than PIL images as stated [in torchvision documentation](https://pytorch.org/vision/main/transforms.html#performance-considerations). But not PIL images but also quicker than numpy array as someone had experimented [here](https://app.semanticdiff.com/gh/huggingface/transformers/pull/28847/overview). Comparing [torchvision v2's implementation](https://github.com/huggingface/transformers/blob/main/src/transformers/image_processing_utils_fast.py#L668-743) to either implementations above, the workflow has changed into:
  1. Convert input batch into tensor
  2. Do everything above with either torch.transforms.v2 or torch.transforms.v2.functionals
