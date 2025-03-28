# Good Coding Practices afaik. Prioritised as follows:
1. Guarantee reproducibility of experiments => Obvious note for exceptional cases where the experiments don't work as intended.
2. Separate configuration (i.e. hyperparameters for ML models) from source code.
3. No overbloat source code or config. Trim down the scope of the repo. Imo, each repo should only 1 backbone, 1 platform (Pytorch, Tensorflow or Jax), 1 image processor. At first, it will be redundant to the original authors having to rewrite some bits in backbone definition whenever there's only a slight chance in the backbone; but think from other researchers and engineers perspectives. Let's say someone has a new idea to something small like random_masking function before PatchEmbedding, instead of just copy+paste class ViT backbone, image processor for ViT, dataloader, loss, they now have to look at +2000 lines of code in HuggingFace transformer or Timm transformer just to navigate to where the code they want to copy+paste.
4. Nested loop less than 3 layers deep please.
 
