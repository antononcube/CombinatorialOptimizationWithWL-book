
## Problem definition

https://mathematica.stackexchange.com/q/111725/34008

## Solution

Below is given a solution derived with ILP combinatorial optimization:
The total of the assigned values to the $5 \times 5$ table is $61$.

I called in the comments this approach to be "brute force" because of the generation of a larger number of variables and conditions and pushing them to `Maximize` or `LinearProgramming`. Same approach was used for [my answer](http://mathematica.stackexchange.com/a/99827/34008) in the discussion ["Refining subset relations"](http://mathematica.stackexchange.com/questions/99274/refining-subset-relations).

## Integer programming formulation 

### Set-up parameters
 
Dimensions of the $m \times n$ table:

    {m, n} = {5, 5};

We consider placing the integers $[1,\dots,d_{max}]$, $d_{max} = 4$ in the $m \times n$ table.

    dmax = 4;

### Variables

Let us make the binary variables $x(k,i,j)=x_{k,i,j}$, $k \in [1,\dots,d_{max}]$, $i \in [1,\dots,m]$, $j \in [1,\dots,n]$ in the following way: $x(k,i,j) = 1$ if the integer $k$ is placed at position $(i,j)$ and it is 0 otherwise.

    ClearAll[vars, x]
    vars = Flatten[Array[x, {dmax, m, n}]];

### Neighbor conditions

Let us define a function (as described in the question) that brings a set of index pairs for the neighbors of given cell $(i,j)$:

$ninds(i,j):= \{ 0 < p_1 \leq m, 0 < p_2 \leq n : p \in \{ (i-1,j),(i+1,j),(i,j-1),(i,j+1)\} \}$ .


    NeighborIndexes[{i_, j_}] := {{i - 1, j}, {i + 1, j}, {i, j - 1}, {i, j + 1}};
    NeighborIndexes[{i_?NumberQ, j_?NumberQ}, {m_, n_}] := 
      Select[{{i - 1, j}, {i + 1, j}, {i, j - 1}, {i, j + 1}}, 
       0 < #[[1]] <= m && 0 < #[[2]] <= n &];

For each cell $(i,j)$ and an integer $k > 1$ placed on that cell we have the conditions:

$\sum_{p \in ninds(i,j)} x(d,p_1,p_2) \geq 1, d \in [1,\dots,k-1]$.

For example, for the integer 4 and $(i,j)$ such that $ninds(i,j)$ has all four neigbors we have :

[![enter image description here][1]][1]

Note that by the nature of the definition of the variables $x(k,i,j)$ we can re-write the neighbor conditions as:

$\sum_{p \in ninds(i,j)} x(d,p_1,p_2) \geq x(k,i,j), d \in [1,\dots,k-1]$.

(This is a very convenient way to keep the maximization problem linear.)

### Neighbor conditions generation

Let us generate the conditions. This can be done in several ways.

    CellConditions[k_, {i_, j_}, {m_, n_}] := 
      Table[Total[Map[x @@ Prepend[#, d] &, NeighborIndexes[{i, j}, {m, n}]]] - x[k, i, j] >= 0, {d, 1, k - 1}];

Example of this function:

    CellConditions[4, {3, 3}, {m, n}]
    
    (* {
     x[1, 2, 3] + x[1, 3, 2] + x[1, 3, 4] + x[1, 4, 3] - x[4, 3, 3] >= 0, 
     x[2, 2, 3] + x[2, 3, 2] + x[2, 3, 4] + x[2, 4, 3] - x[4, 3, 3] >= 0, 
     x[3, 2, 3] + x[3, 3, 2] + x[3, 3, 4] + x[3, 4, 3] - x[4, 3, 3] >= 0}
    *)

Generating all neighbor conditions:

    neighborConds = 
      Flatten@Table[
        CellConditions[d, {i, j}, {m, n}], {d, 2, dmax}, {i, 1, m}, {j, 1,
          n}];
    neighborConds // Length
    
    (* 150 *)

### Other constraints

In order to finish the formulation two other types of constraints have to be added.

**1.** Uniqueness constraints. (Only one integer is assigned per cell.)

    uniqueConstraints = 
      Map[Total[Cases[vars, x[_, #[[1]], #[[2]]]]] == 1 &, 
       Flatten[Table[{i, j}, {i, 1, m}, {j, 1, n}], 1]];
    uniqueConstraints // Length
    
    (* 25 *)

**2.** Positivity / bounded-ness constraints:

    varConstraints = Map[0 <= # <= 1 &, vars];

Because of the uniqueness constraints we do not need to specify $\leq 1$, but I have put it there as a reminder.

### Solution with Maximize (too slow)

At this point we can find the solution with `Maximize`:

    sol = Maximize[
      Join[{vars.vars[[All, 1]]}, neighborConds, uniqueConstraints, 
       varConstraints], vars, Integers]

Using `Maximize` though is **too slow**. I was able to get solutions only for smaller tables and number of integer values to be assigned, e.g. a $3 \times 4$ table and $d_{max} = 3$.

To get the solutions faster we can formulate the problem through vectors and matrices and use `LinearProgramming`.

## Integer Linear Programming formulation

### Convert from symbolic to matrix formulation

    {zeroMat, neighborCondsMat} = 
      CoefficientArrays[neighborConds[[All, 1]], vars];
    Dimensions[neighborCondsMat]
    
    (* {150, 100} *)
    
    {zeroMat, uniquenessCondsMat} = 
      CoefficientArrays[uniqueConstraints[[All, 1]], vars];
    Dimensions[uniquenessCondsMat]
    
    (* {25, 100} *)

    bVec =
      Join[
       Table[{0, 1}, {Dimensions[neighborCondsMat][[1]]}], 
       Table[{1, 0}, {Dimensions[uniquenessCondsMat][[1]]}]
       ];
    
    condMat = 
     Join[Normal[neighborCondsMat], Normal[uniquenessCondsMat]];
    MatrixQ[condMat]
    
    (* True *)

### Solution with `LinearProgramming`

Using `Table[{0, 1}, {Length[vars]}]` as a fourth argument is not necessary because of the uniquness conditions.

    AbsoluteTiming[
     lpSol = LinearProgramming[-vars[[All, 1]], condMat, bVec, 0, Integers]
    ]

    (* Out[74]= {44.4484, {1, 0, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 0, 1, 1, 0, 1, 
      0, 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 
      0, 1, 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 1, 0, 
      0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 0, 1, 0, 0, 1, 0, 0, 1, 0, 1, 0, 0, 0, 0, 0, 
      1, 0, 0}} *)

    lpSol = Thread[vars -> lpSol];
    
    vars[[All, 1]].lpSol[[All, 2]]
    
    (* 61 *)

### Visualize the solution

    pSol = Select[lpSol, #[[2]] == 1 &];
    solMat = SparseArray[Map[Rest[#] -> First[#] &, List @@@ pSol[[All,1]]]];
    MatrixPlotWithValues[solMat]

[![enter image description here][2]][2]

(The definition of the function `MatrixPlotWithValues` is given below.)

## Other solutions

### $5 \times 6$ table

I tried the code above for a $5 \times 6$ table and got the following result after 422 seconds (10 times longer than for $5 \times 5$) on the same computer:

[![enter image description here][3]][3]

The total is $74$.

### $6 \times 6$ table

Because of a comment by @garej I computed the layout for a $6 \times 6$ table (7182 seconds on the same computer):

[![enter image description here][4]][4] 

The total is $90$.
 
## Solution visualization function

Here is the function used for the plots above:

    MatrixPlotWithValues[mat_?MatrixQ] :=
      Block[{gr, m, n},
       {m, n} = Dimensions[mat];
       gr = MatrixPlot[mat];
       Graphics[{gr[[1]], 
         MapThread[
          Text, {Flatten[Transpose@Reverse@mat], 
           Flatten[Table[{i, j} - 1/2, {i, n}, {j, m}], 1]}]}, 
        Frame -> True, 
        FrameTicks -> {Table[{i - 1/2, i}, {i, n}], 
          Table[{j - 1/2, m - j + 1}, {j, m}]}]
       ];

## Generalizations

The solution can be easily adapted for possible generalizations of the problem formulation with tables that are one of:

1. 3D cube,
2. surface of a 3D cube,
3. cylinder,
4. torus,
5. Mobius strip.


  [1]: http://i.stack.imgur.com/JuaTj.png
  [2]: http://i.stack.imgur.com/sCZBvm.png
  [3]: http://i.stack.imgur.com/HaPBTm.png
  [4]: http://i.stack.imgur.com/6Ani9m.png
