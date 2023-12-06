## Advanced Programming Coursework 9 - asynchronous programming

----

Our representation of complex numbers.

```fsharp
type Complex = double * double
```

### Question 1

The Mandelbrot set.

Define the function

```fsharp
mandelbrot : int -> Complex -> bool
```

so that 'mandelbrot n' is the characteristic function of the approximation of the Mandelbrot set by n iterations of the quadratic map.

In other words, define the function so that given

`n : int`

`c : Complex`

`mandelbrot n c` evaluates to true precisely when, according to n iterations of the quadratic map, c belongs to the Mandelbrot set.

The quadratic map is given in "Formal definition" and a condition for deciding whether c belongs to the Mandelbrot set (the given condition requires something for all n, we use it as up to n) is given in "Basic properties".

Here are some hints:
- Start by computing the z_1, z_2, ..., z_n for c. Based on this sequence of values decide whether c belongs to the (approximation of the) Mandelbrot set or not.
- How to add and multiply complex numbers?
- What is the absolute value of a complex number?

```fsharp
let mandelbrot n (c: Complex) =
    let rec iterate a b i =
        let a' = a * a
        let b' = b * b
        if a' + b' > 4.0 then false
        elif i >= n then true
        else iterate (a' - b' + fst c) (2.0 * a * b + snd c) (i + 1)

    iterate 0.0 0.0 0
```

----

### Question 2

(*
   Question 2

   Define the function
 
   divide : int -> int -> (int * int) seq

   so that given

   m : int
   n : int

   'divide m n' evaluates to a sequence of ranges

   (s_1, e_1), ..., (s_m, e_m)

   so that:
   * s_1 = 0 and e_m = n - 1

   * s_{i + 1} = e_i + 1


   That is to say that:   
   * for every x : int such that 0 <= x < n there exists an i such
     that s_i <= x <= e_i

   * the sequence of (s_i, e_i) covers 0, ..., n - 1 without overlap,
     i.e., if s_i <= x <= e_i and s_j <= x <= e_j, then i = j.

   You can think of n as the number of things we have and m as the
   number of buckets among which we distribute them.

   Try to divide fairly.
*)

Define the function

```fsharp
divide : int -> int -> (int * int) seq
```

so that given

`m : int`

`n : int`

`divide m n` evaluates to a sequence of ranges

`(s_1, e_1), ..., (s_m, e_m)`

so that:
- `s_1 = 0` and `e_m = n - 1`
- `s_{i + 1} = e_i + 1`

That is to say that:
- for every `x : int` such that `0 <= x < n` there exists an `i` such that `s_i <= x <= e_i`
- the sequence of `(s_i, e_i)` covers `0, ..., n - 1` without overlap, i.e., if `s_i <= x <= e_i` and `s_j <= x <= e_j`, then `i = j`.

> You can think of `n` as the number of things we have and `m` as the number of buckets among which we distribute them.
> Try to divide fairly.

```fsharp

