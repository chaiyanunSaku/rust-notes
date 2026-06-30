## Page 2

### Sequential prefix sum

```rust
fn prefix_sum(xs: &[i32] -> (Vec<i32>), i32) {
    let mut results = vec![];
    let mut rs = 0i32;
    for &x in xs {
        result.push(rs);
        rs += x;
    }
    (results, rs)
}
```

prefix sum
    1. split in halves -> (l, r)
    2. ps recursively on l, r (in parallel)
    3. add left's total to each of right //map

Work and span, 
O(1), O(1)
2W(n/2), S(n/2)
O(n), O(logn)

total W(n) = 2W(n/2) + O(n) -> solves to O(nlogn)
S(n) = S(n/2) + O(logn) -> solves to O(log**2 n)  // not work efficient, worse than seq
throw away this idea (divine and conquer)

## Contraction 
2 steps process
- Combines items, recurse, expand.

3 1 4 2 9 5 7 9 // 3+1 and 4+2
4 6 13 16       // contracted n -> n/2

ps(con..) [0, 4, 10, 23], 39


prefix_sum
    1. Contract adk elts (n-> n/2)  // this step is map
    2. Recurse
    3. Expand: fill out missing #s // map too
    - even positioned; copy from above,
    - odd positioned; copy & add


```rust
// CAVEAT: will only work if |xs| = 2^k
fn prefix_sum(xs: &[i32]) -> (Vec<i32>, i32) {
    use rayon::iter::*;
    if xs.len() <= 1 { /* handle it */}
    let half = xs.len() /2
    // try .chucks
    let contracted: Vec<i32> = (0..half).into_par_iter()
                    .map(|i| xs[2*i] + xs[2*i+1])
                    .collect();
    let (cres, total) = prefix_sum(&contracted);
    (0..xs.len()).into_par_iter()
        .map(|i| if i % 2 == 0 { cres[i/2] } else [i/2] + xs[i-1])
        .collect();
    (results, total) // it tells us, 3 steps in prefix_sum is nothing more than a resurive map.
}
```

Work and span 
O(n), O(logn)  contract
W(n/2), S(n/2)  recurse
O(n), O(logn)  expand 

W(n) = W(n/2) + O(n) -> O(n)
S(n) = S(n/2) + O(logn) -> O(log^2n)

{ n -> n/2 -> n/4 -> ... -> 1 } logn
all that -> symblo is 1 logn

## POST BREAK
application of prefix_sum?
trying to do parallel filter

### filter
```rust
filter(&[2,5,8,6,7,4,9,16], is_odd)

flags = [0,1,0,0,1,0,1,1,0]
```

flatten 
```rust
[[2,3,1], [4,5], [3], [7,8,7]]

[3         ,2     ,1   ,3]
([0         ,3,    ,5   ,6], 9)

// run prefix sum on it
```

Binary operator: a + b   a x b    
+ is associative "group of parems ,ales mo difference"


prefix_scan | xs = [x0, x1, x2, .... , xn-1]
(+, Identity? I)
prefix_scan(xs) -> ([I, I + x0, I + x1, ... I + xn-2], total)