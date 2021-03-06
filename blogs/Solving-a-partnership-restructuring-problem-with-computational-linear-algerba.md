---
title: "Solving a partnership restructuring problem with computational linear algebra."
date: 2021-04-20T21:22:42-07:00
---

If you’d like to skip straight to the code, jump ahead to the final section.

Today's blog post comes out of a business analytics problem. Lets consider a partnership that is restructuring its assets (with N partners and M assets). The partners currently share the business, for example, partner N1 owns 33% of the company, partner N2 owns 20%, and so on. Each of the M assets carries a current value and a debt owed. Our task is to re-distribute the ownerships of the assets such that each is owned outright by one of the partners. Critically, we need to divvy the assets up without altering the partners' ownership shares. In effect, we want to split up the partnership in a fair manner, preserving each partner's stake in the business.

Linear algebra - to the rescue! First, create an NxM probability matrix encoding the ownership percentages (for example, partner N1 gets 100% of asset M1, 0% of asset M2, etc.). Multiply the NxM probability matrix with the Mx2 matrix encoding the assets' values and the debts. Subtract the result from the current ownership shares (a Nx2 matrix) (making sure that the entries match up to the correct partner). Finally, take the absolute values and sum the entries. This final number encodes the cumulative differences between the restructured ownership stakes and the partners' original ones. We then shuffle the columns up, re-run the pipeline, and record the new cumulative difference. Do this for all the different permutations of the column order and voila, find out whether or not your restructured partnership gets close to the original one. In essence, find the global minimum of all the different permutations - > back out which asset went to which partner - > assess whether or not this is close enough to the original partnership.

For example, consider 6 assets shared among 3 partners. The first matrix encodes the probability matrix (which we will end up shuffling columns for). The second encodes the current value (col1) and debt owed (col2) per asset. The third matrix encodes the three partners' original ownership of both the assets' value (col1) and debt owed (col2).

<p align="center"><img src="https://render.githubusercontent.com/render/math?math=\begin{vmatrix} 1 & 1 & 0 & 0 & 0 & 0\\ 0 & 0 & 1 & 1 & 0 & 0\\ 0 & 0 & 0 & 0 & 1 & 1 \end{vmatrix} * begin{vmatrix} a & b \\ c & d \\ e & f \\ g & h \\ i & j \\k & l \end{vmatrix} - begin{vmatrix} m & n \\ o & p \\ q & r  \end{vmatrix}"> </p>

## How good can this search algorithm do?

And after all that, our result many turn out to be, well, pretty bad. Consider a simplified case: 2 partners own 3 assets shared evenly, but the assets are all equivalent, and, therefore, there is no way to split up the majority ownerships.

Another unattractive feature of our methodology was that we preemptively chose how many assets in total went to each partner, a step in the analytics pipeline that requires experimenting with. For example, in the matrix multiplications above, we decided that each partner got two properties. Some intuition must go into this initial decision after examining the original ownership stakes and the value/debts of the assets. This approach forces you to test all of the reasonable arrangements, which is only feasible for smaller order problems.

## When does this approach become unwieldy?

The approach is certainly a "brute force" method. The for loop through the permutations is O(n) time complexity - but we need to loop across M! permutations. Runtimes can grow with large N, see below:



## Code implementation

```python
### Libraries :
import numpy as np
from sympy.utilities.iterables import multiset_permutations

### Data I/O :
assets_value_debt = np.genfromtext(...) #asset value/debt df
restructure_probabilities = np.genfromtext(...) #initial state df
previous_stake = np.genfromtext(...) #previous asset value/debt df
num_assets = len(assets_value_debt[:,0]) #num of assets to restructure, n
n_factorial = np.math.factorial(num_assets) #num of scenarios, n!

### Search algorithm :
permutations = np.zeros((n_factorial, num_assets)) #initialize df
index = np.arange(0, num_assets) #index that stands for each asset
iter = 0
for p in multiset_permutations(index):
    permutations[iter,:] = p #sympy solves for all permutations
    iter+=1
permutations = permutations.astype(int) #make them ints (originally floats)

results = np.zeros(n_factorial) #initialize df
for iter in np.arange(0, n_factorial):
  # first grab the restructure state by reordering the features
  probabilities_looped = restructure_probabilities[:, permutations[iter,:]]
  # second solve the matrix multiplication step
  results_lopped = np.matmul(probabilities_looped, assets_value_debt)
  # lastly compute and sum the difference in before and after restructuring
  final_difference = np.abs(results_lopped - previous_stake)
  results[iter] = np.sum(final_difference)
np.min(results) #the minimum difference
np.argmin(results) #index of the best probability matrix
```

Thanks for reading! Excited to write more about my analytics projects in the future.
