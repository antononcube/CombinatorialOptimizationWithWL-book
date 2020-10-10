# Filling and transferring water between containers to obtain a desired volume

*Based on the MSE question https://mathematica.stackexchange.com/q/176654.*

## Questions

A classic class of math puzzles involves two irregular containers, $A$ and $B$, having known volumes $V_A$ and $V_B$ and an infinite source of water (from a spigot, say).  You can 

 - fill either container to its top from the spigot
 - pour the water from one container into the other to the receiver's top
 - pour all the water from one container into the other (if it can hold it)
 - pour out (discard) the entire contents of a container

(You cannot place a mark on any intermediate height/volume of water on either container.) 

The goal is to obtain some exact target volume $V_t$ of water (in either container).  

**Example**

Suppose $V_1 = 7$ and $V_2 = 5$ and the target is $V_t = 4$.  In that case you'd perform the following:

$$
\begin{array}{cc}
A & B \\ \hline
7 & 0 \\
2 & 5 \\
2 & 0 \\
0 & 2 \\
7 & 2 \\
4 & 5 \\
{\bf 4} & {\bf 0} 
\end{array}
$$

and end with four (liters) in container $A$.

**Question**

How would you write a search function in *Mathematica* given $V_A$, $V_B$ and $V_t$ that would compute the (minimal) sequence of steps to obtain $V_t$, or "prove" that $V_t$ could never be obtained?

Test the code on $V_A = 12$, $V_B = 9$ and $V_t = 8$.

**An approach**

Suppose the state at step $t$ is:  `{Va[t], Vb[t]}`

Then the possible states at step $t+1$ are (where `Va` and `Vb` without index are the full volumes):

`{0, Vb[t]}`

`{Va[t], 0}`

`{Va, Vb[t]}` 

`{Va[t], Vb}`

`{Va[t]- (Vb - Vb[t]), Vb}`

`{Va, Vb[t] -(Vb - Va[t])}`

Where the last two require conditions on whether they can be performed.  One could then form a decision tree of possible outcomes and search for a path that leads to the target final condition.  I don't see, though, how one would *prove* that a target volume could never be achieved.

## Answer

I have seen this problem solved with some sort of dynamic programming and lazy evaluation set-up. (And I wanted to come up with some sort of integer programming solution for this problem for some time.)

Below is a general solution based on graphs. With this approach the "proof" for not existing solution is hard to justify. Another thing hard to justify is the "unit" with witch the graph nodes are generated. Still, I think the approach is useful and simple enough.

*(The function definitions are given in the last section.)*

### Outline

1. Make all possible combinations of "reasonable" amounts in A and B.

   - E.g. if VA=5 and VB=9 all amount combinations are generated with unit 1. (Or 1/4.)

2. Consider each generated combination of A and B values to be a graph node. Make a directed graph for those nodes based on the permitted operations.

  - Also, make a dictionary of edge-operation pairs.

3. Find shortest path(s) from a specified start combination to a specified end combination.

4. Display the shortest path as a solution together with the corresponding operations.


### Experiments

#### Edges demo

This shows the graph edges generated with the permitted operations for the node `{1,4}` i.e. `{A,B}=={1,4}` if `{VA,VB}=={7,5}`: 

    MakeEdges[{1, 4}, {7, 5}]
    
    (* {Labeled[{1, 4} -> {0, 4}, "empty-a"] , 
     Labeled[{1, 4} -> {1, 0}, "empty-b"] , 
     Labeled[{1, 4} -> {7, 4}, "fill-a"] , 
     Labeled[{1, 4} -> {1, 5}, "fill-b"] , 
     Labeled[{1, 4} -> {0, 5}, "a-into-b"] , 
     Labeled[{1, 4} -> {5, 0}, "b-into-a"]} *)


This shows the graph edges for the node `{0,4}` i.e. `{A,B}=={0,4}`:

    MakeEdges[{0, 4}, {VA, VB}]
    
    (* {Labeled[{0, 4} -> {0, 0}, "empty-b"] , 
     Labeled[{0, 4} -> {7, 4}, "fill-a"] , 
     Labeled[{0, 4} -> {0, 5}, "fill-b"] , 
     Labeled[{0, 4} -> {4, 0}, "b-into-a"]} *)

#### The example in the question

    DisplaySteps@FindSteps[MakeGraph[7, 5], {4, 0}]
    
    (* {{0, 0}, "fill-a", {7, 0}, "fill-b-from-a", {2, 5}, 
       "empty-b", {2, 0}, "a-into-b", {0, 2}, "fill-a", {7, 2}, 
       "fill-b-from-a", {4, 5}, "empty-b", {4, 0}} *)

    DisplaySteps@FindSteps[MakeGraph[7, 5], {0, 4}]
    
    (* {{0, 0}, "fill-a", {7, 0}, "fill-b-from-a", {2, 5},
       "empty-b", {2, 0}, "a-into-b", {0, 2}, "fill-a", {7, 2},
       "fill-b-from-a", {4, 5}, "empty-b", {4, 0}, "a-into-b", {0, 4}} *)


#### The test in the question

This shows that there is no solution for VA=12, VB=9 and Vt=8 if the nodes of the graph are generated with unit 1:

    DisplaySteps@FindSteps[MakeGraph[12, 9, 1], {0, 8}]
    
    (* {} *)
    
For certain Vt we get solutions:

    DisplaySteps@FindSteps[MakeGraph[12, 9, 1], {0, 3}]
    
    (* {{0, 0}, "fill-a", {12, 0}, "fill-b-from-a", {3, 9}, 
       "empty-b", {3, 0}, "a-into-b", {0, 3}} *)


### Definitions

    Clear[MakeEdges]
    MakeEdges[{a_, b_}, {va_, vb_}] :=
      Block[{dict},
       dict = {
         {{a, b} -> {0, b}, "empty-a"},
         {{a, b} -> {a, 0}, "empty-b"},
         {{a, b} -> {va, b}, "fill-a"},
         {{a, b} -> {a, vb}, "fill-b"}
         };
       If[a <= vb - b,
        dict = Append[dict, {{a, b} -> {0, b + a}, "a-into-b"}]
        ];
       If[b <= va - a,
        dict = Append[dict, {{a, b} -> {b + a, 0}, "b-into-a"}]
        ];
       If[a > vb - b,
        dict = 
         Append[dict, {{a, b} -> {a - (vb - b), vb}, "fill-b-from-a"}]
        ];
       If[b > va - a,
        dict = 
         Append[dict, {{a, b} -> {va, b - (va - a)}, "fill-a-from-b"}]
        ];
       dict = DeleteCases[dict, {{x_, y_} -> {x_, y_}, _}];
       Labeled @@@ dict
       ];
    
    Clear[MakeGraph]
    MakeGraph[VA_, VB_] := MakeGraph[VA, VB, GCD[VA, VB]];
    MakeGraph[VA_, VB_, unit_] :=
      Block[{nodes, allEdges},
       nodes = 
        Flatten[Outer[List, Union[Append[Range[0, VA, unit], VA]], 
          Union[Append[Range[0, VB, unit], VB]]], 1];
       allEdges = Union@Flatten[Map[MakeEdges[#, {VA, VB}] &, nodes]];
       {Graph[allEdges], Association[Rule @@@ allEdges]}
       ];
    
    Clear[FindSteps]
    FindSteps[{gr_Graph, dict_Association}, end : {_, _}] :=
      
      FindSteps[{gr, dict}, {0, 0}, end];
    FindSteps[{gr_Graph, dict_Association}, start : {_, _}, 
       end : {_, _}] :=
      Block[{p},
       Which[
        ! MemberQ[VertexList[gr], start],
        Echo[Row[{"Unknown start state ", start}]]; {},
        
        ! MemberQ[VertexList[gr], end],
        Echo[Row[{"Unknown end state ", end}]]; {},
        
        True,
        p = FindShortestPath[gr, start, end];
        <|"amounts" -> p, 
         "operations" -> Map[dict, Rule @@@ Partition[p, 2, 1]]|>
        ]
       ];
    
    Clear[DisplaySteps]
    DisplaySteps[res_Association] := Riffle @@ res;
