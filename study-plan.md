# Algorithm Analysis — Complete Study Guide (Zero to Exam-Ready)

> Based on your kisi-kisi. Four topics:
> 1. Divide and Conquer (Time + Space Complexity)
> 2. Amortized Analysis (Operations per execution)
> 3. Randomized Algorithms (Expected running time — Binary Search, Quick Sort)
> 4. Greedy Algorithms (Prove correctness from pseudocode)

---

## 0. THE FOUNDATION YOU NEED FIRST: Big-O Notation

Before anything else, you MUST understand Big-O. Every topic uses it.

**What it is:** Big-O describes how the *number of operations* (or memory) grows as the input size `n` grows. We ignore constants and small terms, and keep only the dominant (fastest-growing) part.

**Read your graph (top-right of the kisi-kisi) from BEST to WORST:**

| Notation | Name | Meaning | Example |
|----------|------|---------|---------|
| O(1) | Constant | Same work no matter the size | Access `array[5]` |
| O(log n) | Logarithmic | Halves the problem each step | Binary search |
| O(n) | Linear | One pass through the data | Find max in a list |
| O(n log n) | Linearithmic | Good sorting | Merge Sort, Quick Sort (avg) |
| O(n²) | Quadratic | Nested loops | Bubble Sort |
| O(2ⁿ) | Exponential | Doubles each item added | Naive recursion (Fibonacci) |
| O(n!) | Factorial | Tries all orderings | Brute-force Traveling Salesman |

**Rules for simplifying Big-O:**
- Drop constants: `O(2n)` → `O(n)`, `O(500)` → `O(1)`
- Drop smaller terms: `O(n² + n)` → `O(n²)`
- Multiply for nested loops: a loop inside a loop = `O(n) × O(n) = O(n²)`

**Clue to read the graph:** The steeper the curve, the worse it is. `O(1)` is flat (best). `O(n!)` and `O(2ˣ)` shoot straight up (worst). When asked to "estimate using the O-notation graph," you draw/point to which curve matches your operation count.

---

## TOPIC 1: Divide and Conquer (Time Complexity + Space Complexity)

### 1.1 What is Divide and Conquer? (D&C)

A strategy with **3 steps**:
1. **Divide** — break the problem into smaller sub-problems (usually halves).
2. **Conquer** — solve each sub-problem recursively.
3. **Combine** — merge the sub-answers into the final answer.

Classic examples: **Merge Sort, Quick Sort, Binary Search**.

### 1.2 How to compute TIME complexity — the Recurrence + Master Theorem

When an algorithm calls itself, you write a **recurrence relation**:

```
T(n) = a · T(n/b) + f(n)
```
- `a` = how many sub-problems you create
- `n/b` = the size of each sub-problem
- `f(n)` = the work to divide + combine (outside the recursion)

**Master Theorem** — compare `f(n)` against `n^(log_b a)`:

Let `c = log_b(a)`.
- **Case 1:** if `f(n)` grows slower than `n^c` → answer is `O(n^c)`
- **Case 2:** if `f(n)` ≈ `n^c` (same) → answer is `O(n^c · log n)`
- **Case 3:** if `f(n)` grows faster than `n^c` → answer is `O(f(n))`

### 1.3 WORKED EXAMPLE A — Merge Sort

```
MergeSort(A):
    if length(A) <= 1: return A
    mid = length(A) / 2
    left  = MergeSort(A[0..mid])     # 1 sub-problem of size n/2
    right = MergeSort(A[mid..end])   # 1 sub-problem of size n/2
    return Merge(left, right)        # merging takes O(n)
```

**Step-by-step:**
- You split into **2** sub-problems → `a = 2`
- Each is **half** the size → `b = 2`
- Merging two halves is one linear pass → `f(n) = O(n)`
- Recurrence: `T(n) = 2·T(n/2) + O(n)`
- `c = log_2(2) = 1`, so `n^c = n`. Compare `f(n) = n` vs `n^1 = n` → **equal → Case 2**
- **Time = O(n log n)** ✅

**Space complexity** (what the kisi-kisi asks — "memory used during execution"):
- Merge Sort needs a **temporary array of size n** to merge → `O(n)` extra memory
- Plus the recursion stack goes `log n` deep → `O(log n)`
- Dominant term: **Space = O(n)**

### 1.4 WORKED EXAMPLE B — Binary Search

```
BinarySearch(A, target, low, high):
    if low > high: return NOT_FOUND
    mid = (low + high) / 2
    if A[mid] == target: return mid
    if A[mid] < target: return BinarySearch(A, target, mid+1, high)
    else:               return BinarySearch(A, target, low, mid-1)
```
- Only **1** sub-problem each time → `a = 1`
- Half the size → `b = 2`
- Work outside recursion is just a comparison → `f(n) = O(1)`
- Recurrence: `T(n) = 1·T(n/2) + O(1)`
- `c = log_2(1) = 0`, `n^c = n^0 = 1`. Compare `f(n)=1` vs `1` → equal → Case 2
- **Time = O(1 · log n) = O(log n)** ✅
- **Space:** recursive version uses `O(log n)` stack; iterative version uses `O(1)`.

### 1.5 How to think about SPACE COMPLEXITY (zero-knowledge)

Space = total extra memory your algorithm needs beyond the input. Count:
1. **Extra data structures** you allocate (temp arrays, hash maps).
2. **Recursion stack depth** — each recursive call stores a stack frame. Depth of recursion = stack space.

Examples:
- Merge Sort: temp array `O(n)` + stack `O(log n)` → **O(n)**
- Binary Search (recursive): just stack `O(log n)` → **O(log n)**
- A simple loop summing an array: just a counter → **O(1)**

### 🎯 PRACTICE QUESTIONS — Topic 1
1. Write the recurrence relation for an algorithm that splits the input into **3** sub-problems of size **n/3**, and does `O(n)` work to combine. Then solve its time complexity using the Master Theorem.
2. Here is pseudocode. Give its **time** AND **space** complexity, and explain the space part:
   ```
   PowerOfTwo(n):
       if n == 0: return 1
       half = PowerOfTwo(n/2)
       return half * half
   ```
3. Merge Sort uses `O(n)` extra space. Rewrite/explain how an **in-place** sort like Quick Sort differs in space complexity, and state Quick Sort's space complexity (average and worst case).

---

## TOPIC 2: Amortized Analysis (Operations per execution)

### 2.1 What problem does it solve?

Sometimes a single operation is **occasionally expensive** but **usually cheap**. Looking only at the worst single operation is misleading. **Amortized analysis finds the AVERAGE cost per operation across a whole sequence** — even though we're not using probability, we're spreading the rare expensive cost over many cheap ones.

> Analogy: A gym membership costs $600/year. One payment looks huge, but amortized over 365 days it's ~$1.64/day. Amortized analysis does this for operations.

### 2.2 The three methods

**(a) Aggregate method** — Total cost of `n` operations ÷ `n`.
**(b) Accounting (banker's) method** — Charge cheap operations a little extra ("credit") and save it to "pay" for future expensive ones.
**(c) Potential method** — Define a "potential energy" function Φ that stores up before expensive operations.

For your exam, the **Aggregate method** is usually enough and easiest.

### 2.3 WORKED EXAMPLE — Dynamic Array (the classic)

Imagine an array that **doubles in size** when full. Appending is usually `O(1)`, but when full you must copy all elements to a new array (`O(n)` — expensive!).

**Question:** What is the amortized cost of one `append`?

**Aggregate reasoning:** Insert `n` items starting from size 1. The array doubles at sizes 1, 2, 4, 8, ..., n. The copy costs happen at those points:
```
copies = 1 + 2 + 4 + 8 + ... + n  ≈  2n   (geometric series sums to about 2n)
```
- Normal insert cost for `n` items = `n`
- Total work = `n (inserts) + 2n (copies) = 3n`
- Amortized cost per operation = `3n / n = 3 = O(1)` ✅

**Conclusion:** Even though one append can cost `O(n)`, the **amortized cost is O(1)** per operation.

### 2.4 Accounting method on the same example

Charge each insert **3 credits**: 1 pays for the insert itself, 2 are saved. When a doubling/copy happens, the saved credits pay for moving the old elements. The "bank balance" never goes negative → amortized `O(1)` proven. ✅

### 🎯 PRACTICE QUESTIONS — Topic 2
1. A stack supports `push` (cost 1) and `multipop(k)` which pops `k` items (cost = number of items actually popped). Using the **aggregate method**, show that a sequence of `n` operations costs `O(n)` total, so each operation is amortized `O(1)`.
2. A binary counter is incremented `n` times. Incrementing flips some bits (sometimes just 1 bit, sometimes many due to carries). Use the **aggregate method** to show the amortized cost per increment is `O(1)`. (Clue: count how often each bit position flips — bit 0 flips every time, bit 1 every 2nd time, bit 2 every 4th time...)
3. Explain in your own words the difference between **worst-case** cost of a single operation and **amortized** cost, using the dynamic array as your example.

---

## TOPIC 3: Randomized Algorithms (Expected Running Time)

### 3.1 What is a randomized algorithm?

An algorithm that makes **random choices** during execution. Why? To avoid worst-case inputs that an adversary could design against us. The running time becomes a matter of **expectation (average over the random choices)**, not a fixed value.

Two famous ones from your kisi-kisi: **Quick Sort** and **Binary Search** (randomized variants).

### 3.2 Expected value — the one formula you need

```
E[X] = Σ (value × probability of that value)
```
Example: a fair die. `E = (1+2+3+4+5+6)/6 = 3.5`. The "expected running time" is the weighted average of running times over all random choices.

### 3.3 WORKED EXAMPLE — Randomized Quick Sort

```
QuickSort(A):
    if length(A) <= 1: return A
    pivot = A[ RANDOM index ]          # <-- randomization
    left  = elements < pivot
    right = elements > pivot
    return QuickSort(left) + [pivot] + QuickSort(right)
```

**Why randomize?** With a fixed pivot (e.g., always first element), a **sorted** input gives the worst case `O(n²)` — every partition is lopsided (n-1 and 0). By picking a **random pivot**, no specific input can force the worst case.

**Expected running time reasoning (the proof idea):**
- Each level of recursion does `O(n)` work partitioning.
- A random pivot gives a "good enough" split (at least 1/4 : 3/4) with probability ≥ 1/2.
- So the recursion depth is expected `O(log n)` levels.
- `O(n)` work × `O(log n)` levels = **Expected time O(n log n)** ✅
- **Worst case is still O(n²)** (if unlucky every time) but the *probability* of that is astronomically small.

**Estimate on the O-graph:** You'd point to the `O(n log n)` curve for the expected case, and note `O(n²)` as the unlikely worst case above it.

### 3.4 WORKED EXAMPLE — Binary Search (and a randomized view)

Standard Binary Search is deterministic `O(log n)` — it always halves. A **randomized** variant might pick a random split point instead of the exact middle.
- Expected work is still `O(log n)` because on average you still remove a constant fraction each step.
- This shows randomization doesn't always hurt: the *expected* number of comparisons stays logarithmic.

### 3.5 "Modify and prove" — what the exam wants

The kisi-kisi says *"Modifikasi dan membuktikan expected running time."* This means:
1. **Modify**: take a deterministic algorithm, inject a random choice (e.g., random pivot).
2. **Prove**: write out `E[T(n)]` using the expectation formula / recurrence, and show it equals `O(n log n)` (or whatever).
3. **Estimate**: match it to a curve on the O-notation graph.

### 🎯 PRACTICE QUESTIONS — Topic 3
1. Explain why **deterministic** Quick Sort (pivot = first element) is `O(n²)` on an already-sorted array, and show how a **random pivot** fixes this. Walk through what the partitions look like in both cases.
2. For randomized Quick Sort, write the recurrence for the **expected** running time and argue (in words) why it solves to `O(n log n)`. (Clue: expected split is balanced → `E[T(n)] = 2·E[T(n/2)] + O(n)`.)
3. You flip a fair coin until you get HEADS. Using the expected-value formula, compute the **expected number of flips**. (Clue: it's a geometric distribution; answer is `1/p`.) Then explain how this "expected number of trials" idea connects to analyzing randomized algorithms.

---

## TOPIC 4: Greedy Algorithms (Prove correctness from pseudocode)

### 4.1 What is a greedy algorithm?

At each step, make the choice that **looks best right now** (the "locally optimal" / greedy choice), never reconsidering. Hope that these local choices lead to a **globally optimal** solution.

Examples: Coin change (with nice denominations), Activity Selection, Huffman Coding, Dijkstra's shortest path, Kruskal's/Prim's MST.

### 4.2 When does greedy actually WORK? — The two properties to PROVE

A greedy algorithm is **correct** only if BOTH hold:

**(1) Greedy Choice Property** — A globally optimal solution can be reached by making the locally optimal (greedy) choice at each step. (The greedy choice is "safe.")

**(2) Optimal Substructure** — After making the greedy choice, what remains is a smaller version of the same problem, and an optimal solution to the whole = greedy choice + optimal solution to the rest.

### 4.3 How to PROVE greedy correctness — the "Exchange Argument"

This is the standard proof technique the exam wants:
1. Assume there is an optimal solution `OPT` that differs from the greedy solution `G`.
2. Find the first place they differ.
3. **Exchange** `OPT`'s choice there for the greedy choice — show this does NOT make `OPT` worse (it stays optimal, or gets better).
4. Repeat until `OPT` becomes `G`. Therefore `G` is also optimal. ∎

### 4.4 WORKED EXAMPLE — Activity Selection

**Problem:** Given activities with start/finish times, select the maximum number that don't overlap.
**Greedy rule:** Always pick the activity that **finishes earliest** among those still compatible.

```
ActivitySelection(activities):
    sort activities by finish time (ascending)
    selected = [ first activity ]
    last_finish = activities[0].finish
    for each activity a in order:
        if a.start >= last_finish:       # compatible = greedy choice
            selected.add(a)
            last_finish = a.finish
    return selected
```

**Proof it's correct (exchange argument):**
- Greedy choice property: The first activity to finish leaves the most remaining time for others. Any optimal solution's first activity can be swapped for the earliest-finishing one without reducing the count → greedy choice is safe.
- Optimal substructure: After picking it, the remaining problem is "select max activities starting after `last_finish`" — the same problem, smaller. ✅
- Therefore greedy is optimal.

### 4.5 WORKED EXAMPLE — When Greedy FAILS (very important!)

**Coin change with coins {1, 3, 4}, make amount 6:**
- Greedy: take 4 → remaining 2 → take 1, 1 → total **3 coins** (4+1+1)
- Optimal: 3 + 3 → **2 coins** ❌
- **Greedy gives the wrong answer here!** The exam may give you pseudocode and ask you to *prove it's WRONG* by finding a counterexample like this. Finding ONE counterexample disproves greedy.

### 4.6 How to answer "analyze if this pseudocode correctly uses greedy, and prove it"

Your answer structure:
1. State the greedy rule the code uses.
2. Check the **greedy choice property** — is the local choice always safe? (Prove with exchange argument, OR disprove with a counterexample.)
3. Check **optimal substructure**.
4. **Conclude**: either prove it's correct (both properties hold) OR give one counterexample showing it fails.

### 🎯 PRACTICE QUESTIONS — Topic 4
1. Given the coin system {1, 5, 10, 25} (US coins) and a greedy "always take the largest coin ≤ remaining amount" algorithm — argue whether greedy is optimal here. Then show it is NOT optimal for the system {1, 3, 4} using a counterexample.
2. Here is pseudocode for a "greedy" solution that picks the activity with the **shortest duration** first (instead of earliest finish). Prove this is **incorrect** by constructing a counterexample where it selects fewer activities than optimal. (Clue: one short activity can overlap two longer non-overlapping ones.)
3. Explain the **exchange argument** in your own words, then use it to prove that the "earliest finish time" greedy choice for Activity Selection is safe (always leads to an optimal solution).

---

## 📅 SUGGESTED STUDY PLAN (adjust to your time)

| Day | Focus | Goal |
|-----|-------|------|
| 1 | Topic 0 (Big-O) + read the graph | Be able to rank O(1)…O(n!) and simplify expressions |
| 2 | Topic 1: D&C, Master Theorem, time+space | Solve Merge Sort & Binary Search recurrences by hand |
| 3 | Topic 1 practice questions | Do all 3 without notes |
| 4 | Topic 2: Amortized (aggregate + accounting) | Explain dynamic array O(1) amortized |
| 5 | Topic 2 practice + Topic 3: Randomized intro | Expected value formula + Quick Sort reasoning |
| 6 | Topic 3 practice (expected running time proofs) | Write the E[T(n)] recurrence from memory |
| 7 | Topic 4: Greedy + exchange argument | Prove Activity Selection; know a failure counterexample |
| 8 | Mix: redo all practice questions + the graph | Simulate exam conditions |

### Exam-day checklist (map to your 4 kisi-kisi points)
- [ ] **#1 D&C:** Can I write a recurrence AND compute space (memory) complexity?
- [ ] **#2 Amortized:** Can I compute cost per operation using the aggregate method?
- [ ] **#3 Randomized:** Can I modify an algorithm to be randomized and prove expected `O(n log n)`, and point to the right O-graph curve?
- [ ] **#4 Greedy:** Given pseudocode, can I prove correctness with an exchange argument OR disprove it with a counterexample?
