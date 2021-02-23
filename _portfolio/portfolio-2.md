---
title: "A GPS Based solution, for the dynamic assignement problem"
excerpt: "Operations research for trasportation management"
toc: true
comments: true
collection: portfolio
---

[<img src="/images/kofi.png" alt="Buy me a coffee" height="30">](https://ko-fi.com/hamzaim)  

**Logistics optimization**  
Logistics touches every part of an enterprise and even goes beyond to include suppliers, carriers and customers.  Needless to say, logistics is complicated with many interdependent components.  This high degree of complexity is difficult to manage without the use of models that accurately represent the processes and their interactions.
Reduce costs, and deadheads.  

In this Article, we will apply Hungarian Algorithm, which is an optimization algorithm.

# Modelling the problem
After appliying design research, some  
If you are not familiar with trasnportation problems, you can check this good website, [Here](http://web.tecnico.ulisboa.pt/~mcasquilho/compute/_linpro/index.php)

## Algorithm

We use the Hungarian algorithm, Assign n missions to n available trucks
`E_{rv}`	set of available real vehicles.  
`E_{dv}`	set of dummy vehicles  
`E_s`	set of sources of missions  
`E_d`	set of destinations of missions  


## Cost matrix

Latex markup,  
`c_{vpij}={\rm cf}_v\left({d_{pi}+d}_{ij}\right)+{ca\astd}_{ij}-\ p_{ij}`

with  
`c_{vpij}` the cost to assign the truck v located in p to the task i to j.  
`{\rm cf}_v`	cost per km of fuel consumption for the truck v.  
`{\rm ca}_`	cost per km of the resource (driver) allocation.  
`d_{ij}`	distance traveled to perform task from the source i to the destination j.  
`d_{pi}`	distance traveled to perform task from the truck position p to the source i.  
`b_{ij}`	bonus of driver to performs the mission i to j.  
`p_{ij}`	profit of transportation from the source i to the destination j.  


# Solution

**References**  
https://www.arenasimulation.com/business-processes/logistics-optimization-software
