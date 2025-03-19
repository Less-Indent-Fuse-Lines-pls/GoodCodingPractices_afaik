# Good Coding Practices afaik. Prioritised as follows:
1. Guarantee reproducibility of experiments => Obvious note for exceptional cases where the experiments don't work as intended.
2. Separate configuration (i.e. hyperparameters for ML models) from source code.
3. No overbloated configuration/source code. Trim down the scope of the repo. Think of model zoos, if a newly added model vastly differs from existing ones in the model zoos, lots of "if-else" have to be added to address for it then eventually you end up having 2k or 4k lines of code in main.py. All of these can be avoided by creating a new repo. 
4. Fused code wherever possible. Compiler loves it. It's harder to read but you have chatGPT nowadays.
5. Nested loop less than 3 layers deep please
 
