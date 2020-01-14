
## Problem formulation

https://mathematica.stackexchange.com/q/134218/34008

## Solution 

Here is another solution using linear programming (well, "just" `Maximize`) in the spirit of these answers: ["How to fill a grid make its total be largest"](http://mathematica.stackexchange.com/a/112210/34008), ["LinearProgramming approach for 'best teams' algorithm"](http://community.wolfram.com/groups/-/m/t/975898). 

The advantage of this approach is that it gives a lot of flexibility to impose solutions characteristics through the constraints.

Below first we do the bishops problem, then we do the proposed generalization in the question.

### Plotting functions



----------

    Clear[ChessBoard]
    ChessBoard[n_Integer] := Table[Mod[i + j, 2], {i, n}, {j, n}];



----------

    Clear[PlotChessBoardSolution]
    PlotChessBoardSolution[n_, {}, opts : OptionsPattern[]] := 
      ArrayPlot[ChessBoard[n], opts, Mesh -> All];
    PlotChessBoardSolution[n_, solCoords : {{_Integer, _Integer} ..}, 
       opts : OptionsPattern[]] :=
      
      ArrayPlot[ChessBoard[n], opts, Mesh -> All, 
       Epilog -> {Red, PointSize[0.04], 
         Point[# - {1/2, 1/2} & /@ solCoords]}];

### Bishops placement

#### Constraints

----------

    Clear[DiagonalCells]
    DiagonalCells[{i_Integer, j_Integer}, v_, n_Integer] :=
      Union@Select[
        Flatten[Map[Table[{i, j} + k*#, {k, 0, n}] &, {v, -v}], 1], 
        n >= #[[1]] > 0 && n >= #[[2]] > 0 &];


----------

    Clear[LeftRightDiagonalContraint, RightLeftDiagonalContraint]
    LeftRightDiagonalContraint[x_Symbol, {i_Integer, j_Integer}, 
       n_Integer] :=
      Total[x @@@ DiagonalCells[{i, j}, {1, 1}, n]] <= 1;
    RightLeftDiagonalContraint[x_Symbol, {i_Integer, j_Integer}, 
       n_Integer] :=
      Total[x @@@ DiagonalCells[{i, j}, {-1, 1}, n]] <= 1;

Example:

    LeftRightDiagonalContraint[x, {1, 2}, 8]

    (* x[1, 2] + x[2, 3] + x[3, 4] + x[4, 5] + x[5, 6] + x[6, 7] + x[7, 8] <= 1 *)
 
#### Solution (with Maximize)

Chess board size:

    n = 8;

Variables:

    Clear[x]
    vars = Flatten@Array[x, {n, n}];

Binary constraints:

    binConstr = Map[0 <= # <= 1 &, vars];

Diagonal constraints:

    cs = Union@
       Join[
        Table[LeftRightDiagonalContraint[x, {1, j}, n], {j, n}],
        Table[LeftRightDiagonalContraint[x, {j, 1}, n], {j, n}],
        Table[RightLeftDiagonalContraint[x, {1, j}, n], {j, n}],
        Table[RightLeftDiagonalContraint[x, {j, n}, n], {j, n}]
        ];
    Length[cs]

    (* 30 *)

For the bishops problem the solution is found fairly quickly using `Maximize`. Note that we can put additional constraints of the form `x[i,j]==0` or `x[i,j]>0` in order to derive different solutions.


    AbsoluteTiming[
     sol = Maximize[Join[{Total[vars]}, cs, binConstr(*,{x[2,1]==0}*)], 
        vars, Integers];
     ]
    sol[[1]]

   (* {0.19802, Null}

   14 *)

Getting the solution coordinates:


    solCoords = List @@@ Select[sol[[2]], #[[2]] > 0 &][[All, 1]]

    (* {{1, 3}, {1, 4}, {1, 5}, {1, 6}, {2, 1}, {2, 8}, {7, 1}, {7, 8}, {8, 1}, {8, 3}, {8, 4}, {8, 5}, {8, 6}, {8, 8}} *)

Plot the solution:

    PlotChessBoardSolution[n, solCoords, ImageSize -> Small]

[![enter image description here][1]][1]

### Proposed generalization

#### Constraints

    Clear[CoordinatesConstraint]
    CoordinatesConstraint[x_Symbol, {i_Integer, j_Integer}, n_Integer] :=
        Block[{t},
       t = Tuples[Range[n], 2];
       t = Select[t, #[[1]] - #[[2]] == i - j || #[[1]] + #[[2]] == i + j &];
       If[Length[t] <= 1, {}, Total[x @@@ Union[Append[t, {i, j}]]] <= 1]
       ];

Example:

    CoordinatesConstraint[x, {4, 5}, 8]

    (* x[1, 2] + x[1, 8] + x[2, 3] + x[2, 7] + x[3, 4] + x[3, 6] + x[4, 5] + x[5, 4] + x[5, 6] + x[6, 3] + x[6, 7] + x[7, 2] + x[7, 8] + x[8, 1] <= 1 *)

#### Solution (with Maximize)

    n = 8;

    Clear[x]
    vars = Flatten@Array[x, {n, n}];

    binConstr = Map[0 <= # <= 1 &, vars];

    cs = Union@
       Flatten@Table[
         CoordinatesConstraint[x, {i, j}, n], {i, n}, {j, n}];
    cs // Length

    (* 62 *)


    AbsoluteTiming[
     sol = Maximize[Join[{Total[vars]}, cs, binConstr], vars, Integers];
     ]
    sol[[1]]

    (* {31.9932, Null}

     2 *)

    solCoords = List @@@ Select[sol[[2]], #[[2]] > 0 &][[All, 1]]

    (* {{3, 8}, {6, 6}} *)

    PlotChessBoardSolution[n, solCoords, ImageSize -> Small]

[![enter image description here][2]][2]


  [1]: https://i.stack.imgur.com/wUCs0.png
  [2]: https://i.stack.imgur.com/1K0VL.png
