# **Solve algorithm for best teams?**

Anton Antonov  
antononcube@gmail.com   
December, 2016   

## Formulation

For the problem description see [http://community.wolfram.com/groups/-/m/t/972050](http://community.wolfram.com/groups/-/m/t/972050).

I do find the formulation in the discussion opening inconsistent. The constraint formulations below are slightly different from the ones in OP's descriptions. 
The approach allows relatively easily the constraints to be changed or other constraints to be added.

The constraints and objective function were programmed in a way that allows the finding of the number groups for different number of teams and number of courses.

For another, detailed explanation of the used approach see
[this answer](http://mathematica.stackexchange.com/questions/111725/how-to-fill-a-grid-make-its-total-be-largest/112210#112210) of the
Mathematica Stackexchange question ["How to fill a grid make its total be largest"](http://mathematica.stackexchange.com/q/111725/34008). 

### Variables

#### Number of variables

Assuming the number of groups is un-known we can select a large number for `ng` and 
then include the corresponding variables in the conditions and objective function.

Number of teams:

    nt = 9;

Number of groups:

    ng = 6;

Number of courses:

    nc = 3;

#### Variable arrays

    Clear[t, g, c, vt, vg, vc]

Binary variables telling that $i$-th team is going to be used (formed).

    vt = Array[t, nt];

Binary variables telling that $i$-th group is going to be used (formed).

    vg = Array[g, ng];

Binary variable for 

    vc = Array[c, nc];

#### Team in groups

    Clear[ctg, vctg]

Each group has three teams. Each team is assigned to exactly one group per course.

Binary variables:

    vctg = Flatten@Table[ctg[ci, ti, gi], {ci, nc}, {ti, nt}, {gi, ng}];
    Length[vctg]
    (* 162 *)

#### Chefs

    Clear[ch, vch]

Each team can be the chef for a given group and course. There is only one chef per group and course pair.

Binary variable:

    vch = Flatten@Table[ch[ci, ti, gi], {ci, nc}, {ti, nt}, {gi, ng}];
    Length[vch]

    (* 162 *)

#### Happiness

Happiness of team $t_i$ to prepare course $c_j$

    Clear[H, vh]
    vh = Flatten@Table[H[ci, ci], {ci, nc}, {ti, nt}];
    Do[H[ci, ti] = RandomInteger[{0, 3}], {ci, nc}, {ti, nt}]

### Constraints

#### Each team should have 3 courses. 

Each team has eaten 3 ($nc$) courses. 

    eachTeamHadFullMeal = 
      Flatten@Table[Sum[ctg[ci, ti, gi], {gi, ng}, {ci, nc}] == nc, {ti, nt}];
    Length[eachTeamHadFullMeal]
    (*eachTeamHadFullMeal[[1;;2]]*)

    (* 9 *)

#### Each team is assigned to one group per course.

Each team is assigned to one group per course.

    oneGroupPerTeamPerCourse = 
      Flatten@Table[
        Sum[ctg[ci, ti, gi], {gi, ng}] == 1, {ti, nt}, {ci, nc}];
    Length[oneGroupPerTeamPerCourse]
    oneGroupPerTeamPerCourse[[1 ;; 2]]

    (* 27 *)

    (* {ctg[1, 1, 1] + ctg[1, 1, 2] + ctg[1, 1, 3] + ctg[1, 1, 4] + 
    ctg[1, 1, 5] + ctg[1, 1, 6] == 1, 
    ctg[2, 1, 1] + ctg[2, 1, 2] + ctg[2, 1, 3] + ctg[2, 1, 4] + 
    ctg[2, 1, 5] + ctg[2, 1, 6] == 1} *)

#### Each group has three teams.

Each group has three teams.

    threeTeamsPerGroupPerCourse = 
      Flatten@Table[
        Sum[ctg[ci, ti, gi], {ti, nt}] - 3 g[gi] == 0, {gi, ng}, {ci, nc}];
    Length[threeTeamsPerGroupPerCourse]
    threeTeamsPerGroupPerCourse[[1 ;; 2]]

    (* 18 *)

    (* {ctg[1, 1, 1] + ctg[1, 2, 1] + ctg[1, 3, 1] + ctg[1, 4, 1] + 
    ctg[1, 5, 1] + ctg[1, 6, 1] + ctg[1, 7, 1] + ctg[1, 8, 1] + 
    ctg[1, 9, 1] - 3 g[1] == 0, 
    ctg[2, 1, 1] + ctg[2, 2, 1] + ctg[2, 3, 1] + ctg[2, 4, 1] + 
    ctg[2, 5, 1] + ctg[2, 6, 1] + ctg[2, 7, 1] + ctg[2, 8, 1] + 
    ctg[2, 9, 1] - 3 g[1] == 0} *)

#### There can be only one chef per group per course.

There can be only one chef per group per course. Not every team has to be a chef of a course.
Note the conditional dependence upon the `g` values.

    oneChefPerGroupPerCourse = 
      Flatten@Table[
        Sum[ch[ci, ti, gi], {ti, nt}] - 1 g[gi] == 0, {gi, ng}, {ci, nc}];
    Length[oneChefPerGroupPerCourse]
    oneChefPerGroupPerCourse[[1 ;; 3]]

    (* 18 *)

    (* {ch[1, 1, 1] + ch[1, 2, 1] + ch[1, 3, 1] + ch[1, 4, 1] + ch[1, 5, 1] +
     ch[1, 6, 1] + ch[1, 7, 1] + ch[1, 8, 1] + ch[1, 9, 1] - g[1] == 0,
     ch[2, 1, 1] + ch[2, 2, 1] + ch[2, 3, 1] + ch[2, 4, 1] + 
     ch[2, 5, 1] + ch[2, 6, 1] + ch[2, 7, 1] + ch[2, 8, 1] + 
     ch[2, 9, 1] - g[1] == 0, 
     ch[3, 1, 1] + ch[3, 2, 1] + ch[3, 3, 1] + ch[3, 4, 1] + ch[3, 5, 1] +
     ch[3, 6, 1] + ch[3, 7, 1] + ch[3, 8, 1] + ch[3, 9, 1] - g[1] == 0} *)

#### Connect the $\text{ch}$ variables with $\text{ctg}$ variables.

    connectChefTGAndCourseTG = 
      Flatten@Table[-ch[ci, ti, gi] + ctg[ci, ti, gi] >= 0, {ci, nc}, {ti,
          nt}, {gi, ng}];
    Length[connectChefTGAndCourseTG]
    connectChefTGAndCourseTG[[1 ;; 3]]

    (* 162 *)
 
    (* {-ch[1, 1, 1] + ctg[1, 1, 1] >= 0, -ch[1, 1, 2] + ctg[1, 1, 2] >= 0, -ch[1, 1, 3] + ctg[1, 1, 3] >= 0} *)

#### Set any team to be a chef only once.

Set any team to be a chef only once. (I think this means *at most once* given the previous constraint verbal formulation.)

    anyTeamChefAtMostOnce = 
      Table[Sum[ch[ci, ti, gi], {gi, ng}, {ci, nc}] <= 1, {ti, nt}];
    Length[anyTeamChefAtMostOnce]
    anyTeamChefAtMostOnce[[1 ;; 2]]

    (* 9 *)

    (* {ch[1, 1, 1] + ch[1, 1, 2] + ch[1, 1, 3] + ch[1, 1, 4] + ch[1, 1, 5] +
    ch[1, 1, 6] + ch[2, 1, 1] + ch[2, 1, 2] + ch[2, 1, 3] + 
    ch[2, 1, 4] + ch[2, 1, 5] + ch[2, 1, 6] + ch[3, 1, 1] + 
    ch[3, 1, 2] + ch[3, 1, 3] + ch[3, 1, 4] + ch[3, 1, 5] + 
    ch[3, 1, 6] <= 1, 
    ch[1, 2, 1] + ch[1, 2, 2] + ch[1, 2, 3] + ch[1, 2, 4] + ch[1, 2, 5] +
    ch[1, 2, 6] + ch[2, 2, 1] + ch[2, 2, 2] + ch[2, 2, 3] + 
    ch[2, 2, 4] + ch[2, 2, 5] + ch[2, 2, 6] + ch[3, 2, 1] + 
    ch[3, 2, 2] + ch[3, 2, 3] + ch[3, 2, 4] + ch[3, 2, 5] + 
    ch[3, 2, 6] <= 1} *)

#### Team in group cap, less than 4

    teamInGroup = 
      Table[Sum[ctg[ci, ti, gi], {ci, nc}, {gi, ng}] <= 4, {ti, nt}];
    Length[teamInGroup]
    teamInGroup[[1 ;; 2]]

    (* 9 *)

    (* {ctg[1, 1, 1] + ctg[1, 1, 2] + ctg[1, 1, 3] + ctg[1, 1, 4] + 
     ctg[1, 1, 5] + ctg[1, 1, 6] + ctg[2, 1, 1] + ctg[2, 1, 2] + 
     ctg[2, 1, 3] + ctg[2, 1, 4] + ctg[2, 1, 5] + ctg[2, 1, 6] + 
     ctg[3, 1, 1] + ctg[3, 1, 2] + ctg[3, 1, 3] + ctg[3, 1, 4] + 
     ctg[3, 1, 5] + ctg[3, 1, 6] <= 4, 
     ctg[1, 2, 1] + ctg[1, 2, 2] + ctg[1, 2, 3] + ctg[1, 2, 4] + 
     ctg[1, 2, 5] + ctg[1, 2, 6] + ctg[2, 2, 1] + ctg[2, 2, 2] + 
     ctg[2, 2, 3] + ctg[2, 2, 4] + ctg[2, 2, 5] + ctg[2, 2, 6] + 
     ctg[3, 2, 1] + ctg[3, 2, 2] + ctg[3, 2, 3] + ctg[3, 2, 4] + 
     ctg[3, 2, 5] + ctg[3, 2, 6] <= 4} *)

#### All variables are binary

All variables are binary constraints. Needed if `Maximize` is used.

    varConstraints = Map[0 <= # <= 1 &, Join[vctg, vch, vg]];
    varConstraints[[1 ;; 4]]

    (* {0 <= ctg[1, 1, 1] <= 1, 0 <= ctg[1, 1, 2] <= 1, 
      0 <= ctg[1, 1, 3] <= 1, 0 <= ctg[1, 1, 4] <= 1} *)

### Objective function

    objFunc =
      Sum[H[ci, ti] ch[ci, ti, gi], {ti, nt}, {ci, nc}, {gi, ng}];

We can use this objective function in order to minimize the number of groups:

    objFuncMinNG = 
      Sum[H[ci, ti] ch[ci, ti, gi], {ti, nt}, {ci, nc}, {gi, ng}] - Total[vg];     

## Solving with LinearProgramming

Using `Maximize` is very slow, so we have to convert the conditions into matrix-vector formulation to be given to `LinearProgramming`. 

All variables

    vars = Join[vctg, vch, vg];
    Length[vars]

    (* 330 *)

### Convert conditions to matrices

    {zeroMat, mat0} = CoefficientArrays[eachTeamHadFullMeal[[All, 1]], vars];
    Dimensions[mat0]

    {zeroMat, mat1} = 
      CoefficientArrays[oneGroupPerTeamPerCourse[[All, 1]], vars];
    Dimensions[mat1]

    {zeroMat, mat2} = 
      CoefficientArrays[threeTeamsPerGroupPerCourse[[All, 1]], vars];
    Dimensions[mat2]

    {zeroMat, mat3} = 
      CoefficientArrays[oneChefPerGroupPerCourse[[All, 1]], vars];
    Dimensions[mat3]

    {zeroMat, mat4} = 
      CoefficientArrays[connectChefTGAndCourseTG[[All, 1]], vars];
    Dimensions[mat4]

    {zeroMat, mat5} = 
      CoefficientArrays[anyTeamChefAtMostOnce[[All, 1]], vars];
    Dimensions[mat5]

    {zeroMat, mat6} = CoefficientArrays[teamInGroup[[All, 1]], vars];
    Dimensions[mat6]

    bVec =
      Join[
       Table[{nc, 0}, {Dimensions[mat0][[1]]}],
       Table[{1, 0}, {Dimensions[mat1][[1]]}],
       Table[{0, 0}, {Dimensions[mat2][[1]]}],
       Table[{0, 0}, {Dimensions[mat3][[1]]}],
       Table[{0, 1}, {Dimensions[mat4][[1]]}],
       Table[{1, -1}, {Dimensions[mat5][[1]]}],
       Table[{4, -1}, {Dimensions[mat6][[1]]}]
       ];
    Length[bVec]

    condMat = Join[mat0, mat1, mat2, mat3, mat4, mat5, mat6];
    MatrixQ[condMat]

    MatrixPlot[condMat]

 !["condMatPlot"](http://i.imgur.com/Hplnz8cm.png)

    objVec = Normal@CoefficientArrays[{objFunc}, vars][[2]][[1]];
    Length[objVec]
    (* 330 *)


### Solving

    AbsoluteTiming[
     nsol = LinearProgramming[-objVec, condMat, bVec, 
        Table[{0, 1}, {Length[vars]}], Integers];
     ]
    (* {0.063444, Null} *)

    objVec.nsol
    (* 23 *)

    sol = Thread[vars -> nsol];

## Tabulate solution

### Find non-zero groups

    gnzInds = 
      Table[Sum[ctg[ci, ti, gi], {ci, nc}, {ti, nt}] > 0, {gi, ng}] /. sol;
    gnzInds = Pick[Range[ng], gnzInds]

### Tabulation per group
 
This package is for the function `CrossTabulate`.

    Import["https://raw.githubusercontent.com/antononcube/MathematicaForPrediction/master/MathematicaForPredictionUtilities.m"]

In red are the chef team assigments.

    Table[
     Column[{Row[{"group:", gi}], 
       mf = MatrixForm[
         CrossTabulate[
          Flatten[Table[{"team:" <> ToString[ti], 
              "course:" <> ToString[ci], ctg[ci, ti, gi]}, {ti, nt}, {ci, 
              nc}] /. sol, 1]]];
       Do[If[(ch[ci, ti, gi] /. sol) == 1, 
         mf[[1, ti, ci]] = Style[mf[[1, ti, ci]], Red]], {ti, nt}, {ci, nc}];
       mf
       }],
     {gi, gnzInds}]
   
 !["solXTabs"](http://i.imgur.com/Xk9N4Xj.png)

 # Visualize the solution

Here is a visualization of the solution using a graph plot:

     graphEdges = 
      Map[Labeled[("team:" <> ToString[#[[2]]]) -> ("group:" <> ToString[#[[3]]]), 
                   "course:" <> ToString[#[[1]]]] &, 
          Cases[sol, HoldPattern[ctg[___] -> 1], \[Infinity]][[All, 1]]];

     vertices = Union[Flatten[List @@@ graphEdges[[All, 1]]]];

     vcoords =
       Join[
        Block[{t = Flatten@StringCases[vertices, "group:" ~~ ___]}, 
         MapIndexed[# -> 
            0.3 {Cos[#2[[1]] 2 \[Pi]/Length[t]], Sin[#2[[1]] 2 \[Pi]/Length[t]]} &, t]], 
        Block[{t = Flatten@StringCases[vertices, "team:" ~~ ___]}, 
         MapIndexed[# -> 
            0.7 {Cos[#2[[1]] 2 \[Pi]/Length[t]], Sin[#2[[1]] 2 \[Pi]/Length[t]]} &, t]]
        ];

     Legended[
      GraphPlot[List @@@ graphEdges,
       MultiedgeStyle -> All,
         VertexRenderingFunction -> ({If[StringMatchQ[#2, "team:" ~~ __], 
            RGBColor[0.8, 0.8, 1], RGBColor[1, 0.8, 0.8]], EdgeForm[Black],
            Rectangle[# - {0.1, 0.05}, # + {0.1, 0.05}], Black, 
           Text[#2, #1]} &),
       VertexCoordinateRules -> vcoords,
       EdgeRenderingFunction -> (With[{cind = 
             ToExpression[
              StringCases[#3, "course:" ~~ x__ :> x][[1]]]}, {{Green, 
              Brown, Pink}[[cind]], Line[#], Black, 
            Inset[cind, Mean[#], Automatic, Automatic, #[[1]] - #[[2]], 
             Background -> White]}] &),
       ImageSize -> 900],
      Thread[{Green, Brown, Pink} -> Map["course:" <> ToString[#] &, Range[nc]]]]

 !["graphPlot"](http://i.imgur.com/98iqfz2.png)
