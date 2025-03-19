# Good Coding Practices afaik. Prioritised as follows:
1. Separate configuration (i.e. hyperparameters for ML models) from source code.
2. Guarantee reproducibility of experiments => Obvious note for exceptional cases where the experiments don't work as intended.
3. Fused code wherever possible. Compiler loves it. It's harder to read but you have chatGPT nowadays.
4. Nested loop less than 3 layers deep please
 
