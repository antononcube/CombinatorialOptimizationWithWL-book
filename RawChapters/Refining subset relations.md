
## Problem definition

https://mathematica.stackexchange.com/q/99274/34008

## Solution 

I have some sort of a brute force solution with combinatorial optimization for the test problem. Note that the solution does not use all permutations, only the ones for which the permutations-to-values relationships are known. For the general problem the constraints have to be modified to accomodate the union and complement of the permutations. (I need to think about the problem further...)

The general idea is to use variables x[i,j] that have values 0 or 1. The value of x[i,j] is 1 if to the i-th permutation is assigned the j-th value. We form constraints based on the given equations, and for a fixed i0 we allow only one x[i0,j] to be 1.

I am using `Maximize` \ `Minimize` below, but the optimization function and the constriants can be easily recast into a vector and a matrix form respectively. (I.e. amenable for `LinearProgramming`. See [my answer](http://mathematica.stackexchange.com/a/112210/34008) to ["How to fill a grid make its total be largest"](http://mathematica.stackexchange.com/questions/111725/how-to-fill-a-grid-make-its-total-be-largest).)

First we map all known permutations into indecies and make the corresponding replacement rules.

    perms = Union[Flatten[permsToMultiSetVals[[All, 1]], 1]];
    permToIndexRules = Thread[perms -> Range[Length[perms]]];
    indexToPermRules = Reverse /@ permToIndexRules;
    permToIndexRules

[![enter image description here][1]][1]

We do similar mapping for the values.

    msVals = Union[Flatten[permsToMultiSetVals[[All, 2]]]];
    msValToIndexRules = Thread[msVals -> Range[Length[msVals]]];
    indexToMsValRules = Reverse /@ msValToIndexRules;
    msValToIndexRules

[![enter image description here][2]][2]

The following function makes constraints from one line of the given permutations-to-values relationships. 

    Clear[MakeConstraints]
    MakeConstraints[{perms_, vals_}, varName_Symbol] :=
      MakeConstraints[perms, vals, varName];
    MakeConstraints[perms_, vals_, varName_Symbol] :=
      Block[{t = Tally[vals], pInds, vars},
       t[[All, 1]] = t[[All, 1]] /. msValToIndexRules;
       pInds = perms /. permToIndexRules;
       vars = Outer[varName[#2, #1] &, t[[All, 1]], pInds];
       MapThread[Total[#2] == #1 &, {t[[All, 2]], vars}]
      ];

Example of constraints made for one of the known permutations-to-values relationships:

    permsToMultiSetVals[[1]]
    MakeConstraints[permsToMultiSetVals[[1]], x]

[![enter image description here][3]][3]

Equations made from all of the known relationships:

    eqs = Flatten[MakeConstraints[#, x] & /@ permsToMultiSetVals];
    ColumnForm[eqs]

[![enter image description here][4]][4]

All created variables:

    vars = Union[Cases[eqs, x[___], \[Infinity]]]

[![enter image description here][5]][5]

Unique mapping per permutation constraints:

    uniqueConstraints = 
     Map[Total[Cases[vars, x[#, _]]] == 1 &, Union[vars[[All, 1]]]]

[![enter image description here][6]][6]


Positivity constraints:

    varConstraints = Map[0 <= # <= 1 &, vars]

[![enter image description here][7]][7]


Find a solution:

    sol = Maximize[Join[{Total[vars]}, eqs, uniqueConstraints, varConstraints], vars, Integers]

[![enter image description here][8]][8]


Replace the corresponding permutations and values:

    varsToOne = Select[sol[[2]], #[[2]] == 1 &][[All, 1]];
    vsol = Map[(#[[1]] /. indexToPermRules) -> (#[[2]] /. indexToMsValRules) &, 
       varsToOne];
    ColumnForm[vsol]

[![enter image description here][9]][9]
Verification:

    TableForm[Transpose@{permsToMultiSetVals[[All, 1]], 
       permsToMultiSetVals[[All, 2]], permsToMultiSetVals[[All, 1]] /. vsol}, 
     TableHeadings -> {None, {"permutations", "multiset values", 
        "solution replacements"}}, TableDepth -> 2]

[![enter image description here][10]][10]


  [1]: http://i.stack.imgur.com/YsUWQ.png
  [2]: http://i.stack.imgur.com/b0xRT.png
  [3]: http://i.stack.imgur.com/cmN8K.png
  [4]: http://i.stack.imgur.com/dYMK5.png
  [5]: http://i.stack.imgur.com/dse34.png
  [6]: http://i.stack.imgur.com/o3Ngb.png
  [7]: http://i.stack.imgur.com/xMGbo.png
  [8]: http://i.stack.imgur.com/9YWa6.png
  [9]: http://i.stack.imgur.com/t9LWK.png
  [10]: http://i.stack.imgur.com/rBNre.png
