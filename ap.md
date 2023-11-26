## Advanced Programming in F#

----

### Coursework 4

NB!

The ECMA-404 standard specifies a textual syntax for structured data
interchange.

Goal: develop a partial implementation of this specification. In particular, our first goal is to define in F# the datatype(s) suitable for representing the abstract syntax tree of this data interchange format. The second goal is to define some
operations on this representation.

We have the following type alias.

```fsharp
type Name = string
```

#### Task 1

Define the type `Ecma` for representing the possible values of the ECMA-404 interchange format.

The type must satisfy the constraint `equality`.

```fsharp
type Ecma =
    | Object of (Name * Ecma) list   // Obj: List of key-value pairs
    | Number of float
    | Bool of bool
    | String of string
    | Array of Ecma list
    | Null
```

Define the **following functions** for creating ECMA-404 representations of the given data.

(1) Define the function `mkObject : unit -> Ecma` that creates a representation for an empty object structure.

```fsharp
let mkObject () = Object[]
```

(2) Define the function `mkNumber : float -> Ecma` that creates a representation for the given floating-point number.

```fsharp
let mkNumber f = Number f
```

(3) Define the function `mkBool : bool -> Ecma` that creates a representation for the given Boolean value.

```fsharp
let mkBool b = Bool b
```

(4) Define the function `mkString : string -> Ecma` that creates a representation for the given string value.

```fsharp
let mkString s = String s
```

(5) Define the function `mkArray : Ecma list -> Ecma` that creates a representation for an array whose elements are represented by the given list of `Ecma` values.

```fsharp
let mkArray ele = Array ele
```

(6) Define the function `mkNull : unit -> Ecma` that creates a representation of the ECMA-404 `null` value.

```fsharp
let mkNull () = Null
```

#### Task 2

Define the function `addNameValue : Name * Ecma -> Ecma -> Ecma` so that `addNameValue (n, v) e` evaluates to an ECMA representation `e'` that is:

- equal to `e` if `e` is not an object representation
- a representation for the object `e` extended with the name-value pair `(n, v)`, otherwise.

```fsharp
let addNameValue (n, v) e =
  match e with
    | Object obj -> Object (obj@[(n, v)])
    | _ -> e  
```
<br>

Define the function `addValue : Ecma -> Ecma -> Ecma` so that `addValue v e` evaluates to an ECMA representation `e'` that is:

- equal to `e` if `e` is not an array representation
- a representation for the array `e` with the value `v` added as the last element, otherwise.

```fsharp
let addValue v e =
    match e with
    | Array arr -> Array (arr@[v])
    | _ -> e
```

#### Task 3

Define the function `countValues : Ecma -> int` that counts the number of ECMA values in the given representation.

Keep in mind that both objects and arrays are themselves values and may contain other values inside.

Furthermore, the following should hold:

- `1 + countValues e <= countValues (addValue v e)` if `e` is an array representation
- `1 + countValues e <= countValues (addNameValue (n, v) e)` if `e` is an object representation

```fsharp
let rec countValues (e: Ecma) : int =
  match e with
  | Object obj -> 1 + List.sumBy (fun (_, v) -> countValues v) obj
  | Array arr -> 1 + List.sumBy countValues arr
  | _ -> 1
```

#### Task 4

```fsharp
type Path = Name list
```

Define the function `listPaths : Ecma -> Path list` that computes the full path for all the values in the given ECMA representation.

A path is just a list of names that take us from the root of the representation to a particular value.

For arrays, we consider the same path to take us to the array and to all of the elements in the array. Thus, for an array, we include the path to it and its elements only once in the result.

If `e : Ecma` represents the following structure

```json
{
  "abc" : false,
  "xs"  : [ { "a" : "a" }, 1.0, true, { "b" : "b" }, false ],
  "xyz" : { "a" : 1.0,
            "b" : { "b" : "b" } },
  "ws"  : [ false ]
}
```

then  `listPaths e` should result in

```fsharp
[
  [];
  ["abc"];
  ["xs"];
  ["xs"; "a"];
  ["xs"; "b"];
  ["xyz"];
  ["xyz"; "a"];
  ["xyz"; "b"];
  ["xyz"; "b"; "b"];
  ["ws"]
]
```

The ordering of paths in the result list matters:

- paths to (sub)values in an array respect the order of elements in the array
- paths to values in an object respect the order in which the values were added to the object (most recently added appears last).

Note that the empty list denotes the path to the root object.

```fsharp
let rec paths e =
    match e with
    | Object obj ->
        obj
        |> List.collect (fun (n, v) ->
            match v with
            | Array _ | Object _ ->
                let nestedPaths = paths v |> List.map (fun p -> n :: p)
                [ [n] ] @ nestedPaths
            | _ -> [ [n] ])
    | Array ele ->
        ele
        |> List.collect (fun ele -> paths ele |> List.map (fun p -> p))
    | _ -> []

let listPaths (ecma: Ecma) : List<Path> = [ [] ] @ paths ecma
```

#### Task 5

Define the function `show : Ecma -> string` that computes a string representation of the given ECMA representation in such a way that the ordering requirements from the previous task are respected.

The result should not contain any whitespace except when this whitespace was part of a name or a string value.

```fsharp
let rec show (e: Ecma) : string =
    match e with
    | Null -> "null"
    | Bool b -> b.ToString().ToLower()
    | Number n -> n.ToString()
    | String s -> "\"" + s + "\""
    | Array arr -> 
        "[" + String.concat "," (List.map show arr) + "]"
    | Object obj -> 
        "{" + String.concat "," (List.map (fun (n, v) -> "\"" + n + "\":" + show v) obj) + "}"
```

#### Task 6

Define the function `delete : Path list -> Ecma -> Ecma` so that `delete ps e` evaluates to a representation `e'` that is otherwise the same as `e` but all name-value pairs with paths in the path list `ps` have been removed.

When the user attempts to delete the root object, `delete` should throw an exception. Hint: use `failwith` in the appropriate case.

```fsharp
let rec delAtPath p (e: Ecma) =
    match e with
    | Object obj ->
        match p with
        | [] -> Object obj
        | [key] -> 
            let filteredPairs = List.filter (fun (curK, _) -> curK <> key) obj
            Object filteredPairs
        | key :: remainingPath -> 
            let updatedObj = 
                obj 
                |> List.map (fun (curKey, value) -> 
                    if curKey = key then 
                        (curKey, delAtPath remainingPath value) 
                    else 
                        (curKey, value))
            Object updatedObj
    | Array ele ->
        let updatedEle = ele |> List.map (fun ele -> delAtPath p ele)
        Array updatedEle
    | _ -> e

let delete ps e =
    ps 
    |> List.fold (fun curE pathToDel ->
        match pathToDel with
        | [] -> failwith "Cannot delete the root object."
        | _ -> delAtPath pathToDel curE) e
```

#### Task 7

Define the function `withPath : Path list -> Ecma -> Ecma list` so that `withPath ps e` evaluates to a list of object representations consisting of those objects in `e` that are represented by a path from the list `ps`.

The result list must respect the ordering requirements from Task 4.

```fsharp
let rec findValAtPath (p : Path) (e : Ecma) : Ecma list =
  match p with
  | [] -> 
    match e with 
    | Object _ -> [e] 
    | Array ele -> 
        ele |> List.collect (fun ele ->
          match ele with 
          | Object _ | Array _ -> findValAtPath [] ele
          | _ -> []) 
    | _ -> []
  | headPath :: tailPath ->
    match e with
    | Object pairs -> 
        pairs |> List.collect (fun (key, value) ->
          if key = headPath then findValAtPath tailPath value else [])
    | Array elements ->
        elements |> List.collect (fun element -> findValAtPath (headPath :: tailPath) element)
    | _ -> []

let withPath (ps : Path list) (e : Ecma) : Ecma list =
  let allPathsInEcma = listPaths e
  let filteredPaths = List.filter (fun path -> List.contains path ps) allPathsInEcma
  filteredPaths |> List.collect (fun path -> findValAtPath path e)
```

----

### Coursework 5

For introduction to FSharpON please check coursework4.fsx for references.

In this coursework we continue with the topic of trees and object notation. This
time the task is, given a description of a set of values in an Ecma,
how to retrieve, modify and delete those values. This is a bit similar
to questions 6 and 7 in coursework 4.

The following material of basic definitions is taken from CW4.

--- Taken from CW4 ---

```fsharp
type Name = string
```

Define data structure(s) for representing FsharpON

```fsharp
type Ecma =
    | Object of (Name * Ecma) list   // Obj: List of key-value pairs
    | Number of float
    | Bool of bool
    | String of string
    | Array of Ecma list
    | Null
```
Define the function `mkObject : unit -> Ecma` that creates a representation for an empty object structure.

```fsharp
let mkObject () = Object[]
```

Define the function `mkNumber : float -> Ecma` that creates a representation for the given floating-point number.

```fsharp
let mkNumber f = Number f
```

Define the function `mkBool : bool -> Ecma` that creates a representation for the given Boolean value.

```fsharp
let mkBool b = Bool b
```

Define the function `mkString : string -> Ecma` that creates a representation for the given string value.

```fsharp
let mkString s = String s
```

Define the function `mkArray : Ecma list -> Ecma` that creates a representation for an array whose elements are represented by the given list of `Ecma` values.

```fsharp
let mkArray ele = Array ele
```

Define the function `mkNull : unit -> Ecma` that creates a representation of the ECMA-404 `null` value.

```fsharp
let mkNull () = Null
```

Define the function `addNameValue : Name * Ecma -> Ecma -> Ecma` so that `addNameValue (n, v) e` evaluates to an ECMA representation `e'` that is:

- equal to e if e is not an object representation
- a representation for the object e extended with the name-value pair (n, v), otherwise.

```fsharp
let addNameValue (n, v) e =
  match e with
    | Object obj -> Object (obj@[(n, v)])
    | _ -> e
```

Define the function `addValue : Ecma -> Ecma -> Ecma` so that `addValue v e` evaluates to an ECMA representation `e'` that is:
- equal to e if e is not an array representation
- a representation for the array e with the value v added as the last element, otherwise.

```fsharp
let addValue v e =
    match e with
    | Array arr -> Array (arr@[v])
    | _ -> e
```

--- End of CW4 material ---

--- Start of CW5---

Given a type of expressions.

```fsharp
type BExpr = True
           | Not      of BExpr
           | And      of BExpr * BExpr
           | Or       of BExpr * BExpr
           | HasKey   of Name
           | HasStringValue of string
           | HasNumericValueInRange of (float*float)
           | HasBoolValue of bool
           | HasNull
```

The type BExpr is just a discriminated union. The intended interpretation of values of type BExpr is as predicates on values of type Ecma.

- **True**: evaluates to true on any Ecma
- **Not b**: evaluates to true on precisely those Ecma for which b evaluates to false
- **And (b1, b2)**: evaluates to true on precisely those Ecma for which both b1 and b2 evaluate to true
- **Or (b1, b2)**: evaluates to true on precisely those Ecma for which at least one of b1 and b2 evaluates to true
- **HasKey k**: evaluates to true on precisely those Ecma that are objects and that contain a key k.
- **HasStringValue s**: evaluates to true on precisely those Ecma that are either Ecma strings with value s, objects that contain a value s,or arrays that contain a value s.
- **HasNumericValueInRange (xmin,xmax)**: evaluates to true on precisely those Ecma that either are:
  - **numeric Ecma** with value in closed range xmin,xmax,
  - **objects** with a numeric value in closed range xmin,xmax,
  - **arrays** with a numeric value in closed range xmin,xmax.
- **HasBoolValue b**: evaluates to true on precisely those Ecma that are either:
    - **Bool Ecma** with value b,
    - **objects** that contain a Boolean value b,
    - **arrays** that contain a Boolean value b.
- **HasNull**: evaluates to true on precisely those Ecma that are either:
    - **Null Ecma**,
    - **objects** that contain a Null value,
    - **arrays** that contain a Null value.

Here is a type of `selector` expressions.

```fsharp
type Selector = Match     of BExpr
              | Sequence  of Selector * Selector
              | OneOrMore of Selector
```

The type Selector is just a discriminated union. The intended interpretation of values of type Selector on values of type Ecma is as sets of values in that Ecma. We also refer to the set of values described by s : Selector as the set of values selected by s.

- **Match b**: 
  
  the singleton set consisting of the root value if the expression b evaluates to true and the empty set otherwise.

- **Sequence (s, s')**:

    the set consisting of those values in the Ecma tree that are selected by the selector s' starting from any child value of a value that is selected by the selector s (starting from the root value).

    In other words, first determine the set of values selected by s (starting from the root value). For every child c of a value in this set, determine the set of values selected by s' (starting from c) and take the union of such sets as the result.

    In other words, start from the root value with the selector s. For the values that it selects, continue with selector s' from their child values and collect the results.

- **OneOrMore s**:
    
    select the values selected by the selector s and, in addition, from the child values of the values in this set select the values selected by OneOrMore s.

    Thus, you can think of the values selected by `OneOrMore s` as the union of the following sets:

    - values selected by s
    - values selected by Sequence (s, OneOrMore s)
  
----

#### Task 1

Translate the following informal descriptions into values of type BExpr and Selector.

Define the values b1, b2 and b3 of type BExpr so that:

- b1 evaluates to true on those Ecma that are object values containing the keys "blue" and "left" but do not have the key "red".

- b2 evaluates to true on those Ecma that are numeric values with the value in the range [-5, 5).

- b3 evaluates to true on those Ecma that have the string value "b3" or that are object values which have the key "b3".

Define the values s1, s2 and s3 of type Selector so that:

- s1 selects all object values with key "abc" that are at depth 3
- s2 selects all values v such that v is a child of some value and all of the ancestors of v have the string value "xyz"
- s3 selects all values v such that:
    - v is a child of a value t
    - t does not have a string value "xyz"
    - t is the root value
  
We consider the root value to be at depth 1.

```fsharp
let b1 = And(HasKey "blue", And(HasKey "left", Not(HasKey "red")))
let b2 = And(HasNumericValueInRange (-5.0, 5.0),(Not(HasNumericValueInRange (5.0, 5.1))))
let b3 = Or(HasStringValue "b3", HasKey "b3")

let s1 = Sequence(Sequence(Match True, Match True), Match (HasKey "abc"))
let s2 = OneOrMore(Match(HasKey "xyz"))
let s3 = Sequence(Match(Not(HasStringValue "xyz")), Match True)
```

----

#### Task 2

Define the function `eval : BExpr -> Ecma -> bool` which evaluates the given expression on the given Ecma.

Evaluating a BExpr only considers properties of the root value of the Ecma and its immediate child values that are leaves (if there are any).

In other words, for any `b : BExpr` and `e : Ecma`

`eval b e = eval b e'`

where `e'` is `e` with all of its non-leaf child values replaced with the representation for null.

```fsharp
let rec eval b e =
    match b with
    | True -> true
    | Not b' -> not (eval b' e)
    | And (b1, b2) -> (eval b1 e) && (eval b2 e)
    | Or (b1, b2) -> (eval b1 e) || (eval b2 e)
    | HasKey k -> 
        match e with
        | Object obj -> List.exists (fun (k', _) -> k = k') obj
        | _ -> false
    | HasStringValue s ->
        match e with
        | String s' -> s = s'
        | Object obj -> List.exists (fun (_, v) -> match v with String s'' -> s = s'' | _ -> false) obj
        | Array arr -> List.exists (fun v -> match v with String s'' -> s = s'' | _ -> false) arr
        | _ -> false
    | HasNumericValueInRange (xmin, xmax) ->
        let epsilon = 1e-15
        match e with
        | Number f -> xmin <= f && f < xmax || (abs (f - xmin) < epsilon)
        | Object obj -> List.exists (fun (_, v) -> 
            match v with 
            | Number f -> xmin <= f && f < xmax || (abs (f - xmin) < epsilon) 
            | _ -> false) obj
        | Array arr -> List.exists (fun v -> 
            match v with 
            | Number f -> xmin <= f && f < xmax || (abs (f - xmin) < epsilon) 
            | _ -> false) arr
        | _ -> false
    | HasBoolValue b' ->
        match e with
        | Bool b'' -> b' = b''
        | Object obj -> List.exists (fun (_, v) -> match v with Bool b'' -> b' = b'' | _ -> false) obj
        | Array arr -> List.exists (fun v -> match v with Bool b'' -> b' = b'' | _ -> false) arr
        | _ -> false
    | HasNull ->
        match e with
        | Null -> true
        | Object obj -> List.exists (fun (_, v) -> v = Null) obj
        | Array arr -> List.exists (fun v -> v = Null) arr
        | _ -> false
```

----

#### Task 3

> 3 Test Cases failed
> nested OneOrMore
> Nested OneOrMore: narrow tree
> OneOrMore in Sequence: large tree

```fsharp
type Description = Key   of string
                 | Index of int

type Path = Description list
```

Define the function `select : Selector -> Ecma -> (Path * Ecma) list` that computes the set of values in the given Ecma described by the given Selector. 

The result is a list of pairs where the second component is a selected value and the first component is the full path to this value.

The path to the root value is the empty list.

If you follow a child value of an object, then you add the key of that value to the path. If you follow a child of an array, then you add the index of that value in the array to the path (the oldest value has index 0).

The order of values in the result list must respect the order of values in the given Ecma. More precisely, in the result list:
- a value must appear before any of its children
- a value must not appear before its older siblings and their descendants

This task is similar to evaluating a BExpr on an Ecma. The difference is that instead of a BExpr we have a Selector and instead of a bool we compute a (Path * Ecma) list. In this case we also consider child values.

```fsharp
let rec select s e =
    let rec matchWithChildren s e path =
        match e with
        | Object fields ->
            fields |> List.collect (fun (k, v) -> selectHelper s v (path @ [Key k]))
        | Array elements ->
            elements |> List.mapi (fun i v -> selectHelper s v (path @ [Index i])) |> List.collect id
        | _ -> []

    and selectHelper s e path =
        match s with
        | Match b -> 
            if eval b e then [(path, e)] else []
        | Sequence (s1, s2) ->
            let firstMatches = selectHelper s1 e path
            firstMatches |> List.collect (fun (p, v) -> matchWithChildren s2 v p)
        | OneOrMore s ->
            let directMatches = selectHelper s e path
            let childMatches = matchWithChildren (OneOrMore s) e path
            List.append directMatches childMatches
    selectHelper s e []
```

----

#### Task 4

> 3 Test Cases failed
> OneOrMore and a large tree
> update spec II
> update spec

