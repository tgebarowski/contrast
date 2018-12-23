---
id: 254
title: Distributed Game of Life with OpenMPI
date: 2009-03-01T19:43:06+00:00
author: tomasz
layout: post
guid: http://myitcorner.com/?p=254
excerpt_separator: <!--more-->
permalink: /2009/03/01/distributed-game-of-life-with-openmpi/
categories:
  - Projects
tags:
  - ANSI C
  - Distributed
  - OpenMPI
---
Game of Life is a simple simulation developed by John Conway. It uses a 2 dimensional array of cells, in each every cell can be either &#8220;alive&#8221; or &#8220;dead&#8221;. At each iteration of the process the application logic decides what will be the next status of the cell. This is done basing on the number of adjacent alive cells (including diagonals), and a set of following rules: 

  * If a cell has three neighbors that are alive, the cell will be alive. If it was already alive it
  
    will remain so, and if it was dead it will become alive.
  * If a cell has two neighbors that are alive, there is no change in the state of the cell
  * In all other cases the cell will be dead

<!--more-->

Is is important that cells are wrapping over the game board.

The implementation of the assignment was done using OpenMPI library. The whole game board (matrix of NxN elements) is divided into P-processes in such a way that every process gets a given sub-matrix. Depending on the number of used processes the size of the data processed by a single process can vary.
  
In a typical case a single process is responsible for solving the problem of size N/P, where N stands for the height of the game board. In some cases (when N/P gives some reminder) the portion of data processed by the last process has a different size (reminding).
  
Every process is performing a simulation on its own portion of private data. The neighboring cells at boundaries are exchanged between processes in every iteration step.
  
Finally (in the last iteration) every process calculates the number alive cells and sends it to the root process which reduces the data from all processes and calculates the sum.

In order to compile the application the following set of actions must be invoked:

```bash
./boostrap
./configure
make
```

Application can be downloaded from here:
[gol.tgz](/assets/gol.tgz)