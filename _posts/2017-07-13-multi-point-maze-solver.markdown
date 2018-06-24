---
layout: post
title:  "Multi-point maze solver"
date:   2017-07-13 20:10:00 -0600
categories: software
---
This Java tool is a simple application of the [Lee algorithm](https://en.wikipedia.org/wiki/Lee_algorithm) for (rectilinear) maze routing, applied to a non-grid space using [Bresenham's line algorithm](https://en.wikipedia.org/wiki/Bresenham%27s_line_algorithm).

It was written as part of a course on VLSI algorithms, in Java. The [Processing library](https://processing.org) (3.0.1) allowed quick development of a basic graphic interface. The user can draw a maze and define points within it that should be connected.

<img alt="Screenshot of screen for editing maze" border="0" src="https://mberka.com/_/rsrc/1500001304793/software/multi-point-maze-solver/MazeSetup.jpg" />

The Lee algorithm can then be used to find the shortest path (or an unusually long one, if desired).

<div style="display:block;text-align:left"><img alt="Screenshot of maze with route between two points." border="0" src="https://mberka.com/_/rsrc/1500002819298/software/multi-point-maze-solver/Route2Points.jpg" /></div>

Altering the clearance required by the wires, paths through narrower openings can be enabled:

<div style="display:block;text-align:left"><img alt="Route through maze passing through narrower gaps between barriers." border="0" src="https://mberka.com/_/rsrc/1500002927695/software/multi-point-maze-solver/RouteNarrowWire.jpg" /></div>
<div style="display:block;text-align:left">

<div style="display:block;text-align:left">Large numbers of endpoints can be defined, and the router will attempt to connect each to an existing path.
<div style="display:block;text-align:left"><div style="display:block;text-align:left"><img alt="Route connecting many endpoints in the maze." border="0" src="https://mberka.com/_/rsrc/1500003029041/software/multi-point-maze-solver/RouteManyPoints.jpg" /></div>

Mazes can be imported and exported as segment-based text files, for storage and possible use in other systems.

<div style="display:block;text-align:left"><img alt="Maze import/export screen with maze code." border="0" src="https://mberka.com/_/rsrc/1500003128573/software/multi-point-maze-solver/MazeFileIO.jpg" /></div>

A standalone archive for 32-bit windows is attached. The files in /source contain
the original code, which can be executed in and exported for other operating systems with the Processing environment. Further development is possible if desired.
