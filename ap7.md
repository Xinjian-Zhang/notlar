### Coursework 7 Property-based testing

---Aditional Files: ---

File 1: FileSystem.fs

```fsharp
module FileSystem

(*
   This module provides a type FsTree. We often refer to elements of
   this type as filesystems. But FsTree is just a datatype of certain
   trees.

   In particular, there is no difference between files and directories
   in this representation. We often refer to a node in the tree as a
   directory.

   This module also provides incomplete definitions of certain
   functions operating on filesystems. Note that you have to define
   the required FsCheck properties against these functions.

   It is also possible to generate a signature file of the module by using 
   the attached project definition. Signature files are explained here:
   https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/signature-files
*)

    type Path = string list 

    type FsTree = { name     : string
                  ; children : FsTree list }



    // Evaluates to true on those FsTree that consist of only the root
    // node (i.e., without any child nodes)

    let isEmpty (fs : FsTree) : bool = failwith "not implemented"


    // The list of paths to all of the directories in the filesystem

    let show (fs : FsTree) : Path list =
        failwith "not implemented"


    // Create a new directory at path p in the silesystem fs. If the
    // directory exists, then return the filesystem as is.

    let create (p : Path) (fs : FsTree) : FsTree =
        failwith "not implemented"


    // Delete the directory at path p in the filesystem fs. If the
    // directory does not exist, then return the filesystem as is. If the
    // path p denotes the root node of the filesystem, then return the
    // filesystem as is.

    let delete (p : Path) (fs : FsTree) : FsTree =
        failwith "not implemented"


```

File 2: FileSystem.fsi

```fsharp
module FileSystem

type Path = string list

type FsTree =
    {
      name: string
      children: FsTree list
    }

val isEmpty: fs: FsTree -> bool

val show: fs: FsTree -> Path list

val create: p: Path -> fs: FsTree -> FsTree

val delete: p: Path -> fs: FsTree -> FsTree
```

File 3: FileSystem.fsproj

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <OtherFlags>--sig:FileSystem.fsi</OtherFlags>
  </PropertyGroup>

  <ItemGroup>
    <Compile Include="FileSystem.fs" />
  </ItemGroup>


</Project>
```
----

Coursework 7: 

```fsharp

#if INTERACTIVE // A directive that makes it possible to to compile the file
                // and still run the interactive console with library dependencies.

#r "nuget: FsCheck, Version=2.16.6"
#load "FileSystem.fs"
#endif

open FsCheck
```

----

#### Task 1 

Define the predicates

`fsTreeWf : FsTree -> bool`

`pathWf   : Path -> bool`

that evaluate to true on precisely those values of FsTree and Path that are well-formed.

The directory names in a well-formed Path cannot be empty.

A value of type FsTree is well-formed when:
   - the path to any directory in the filesystem is well-formed
   - the filesystem is deterministic (for any given path, there is at most one directory in the filesystem with that path)

Define these predicates so that they traverse their input only once.

```fsharp
let uniqueCheck fsTrees=
    let names  = List.map (fun tree -> tree.name) fsTrees
    List.length names = List.length (List.distinct names)

let rec fsTreeWf tree=
    if tree.name = "" then false
    else 
        match tree.children with
        | [] -> true
        | children -> 
            uniqueCheck children &&
            List.forall fsTreeWf children

let pathWf p= 
    p <> [] && not (List.contains "" p)
```

----

#### Task 2

Define an FsCheck property

`createIsWf : Path -> FsTree -> Property`

which checks that creating a well-formed path p in a well-formed filesystem fs results in a well-formed filesystem.

Define this using a conditional property (==>).

```fsharp
let createIsWf p fsTree= 
   (pathWf p && fsTreeWf fsTree) ==> 
      lazy (fsTreeWf (create p fsTree))
```

----

#### Task 3

Define a generator

`wfTrees : Gen<FsTree>`

that generates well-formed filesystems.

Define a generator

`wfPaths : Gen<Path>`

that only generates well-formed paths.

Define these generators in such a way that none of the generated data is wasted (i.e., discarded). In other words, all the data that you (randomly) generate must be used in the the output of the generator.

You may wish to use the predicates defined above to check that the generators indeed only generate well-formed data. Or that the predicates are defined correctly.

```fsharp
let wfTrees: Gen<FsTree> = 
    let genValidName = Gen.sized (fun size -> Gen.resize (max 1 size) Arb.generate<string>)

    let rec generateWfTree depth: Gen<FsTree> = 
        gen {  
            let! name = genValidName
            match depth with
            | 0 -> return { name = name; children = [] }
            | _ -> 
                let! children = Gen.listOf (generateWfTree (depth - 1))
                return { name = name; children = children }
        }
    Gen.sized (fun size -> generateWfTree (size / 4))

let wfPaths: Gen<Path> =
   Arb.generate<Path> |> Gen.filter pathWf
```

----

#### Task 4

Define an FsCheck property

`deleteIsWellFormed : Path -> FsTree -> bool`

which checks that given

`p  : Path`

`fs : FsTree`

we have that the result of deleting p from fs is well-formed.

You may assume that this property is only used with "well-formed" generators (meaning that p and fs are well-formed).

```fsharp
let deleteIsWellFormed path tree =
    let modifiedTree = delete path tree
    fsTreeWf modifiedTree
```

----

#### Task 5

Define an FsCheck property

`createCreates : Path -> FsTree -> bool`

which checks that given

`p  : Path`

`fs : FsTree`

we have that the path p is included (exactly once) in the result of show after we have created the directory p in fs.

```fsharp
let createCreates (newPath: Path) (fsTree: FsTree): bool =
    let updatedTree = create newPath fsTree
    let allPaths = show updatedTree
    let occurrencesOfNewPath = allPaths |> List.filter (fun p -> p = newPath) |> List.length
    occurrencesOfNewPath = 1
```

----

#### Task 6

Define an FsCheck property

`deleteDeletes : Path -> FsTree -> bool`

which checks that given

`p  : Path`

`fs : FsTree`

we have that the path p is not in the result of show after we have deleted the directory p from fs.

You may assume that this property is only used with "well-formed" generators (meaning that p and fs are well-formed).

```fsharp
let deleteDeletes (targetPath: Path) (fsTree: FsTree): bool =
    let updatedTree = delete targetPath fsTree
    let allPathsAfterDeletion = show updatedTree
    not (List.exists (fun p -> p = targetPath) allPathsAfterDeletion)
```

----

#### Task 7

Define this property using the property

`showShowsEverything' : FsTree -> Property`

which checks that given an

`fs : FsTree`

we have that by deleting one by one all of the items in the result of show fs we end up with an empty filesystem.

```fsharp
let showShowsEverything (fsTree: FsTree): bool =
    let rec deleteAllPaths tree paths =
        match paths with
        | [] -> tree
        | firstPath :: remainingPaths ->
            let deletedTree = delete firstPath tree
            deleteAllPaths deletedTree remainingPaths

    let pathsToShow = show fsTree
    let deletedTree = deleteAllPaths fsTree pathsToShow
    isEmpty deletedTree
```

----

#### Task 8

Define an FsCheck property

`createAndDelete : FsTree -> Path -> Path -> Property`

which checks that given

`fs : FsTree`

`p1 : Path`

`p2 : Path`

we have that, if p1 is not a prefix of p2, then

1. creating directory p1 in fs
2. creating directory p2 in the result
3. deleting p1 from the result

produces a filesystem that still contains p2.

You may assume that this property is only used with "well-formed" generators (meaning that fs, p1 and p2 are well-formed).

```fsharp
let rec isNotPrefix (prefix: Path) (fullPath: Path): bool =
    match prefix, fullPath with
    | pHead :: pTail, fHead :: fTail -> 
        if pHead <> fHead then true else isNotPrefix pTail fTail
    | pHead :: pTail, [] -> true
    | _, _ -> false

let createAndDelete (fs: FsTree) (fstPath: Path) (sndPath: Path) =
    (isNotPrefix fstPath sndPath && pathWf fstPath && pathWf sndPath)
    ==> lazy (fs |> create fstPath |> create sndPath |> delete fstPath |> show |> List.contains sndPath)
```