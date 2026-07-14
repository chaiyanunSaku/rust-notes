# Parallel Fibonacci via Matrix Prefix Scan — Study Notes

Everything from the assignment, in the order it makes sense.

---

## 1. The task

Write `pub fn par_fib_seq(n: u32) -> Vec<num_bigint::BigUint>` that returns
`[f1, f2, ..., fn]`, where `f1 = f2 = 1` and `f(n+2) = f(n+1) + f(n)`.

Pretending BigUint add/multiply is O(1), it must run in:

- **O(n) work**
- **O(log² n) span**

Plus a block comment deriving those bounds.

`BigUint` = an arbitrarily-large unsigned integer (Fibonacci numbers get huge,
past what a normal `u64` can hold). `Vec<BigUint>` = a growable list of them.

---

## 2. The math trick

Define the matrix `A = [[1, 1], [1, 0]]`. Then:

```
A · [f2, f1] = [f3, f2]
A²· [f2, f1] = [f4, f3]
Aᵏ· [f2, f1] = [f(k+2), f(k+1)]
```

More usefully, the powers themselves are made of Fibonacci numbers:

```
Aᵏ = [[f(k+1), f(k)], [f(k), f(k-1)]]
```

So the **top-left entry of Aᵏ is f(k+1)**. If we compute all the powers
`A¹, A², A³, ...` and read the top-left off each, we get the sequence.

Check by hand: `A = (1,1,1,0)`, and `A² = (2,1,1,1)`, `A³ = (3,2,2,1)`,
`A⁴ = (5,3,3,2)`. Top-lefts: `1, 2, 3, 5` = `f2, f3, f4, f5`. ✓

---

## 3. Why this is a "prefix scan"

**Prefix scan** = running totals of a list under *any associative operator*,
not just `+`. Examples:

- numbers with `+`  → running sums   (this is "prefix sum")
- matrices with matrix-multiply → running products ← what we need

Computing the powers is a running product:

```
input:  [A,  A,  A,  A, ...]
scan:   [A¹, A², A³, A⁴, ...]     (each = previous × one more A)
```

That's the same algorithm as prefix sum, with `mat_mul` swapped in for `+`.
Matrix multiply is **associative**, which is the only property a scan requires.

---

## 4. How the parallel scan works (contract / recurse / expand)

The scan is recursive: it calls itself on a **half-length** list, hits a base
case, then builds the answer back up.

**Base case** — list of length ≤ 1: the running product is just the element
itself, return it as-is.

**Contract** — pair up neighbors and combine each pair, halving the length.
All pairs are independent, so this is a parallel map:
`contracted[i] = combine(xs[2i], xs[2i+1])`.

**Recurse** — scan the half-length `contracted` list. Result `scanned[j]` =
running product up to and including pair `j`.

**Expand** — build the full-length answer using `scanned`, in parallel:

- **odd index i**: right element of its pair → answer is the whole pair's
  product, already in `scanned[i/2]`. (copy)
- **even index i (>0)**: left element → (product of all previous pairs) ×
  (this element) = `combine(scanned[i/2 - 1], xs[i])`.
- **i == 0**: nothing before it → just `xs[0]`.

### Trace to convince yourself (length 4, xs = [A,A,A,A])

```
contract: [A·A, A·A] = [A², A²]
recurse:  scan([A², A²]) = [A², A⁴]     (scanned)
expand:
  i=0: xs[0]                       = A¹
  i=1: scanned[0]                  = A²
  i=2: scanned[0] · xs[2] = A²·A   = A³
  i=3: scanned[1]                  = A⁴
result: [A¹, A², A³, A⁴]   ✓
```

---

## 5. Building the full function

1. Build a list of matrices to scan.
2. Scan it → the powers.
3. Read the top-left (`.0`) off each power → Fibonacci numbers.

**Offset detail:** the top-lefts of `A¹, A², ...` give `f2, f3, ...`, but the
sequence must start at `f1`. Two clean fixes:

- Hardcode `f1 = 1` at the front, then append top-lefts of `A¹ .. A^(n-1)`, OR
- Put an **identity matrix** `I = (1,0,0,1)` at position 0 of the input (since
  `M · I = M`), which shifts the powers so the top-lefts line up as
  `f1, f2, ..., fn` directly.

**Power-of-two caveat:** the halving scan assumes the length is a power of two.
For general `n`, pad the list up to the next power of two with identity matrices
`I` (harmless, since `M · I = M`), scan, then drop the padded tail.

---

## 6. Work and span — the analysis

Assume BigUint add/multiply (and therefore one `mat_mul`) is O(1).

### The atom: a parallel map over m elements

- **Work = O(m)** — someone must do all m operations.
- **Span = O(log m)** — the runtime splits the range in half repeatedly (a
  tree), so fanning the work out and back is log-deep.

### The scan

Each call = a couple of parallel maps (contract + expand) + one recursive call
on half the size.

**Span → O(log² n):**

```
number of levels  = log n     (list halves each call: n → n/2 → ... → 1)
span per level    = O(log n)  (each level is a parallel map)
total span        = log n × log n = O(log² n)
```

As a recurrence: `S(n) = S(n/2) + O(log n)` → `O(log² n)`.

**Work → O(n):**

```
n + n/2 + n/4 + ... + 1 = 2n = O(n)   (geometric series)
```

### Whole function

Building the input list and extracting top-lefts are each one parallel map
(O(n) work, O(log n) span) — dominated by the scan. So:

- **Total work: O(n)**
- **Total span: O(log² n)**

---

## 7. The big-picture point

The naive recurrence `f(k) = f(k-1) + f(k-2)` has O(n) work but also **O(n)
span** — each number depends on the previous one, a single forced chain that
parallelism can't break.

Because matrix multiply is **associative**, the same computation can be
rearranged as a prefix scan whose critical path (span) is only **O(log² n)**,
while keeping the same **O(n)** total work. Same work, much shorter dependency
chain → parallelism can actually help.

---

## 8. Vocabulary cheatsheet

- **Work** — total operations, as if one worker did everything. Parallelism
  does NOT reduce it.
- **Span** — longest chain of steps that *must* run in order; the fastest
  possible finish time with unlimited workers.
- **log₂ n** — how many times you halve n to reach 1 (≈30 for a billion).
  This is why halving in recursion produces a `log`.
- **log² n** — `(log n)²`, i.e. log then squared. NOT log(log n). (≈900 for a
  billion — still tiny next to n.)
- **Associative operator** — `(a·b)·c = a·(b·c)`. Required for prefix scan.
  Both `+` and matrix-multiply qualify.
- **Identity matrix** `I = (1,0,0,1)` — the "do nothing" value for
  matrix-multiply: `M · I = M`. The multiplicative analogue of `0` for `+`.
- **Speed ranking:** `O(log n) < O(log² n) < O(n)`.