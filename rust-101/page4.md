## simd

simd is like a dot product

```rust
pub fn par(x: &[f64], y: &[i64]) -> f64 {
    use rayon::iter::*;
    x.par_iter()
        .zip(y.par_iter())
        .map(|(&xi, &yi)| xi * yi)
        .sum()
}
```
memory bandwidth stuffs.


# compute the product between two matrices

syskills cach hit cach miss ijk loops
imperative -> functional -> par

```rust
