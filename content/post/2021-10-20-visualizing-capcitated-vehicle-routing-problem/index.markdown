---
title: Capacitated Vehicle Routing Problem
summary: Solving, visualizing VRP using meta-heuristic algorithm
author: Hai Vo
date: '2021-10-20'
slug: []
categories: [Project]
tags: [VRP, shiny, Optimization]
---

<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<script src="{{< blogdown/postref >}}index_files/viz/viz.js"></script>
<link href="{{< blogdown/postref >}}index_files/DiagrammeR-styles/styles.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/grViz-binding/grViz.js"></script>
<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<script src="{{< blogdown/postref >}}index_files/viz/viz.js"></script>
<link href="{{< blogdown/postref >}}index_files/DiagrammeR-styles/styles.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/grViz-binding/grViz.js"></script>
<script src="{{< blogdown/postref >}}index_files/htmlwidgets/htmlwidgets.js"></script>
<script src="{{< blogdown/postref >}}index_files/viz/viz.js"></script>
<link href="{{< blogdown/postref >}}index_files/DiagrammeR-styles/styles.css" rel="stylesheet" />
<script src="{{< blogdown/postref >}}index_files/grViz-binding/grViz.js"></script>

## Objective

In this project, I try to implement:

-   nearest neighbor
-   genetic algorithm
-   simulated annealing

to find the solution for Capacitated Vehicle Routing Problem ([CVRP](https://en.wikipedia.org/wiki/Vehicle_routing_problem)). I using shiny to visualize the algorithm result on several benchmark data set. You can find all my code of shiny app on [github](https://github.com/vohai611/cvrp-genetic). Or visit my app [here](https://shiny.vohai.xyz/cvrp/)

The picture below is a screenshot from the app:
![img](app-screenshoot.png)

## Approach

### Overview

CVRP is a famous problem in optimization fields. The problem ask for an optimal set of routes for a fleet to visit and serves all customer demand with limited vehicles capacity. There is a single depot that has to start and end its tour in there. Every time vehicles back to the depot, its capacity restore.
The following graph is the example of CVRP
<Picture>

This problem is an extend of traveling salesmen problem (TSP) and vehicles routing problem (VRP).

Some of CVRP application in the real world:

-   Product delivery system
-   Distribution of goods to the stores
-   Waste collection

Determining optimal CVRP solution is extremely compute expensive when the number of customers grow up. Therefore, using heuristic and meta heuristic are often more suitable to find near optimal solution compare with using exact algorithm.

TLDR;

### What is heuristic and meta heuristic?

**1. Heuristic**

Heuristic is an algorithm to find approximate solution to a problem. For example: In nearest neighbor heuristic, we continuously go to the nearest customer until we visit all of them. It is not optimal but in several case, could bring an acceptable result.

**2. Meta heuristic**

Meta heuristic is a procedure to search for optimal solution. It treat an objective function as a black box,
giving it input and observe output. Depend on the output, it will trying to modify the input to make improvement.

While heuristic is problem-dependent algorithm. Meta heuristic normally don’t and can apply to multiple problem.

### “Random” Nearest neighbor algorithm

This algorithm work as follows:

<div class="figure" style="text-align: center">

<div id="htmlwidget-1" style="width:672px;height:480px;" class="grViz html-widget"></div>
<script type="application/json" data-for="htmlwidget-1">{"x":{"diagram":"digraph {\n                  graph [layout = dot, rankdir = TB]\n                  node [shape = rectangle]\n                  rec1 [label = \"1. Choose random customer, and visit\"]\n                  rec2 [label = \"2. From there, continuously go to the nearest customer\"]\n                  rec3 [label = \"3. When the vehicles run out of capacity, go back to the depot\"]\n                  rec4 [label = \"4. Repeat the step 1-3 until all customer are visited\"]\n                  rec1 -> rec2 -> rec3 -> rec4\n                  }","config":{"engine":"dot","options":null}},"evals":[],"jsHooks":[]}</script>
<p class="caption">
Figure 1: Nearest neighbor algorithm
</p>

</div>

### Genetic algorithm

Genetic algorithm (GA) is a kind of meta-heuristic. It base on concept of natural evolution by selection.

The implement of GA is show as below:

<div class="figure" style="text-align: center">

<div id="htmlwidget-2" style="width:672px;height:480px;" class="grViz html-widget"></div>
<script type="application/json" data-for="htmlwidget-2">{"x":{"diagram":"digraph {\n                  graph [layout = dot, rankdir = TB]\n                  node [shape = rectangle]\n                  rec1 [label = \"1. Initialize population\"]\n                  rec2 [label = \"2. Caculate objective value\"]\n                  rec3 [label = \"3. Mating pair base on objective value\"]\n                  rec4 [label = \"4. Cross over to produce next generation\"]\n                  rec5 [label = \"5. Mutate offspring gene\"]\n                  rec6 [label = \"6. Insert offspring to population\"]\n                  rec7 [label = \"7. Stop when condtion met\"]\n                  #\n                  rec1 -> rec2 -> rec3 -> rec4 -> rec5 -> rec6 -> rec7\n                  }","config":{"engine":"dot","options":null}},"evals":[],"jsHooks":[]}</script>
<p class="caption">
Figure 2: Genetic algorithm
</p>

</div>

In my shiny App, I using the `{GA}` package to run generic permutation search.

### Simulated annealing

Simulated annealing (SA) is also a meta-heuristic. The idea is borrow from annealing in metallurgy. In short, when heating to a certain temperature and slowly cooling the material, one can alter its physic properties.
In context of mathematics problem, the algorithm is implement as below:

<div class="figure" style="text-align: center">

<div id="htmlwidget-3" style="width:672px;height:500px;" class="grViz html-widget"></div>
<script type="application/json" data-for="htmlwidget-3">{"x":{"diagram":"digraph {\n  graph [layout = dot, rankdir = TB]\n  \n  node [shape = rectangle]        \n  rec1 [label = \"1. Generate a random solution\"]\n  rec2 [label = \"2. Calculate objective function\"]\n  rec3 [label = \"3. Generate a random neighbor solution and calculate it’s objective value\"]\n  rec4 [label = \"4. If neighbor solution is better, move our search to that way. \"]\n  rec5 [label = \"5. If it worse, depend on the current temperature\n our search might still move to its way with a random change.\n The chance is higher when the temperature is high\"]\n  rec6 [label = \"6. Repeat step 3-4 until stop condition meet\"]\n  # edge definitions with the node IDs\n  rec1 -> rec2 -> rec3 -> rec4 -> rec5 -> rec6\n  }","config":{"engine":"dot","options":null}},"evals":[],"jsHooks":[]}</script>
<p class="caption">
Figure 3: Simulated annealing algorithm
</p>

</div>

The key point of SA is that, it move search focus to a worse solution in order to find the better one. This properties help algorithm to escape local optima.  
  

![Escape local optima](SA.jpg)

In the shiny app, my strategies is to generate first solution by nearest neighbor and run SA on all sub-route. The neighbor solution are generated by choose two random point and reverse the path. For example:  

<div class="figure" style="text-align: center">

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-3-1.png" alt="Original solution" width="672" />
<p class="caption">
Figure 4: Original solution
</p>

</div>

<div class="figure" style="text-align: center">

<img src="{{< blogdown/postref >}}index_files/figure-html/unnamed-chunk-4-1.png" alt="Neighbor solution" width="672" />
<p class="caption">
Figure 5: Neighbor solution
</p>

</div>
