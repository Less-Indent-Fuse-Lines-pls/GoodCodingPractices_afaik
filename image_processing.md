# List of coding practice that speedups image processing workflow:
- [ ] Use Dataset.map(batched=True). See performance boost [here](https://huggingface.co/learn/nlp-course/en/chapter5/3?fw=pt#the-map-methods-superpowers) (0scroll down to see table). But you need to use num_proc and set batched=True in map(), otherwise it will be slower than default set_transform() [as reported here](https://discuss.huggingface.co/t/using-map-take-7-2x-times-longer-than-set-transform/62285).
- [ ] Modify your transforms.Compose to use torchvision v2 instead of torchvision v1. In this post, "torchvision v2" = ViTImageProcessorFast on HuggingFace, "torchvision v1" = ViTImageProcessor on HuggingFace. 
- [ ] Use TorchAug with num_chunks=1 to deal with any random data augmentation (like RandomResizedCrop) where you are forced to loop through each image in a batch. Based on [TorchAug speed benchmark](https://github.com/juliendenize/torchaug/blob/main/docs/source/include/speed_comparison.md) and given that([HuggingFace is still looping through each sample in a batch](https://github.com/huggingface/transformers/blob/main/src/transformers/image_processing_utils_fast.py#L721-L738), you can already gain 3.58x speedup over HuggingFace's ViTImageProcessingFast just for using TorchAug's RandomResizedCrop alone. A little sidenote, kornia also claims to have speedup like TorchAug, [which is entirely false claim](https://github.com/kornia/kornia/issues/1559)
  
# Why adapt torchvision v2 
From my personal benchmark of a typical image preprocessing workflow for vit-base-patch16-224, I notice 2 things:
1. While data augmentation functions are still more efficient to use them on PIL image than torch tensor, 0.2ms speedup per image.
2. Most significant speedup is whenever you can employ v2.functional, >0.4ms speedup per image.
If these number looks small, scale it up by batch_size * num_training_iter * epochs like say 0.2 * 256 * (dataset_size/256) * 200. As your dataset size grows to the around millions as you want to train a new foundation model or finetune foundation model in a completely new domain. Yea, you easily looking at 33 GPU hours saved.

preprocessing tensor has been quicker than PIL images as stated [in torchvision documentation](https://pytorch.org/vision/main/transforms.html#performance-considerations). But not PIL images but also quicker than numpy array as someone had experimented [here](https://app.semanticdiff.com/gh/huggingface/transformers/pull/28847/overview). Comparing [torchvision v2's implementation](https://github.com/huggingface/transformers/blob/main/src/transformers/image_processing_utils_fast.py#L668-L743) to either implementations above, the workflow has changed into:
  1. Convert input batch into tensor
  2. Do everything above with either torch.transforms.v2 or torch.transforms.v2.functionals

From 2 most popular ViT implementations: [HuggingFace's implementation](https://github.com/huggingface/transformers/blob/v4.49.0/src/transformers/models/vit/image_processing_vit.py#L152-L283) and [Timm's implementation](https://github.com/huggingface/transformers/blob/main/examples/pytorch/image-classification/run_image_classification.py#L337-L362), image processing workflow before torchvision v2 is as follows:
  1. Resize/RandomResizedCrop PIL images/numpy array.
  2. RandomHorizontalFlip for Timm's implementation; Rescale for Huggingface's implementation.
  3. Convert PIL image to torch tensor then rescale and normalise for Timm's implementation; rescale and normalise numpy array then convert to torch tensor for HuggingFace's implementation. <br>
    
Since torchvision v2, processing image as tensor has been quicker than PIL images as stated [in torchvision documentation](https://pytorch.org/vision/main/transforms.html#performance-considerations). But not PIL images but also quicker than numpy array as someone had experimented [here](https://app.semanticdiff.com/gh/huggingface/transformers/pull/28847/overview). Comparing [torchvision v2's implementation](https://github.com/huggingface/transformers/blob/main/src/transformers/image_processing_utils_fast.py#L668-L743) to either implementations above, the workflow has changed into:
  1. Convert input batch into tensor
  2. Do everything above with either torch.transforms.v2 or torch.transforms.v2.functionals

# Reality-check
- When HuggingFace adopt ViTImageProcessorFast and deprecate ViTImageProcessor, some state-of-the-art models stubbornly stays ViTImageProcessor (see [here](https://github.com/huggingface/transformers/issues/36193)). Considering how moving from "torchvision v1" -> "torchvision v2" is mostly changing some lines of code like say "transforms.Normalise" -> "v2.functional.normalise". Imo, this is the symptom of overbloated code.
