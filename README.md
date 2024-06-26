# Novel Algorithms

## Training a Neural Network in constant time for an arbitrary amount of training data
(this seems to be a novel algorithm as indicated in the comments [here](https://ai.stackexchange.com/questions/46122/downsides-of-training-a-neural-network-in-constant-time))

- Instead of the normal pipeline (collect all training data, train a model with all of the training data) that will take weeks or months to train for big data,
  we collect the training data into a database like a graph database or into a file database like LMDB.
- We do _not_ train the model on this large training data.
- Now, in the inference phase, for example when translating a sentence from one language to an other, we retrieve a constant relevant amount of training data from the huge database.
- After this, train a model using slow state-of-the-art solutions and the retrieved data. It is also possible to retrieve another constant amount of general data not related to the input, to better learn the general concept.
- Now, do the inference on this trained model.
- We now have a very fast first training phase because we only have to store the training data into the database.
- The inference phase will be much slower because we have to train a model each time, but the inference time will still be acceptable (10 seconds to 10 minutes, unclear) due to the constant amount limitation (similiar to Insertion Sort is slow for large $N$ but the fastest for small $N$).
- **This approach makes big data Machine Learning practical for normal users.**

## PartitionMergesort

We sort the first $r\cdot N$ elements using unstable in-place Mergesort that requires an internal buffer of size $m$ to merge $m$ and $n$ elements where $m\leq n$ and $r\geq\frac{1}{2}$ and $r\leq\frac{2}{3}$.
We can see that this unstable in-place Mergesort can sort at most $\frac{2}{3}\cdot N$ elements without extra tricks.

After we have sorted the first $r\cdot N$ elements, we take the middle element of the sorted elements (the median) and partition the remaining elements according to this pivot element.
After this, we blockswap the elements right to the median with the elements left of the side that was partitioned.

This algorithm is similiar to the normal PartitionSort (a.k.a. _Leapfrogging Samplesort_) but because the first half is sorted using in-place Mergesort, we can guarantee $O(N\cdot\log_2{N})$ worst case and $\approx N\cdot\log_2{N}$ average case.
Choosing a higher $r$ requires a higher number of swaps and a lower number of comparisons in worst case and vice versa. But all choices of $r$ guarantee a worst case of $O(N\cdot\log_2{N})$.

In the following, we can see that the number of comparisons and number of writes to memory is quite low, maybe even better than an optimized HeapSort.
We only use linear access patterns with _PartitionMergesort_, so we can assume that the algorithm will be much faster than HeapSort for large $N$ and also faster than Mergesort in average case because the partition part is a plain Quicksort.

### Choosing $r=\frac{1}{2}$
+ $\tfrac{1}{2}$ of the sorting is done by Mergesort and $\tfrac{1}{2}$ by Quicksort


- Best case: $N\cdot\log_2{N}-N$ comparisons
- Average case: less than $N\cdot\log_2{N}$ comparisons, less than $N\cdot\log_2{N}$ swaps.
- Worst case: less than $2\cdot N\cdot\log_2{N}$ comparisons, $1\cdot N\cdot\log_2{N}$ swaps.

### Choosing $r=\frac{2}{3}$
+ $\tfrac{2}{3}$ of the sorting is done by Mergesort and $\tfrac{1}{3}$ by Quicksort

- Best case: $N\cdot\log_2{N}-N$ comparisons
- Average case: less than $N\cdot\log_2{N}$ comparisons, $\approx 1.05\cdot N\cdot\log_2{N}$ swaps.
- Worst case: less than $1.5\cdot N\cdot\log_2{N}$ comparisons, $\approx 1.2\cdot N\cdot\log_2{N}$ swaps.

### Choosing Quicksort
As a comparisons, here is the behaviour of Quicksort

- Best case: $N\cdot\log_2{N}$ comparisons, 0 swaps
- Average case: $1.39\cdot N\cdot\log_2{N}$ comparisons, $\frac{1}{4}\cdot N\cdot\log_2{N}$ swaps.
- Worst case: $\frac{N\cdot (N+1)}{2}$ comparisons, $\frac{1}{2}\cdot N\cdot\log_2{N}$ swaps.

##  Rotation-based partitioning (nearly) without rotations

Algorithms like the recursive stable in-place merging algorithms like RecMerge and [Symmerge](http://itbe.hanyang.ac.kr/ak/papers/esa2004.pdf), and recursive stable in-place [partitioning](https://en.cppreference.com/w/cpp/algorithm/stable_partition) algorithms use [rotations](https://github.com/scandum/rotate) to bring the elements in the correct side.

Instead of rotations, we can use blockswapping of the shorter side with the longer to solve the task. The longer side will then stay in a rotated state. We can just store the index of this rotation state.
In the next recursion, we repeat this with the index in mind. Both sides will never have more than one rotated state. Using this method, we can reduce the number of swaps by factor two.
In the last recursion where one side has length 0 and the other side is in a rotated state, we have to make one real rotation. This is the only place. We can use the simple triple reversal to finish this.

### Example
```
[1 2 3][4 5 6 7] -> [5 6 7|4][1 2 3]
```

### Applying the idea to the merging algorithms
Applying this idea to the partitioning algorithm, is not too difficult, but applying this idea to the merging algorithms can make the algorithm more complex because those algorithms require binary searching.
It is more difficult to binary search in such a rotated block. We can see that if we always keep one side in one block, we can always binary search in this side. Thus, the binary search will be the same and the complexity of implementing is not much higher than before.

### The same idea applied to an other algorithm
The same idea can also be applied to the merging methods that require _block rearrangement_, like [this one](https://academic.oup.com/comjnl/article-pdf/30/4/372/1068585/300372.pdf) and others. We can then already merge blocks when they are still in out-of-order state and delay the block rearrangement to the very last step when we really need the blocks to be in sorted order.
For a Mergesort this would mean:
- Merge two inputs where both inputs already have out-of-order blocks that normally would be rearranged. The very first step does not has out-of-order blocks.
- In the last step of Mergesort where each input has length $\frac{N}{2}$, we repeat the merge one last time.
- We now have a completely sorted input but it is still in blocks of same size that are not in order.
- This is the first and only time where we do the rearrangement.
- This results in $\approx N\cdot\log_2{N}+N$ memory reads and writes and $N\cdot\log_2{N}-N$ comparisons. The default Mergesort with $\frac{N}{2}$ extra memory requires $\approx \frac{3}{2}\cdot N\cdot\log_2{N}$ memory reads and writes and $N\cdot\log_2{N}-N$ comparisons.

The extra cost of the proposed method are some extra cache misses (but limited due to the block size) and extra branches.
Due to the reduced number of memory reads and writes and the lower number of external memory, it could be of high practical relevance.

## Merging with $O(\log_2{N})$ memory and the optimal number of comparisons
This novel algorithm is a recursive variant of the normal tape merge algorithm, thus, I call it _RecTapeMerge_. It's a little bit faster than RecMerge and Symmerge and for very big inputs it's even more faster. It's always slower than the normal tape merge, though.
Instead of binary searching, _RecTapeMerge_ uses linear searching. Thus, it will compare the exact same elements and number of elements like the normal merging method. The merging methods that depend on binary searching require a higher number of comparisons in average case. Like the other two, this algorithm requires $O(N\cdot\log_2{N})$ swaps, which is _not_ optimal. Because of the minimum number of comparisons, it's a good candidate for a hybrid method when a fixed size amount of memory exists as buffer.

The steps of the algorithm are as follows: 

1. While the next element in the left side is $\leq$ the next element in the right side: just increment the pointer in the left side.
2. While the next element in the right side is smaller than the next element in the left side: swap them.
3. Now, we have a block of some length $k$ in the middle where all $k$ elements are from the left side and all of these $k$ elements are the smallest $k$ from the left side.
   Now, recursively merge those $k$ elements with the right side.
4. We have finished some $l$ elements, with $l\geq k$. Now, if at least $l$ elements are still in the left side: blockswap them. Otherwise rotate them with the $l$ elements.
   Those $l$ elements are now at its final position.
5. We have the same situation like 3., but with $l$ elements in the middle instead of $k$ elements. Continue from 3.

We can see, that the algorithm most of time does blockswapping, but in each recursion, when not enough elements are left, it requires a rotation.

## In-place Mergesort with the optimal number of comparisons
- Choose the unstable in-place Mergesort that can sort $r\cdot N$ elements with $r\geq\frac{1}{2}$ and $r\leq\frac{2}{3}$.
- Choose $r=\frac{1}{2}$ and sort the first half of the input.
- Using the same method, sort the third quarter.
- Using a recusive call, sort the fourth quarter.
- Merge the third and the fourth quarter using _RecTapeMerge_.
- Merge the first half and the second half using _RecTapeMerge_.

Since _RecTapeMerge_ has the optimal number of comparisons and we merge $N + \frac{N}{2} + \frac{N}{4} + \frac{N}{8} + \dots$,
we get the following number of comparisons and swaps:

- Best case: $N\cdot\log_2{N}-N$ comparisons
- Average case: $N\cdot\log_2{N}-N$ comparisons, less than $2\cdot N\cdot\log_2{N}$ swaps.
- Worst case: $N\cdot\log_2{N}-N$ comparisons, $2\cdot N\cdot\log_2{N}$ swaps.

## Merging with $O(\sqrt{N})$ memory and the optimal number of comparisons
This can be done using [this](https://www.sciencedirect.com/science/article/abs/pii/S002001900500339X) merging algorithm. But instead we use external memory of size $2\cdot\sqrt{N}$ elements + $1\cdot\sqrt{N}$ index positions.
Because of the external memory, the algorithm becomes optimal w.r.t. the number of comparisons and the number of writes to memory falls from $3\cdot N$ to $2\cdot N$ which is nearly optimal.
[This algorithm](https://academic.oup.com/comjnl/article-pdf/30/4/372/1068585/300372.pdf) has a very similiar memory requirement and the same number of comparisons and writes to memory, as it seems.
It would be interesting to see, which one is faster on modern hardware and if one of them is fast enough to replace the default merging method that requires external memory for $N/2$ elements and has the same number of comparisons for all inputs like the other two mentioned above. 
