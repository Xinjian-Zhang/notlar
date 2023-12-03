## Advanced Programming Coursework 8 - Sequences, laziness and computation expressions

----

### Task 1: Pascal's triangle

```
            1
            1 1
        1 2 1
        1 3 3 1
        1 4 6 4 1
        ...........
    .............
    ............... 
```

Define the function

`next : int list -> int list`

that, given a row of the triangle, computes the next row. The function `List.windowed` may be useful here.

Define the function

`triangle : int list seq`

which consists of the rows of Pascal's triangle (represented as int list). Do not use sequence expressions. Define this using `Seq.unfold`.

Define the function

`evens : int -> int list`

so that

`evens n`

evaluates to a list (of length n) consisting of the sums of elements in the first n rows of Pascal's triangle that have an even number of elements.

```fsharp
let next (row: int list): int list =
    let extendedRow = [0] @ row @ [0]
    extendedRow 
    |> List.windowed 2 
    |> List.map (fun window -> List.sum window)

let triangle: int list seq =
    Seq.unfold (fun state -> Some(state, next state)) [1]

let evens (n: int): int list =
    triangle
    |> Seq.filter (fun row -> (List.length row) % 2 = 0)
    |> Seq.map List.sum
    |> Seq.take n
    |> Seq.toList
```

----

### Task 2

Define the function

`generate : 'a list -> ('a list -> 'a) -> 'a seq`

so that

`generate xs f`

evaluates to a sequence consisting of the elements in xs followed by
elements computed by the function f.

More precisely, if List.length xs = n, then s_i (the i-th element in
the sequence) is

* the i-th element of the list xs   if i < n
* f [s_{i - n}; ... ; s_{i - 1}]     otherwise

evaluates to a sequence consisting of the elements in xs followed by elements computed by the function f.

More precisely, if `List.length xs = n`, then `s_i` (the i-th element in the sequence) is

- the i-th element of the list xs if i < n
- `f [s_{i - n}; ... ; s_{i - 1}]` otherwise

> Note that f must be applied to lists of same length as xs.
> You may assume that xs is not empty.
> Define this using sequence expressions.
> Make sure that the calculation of an element of the sequence uses the function f at most once.
> The function `Seq.cache` may be useful here.

```fsharp
let generate (xs: 'a list) (f: 'a list -> 'a): 'a seq =
    let n = List.length xs
    let rec nextEle window =
        let newEle = f window
        seq {
            yield newEle
            let updatedWindow = (List.tail window) @ [newEle]
            yield! nextEle updatedWindow
        }

    Seq.append (Seq.ofList xs) (nextEle xs) |> Seq.cache
```

----

### Task 3 Longest common subsequence

We have two arrays, xs and ys, and we wish to find the length of the longest common subsequence in these two arrays.

Example:
- xs = [| 1; 2; 3; 4 |]
- ys = [| 5; 1; 6; 4 |]

Length of the longest common subsequence is 2.

This can be solved using dynamic programming.

Let D be a two-dimensional array that holds the results of the subproblems:

- `D[i, j]` is the length of the lcs of `xs[0 .. i - 1]` and `ys[0 .. j - 1]`.

Solving the subproblems:

- if `xs[i - 1] = ys[j - 1]` then we follow two subproblems (shorten one sequence at a time):
    `D[i, j] = max(D[i - 1, j], D[i, j - 1])`

- otherwise we take the maximum of two subproblems:
    `D[i, j] = max D[i - 1, j] D[i, j - 1]`

- base cases:
    `D[i, 0] = D[0, j] = 0`

Observation: it is not necessary to fill the entire table D to calculate `D[i, j]`.

If we decide to fill only those parts of the table that are necessary to compute `D[i, j]`, then we need to be careful to not use the values in the unfilled parts in our calculation.

However, we can use lazy values instead and let the runtime figure out which entries in the table and in which order need to be calculated.

Define the function

`lcs : ((int * int) -> unit) -> 'a [] -> 'a [] -> Lazy<int> [,]`

so that

`lcs m xs ys`

evaluates to the table D for xs and ys except that the entries in the table are lazy. An entry in the table is computed only when we ask for the value (of the `Lazy<int>`) or the computation of another entry requires the value of this entry.

The function m must be applied to `(i, j)` when the entry `D[i, j]` is actually computed. For example, you can use printfn as m to make the order of calculations visible.

```fsharp
let lcs (m: (int * int) -> unit) (xs: 'a []) (ys: 'a []): Lazy<int> [,] =
let lenX = xs.Length
let lenY = ys.Length
let D = Array2D.init (lenX + 1) (lenY + 1) (fun _ _ -> lazy(0))

for i in 0..lenX do
    for j in 0..lenY do
        D.[i, j] <- lazy (
            if i = 0 || j = 0 then
                m (i, j); 0
            else
                m (i, j)
                if xs.[i - 1] = ys.[j - 1] then
                    max (D.[i - 1, j].Value) (D.[i, j - 1].Value) + 1
                else
                    max (D.[i - 1, j].Value) (D.[i, j - 1].Value)
        )

D
```

----

### Task 4

This task is similar to task 4 of coursework 6. Now it is necessary to rewrite your solution using the CPS builder syntax introduced in Lecture 11 (check out adprog11.fsx in the course-materials repository).

Below you find the definition of a type Tree of leaf-labeled trees. Write a function maxAndMax2InTree : int Tree -> int * int that returns the maximum and and the second largest elements of the tree.

Use the CPSBuilder based continuation-passing style in your implementation, i.e. use the computation expression syntax.

Flattening the tree to a list and processing the list is not considered a correct solution.

If the input is a tree with a single element, then the second element in the returned pair should be the minimum value of int.

If there are 2 instances of the same maximum value in the tree, then the two values should be returned as pair.

```fsharp
type CPSBuilder () =
    member this.Bind (cps, f) = fun cont -> cps (fun x -> f x cont)
    member this.Return x      = fun cont -> cont x

let cps= new CPSBuilder ()

let runCPS cps = cps id

type 'a Tree =
  | El of 'a
  | Br of 'a Tree * 'a Tree
```

Task 4 starts here:

```fsharp


对于Task 4的代码，请你检查一下是否满足题目要求。如果是对的，由于代码写的过于冗余和复杂，请你尽可能地优化代码结构，尽量改变代码的结构，以防止被判定为抄袭。命名最好延续Task1,2,3的风格，以及遵循题目的要求。如果代码不对，给我正确答案。