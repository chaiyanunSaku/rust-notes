## parallel technique II

Prime Sieve

Closest pair in 2D



Prime Sieve's total running time = n/2 + n/3 + n/5 + .... + n/p  = sum of prime p<= n; n/p
= sum of prime p <= n; 1/p = O(nloglogn)


Contractions

```rust
fn sieve(n) {   
    sieve(n.sqrt())  // find all primes <= n.sqrt()
    // for each prime <= n.sqrt() in parallel corrst out their multiples.
}
```


# Seq Sieve
```rust
fn seq_sieve_into(n: u32, s: &mut [bool]) {
    let sqrt_n = (n as f64).sqrt() as usize;
    for p in 2..=sqrt_n as usize {
        if !s[p] { continue; }
        // p must be prime
        for i in 2..=(n/p) as usize {
            s[i*p] = false;
        }
    }
}
```


```rust
fn seq_sieve(n: u32) -> Vec<u32> {
    let mut s: Vec<bool> = (0..=n).map(|_| true).collect();

    seq_sieve_into(n, &mut s);
    
    (2..=n).filter(|&k| s[k as usize]).collect()
}

```

Parallel

```rust
fn par_sieve_into(n: u32, s: &mut [bool])
```