---
layout: post
title:  "Tutorial: Isometric (2.5D) Puzzle Game With Corona (Part 1)"
date:   2014-07-25 00:00:00 +1100
permalink: /tutorial-isometric-2-5d-puzzle-game-with-corona-part-1
tags: game_development, corona, lua
---

This tutorial assumes knowledge of:

* Corona SDK
* Tiled Map Editor

Here’s a short video of the game:

[![Corona isometric puzzle game](http://img.youtube.com/vi/C_JbHhReU6o/0.jpg)](http://www.youtube.com/watch?v=C_JbHhReU6o "Corona isometric puzzle game")

You can get the complete code here on Bitbucket: https://bitbucket.org/anthonygore/corona-isometric-map-template

In part 1 of the tutorial we’ll look at making an isometric map with Tiled Map Editor (a free tiled map tool http://www.mapeditor.org)

Firstly, create a new map. You’ll need to set the orientation to “Isometric” and the tile size to whatever you like, so long as the width is twice the height (shortly you’ll see why)

![Tiled Map Editor - New Map](/assets/img/2014-07-25-tme-1.png)

If you go to View > Show Grid, you’ll see the outline of your isometric grid:

![Grid](/assets/img/2014-07-25-grid-3.png)

Now, import your tiles by going Map > New Tileset… . Getting the width/height correct can be tricky, and the settings will be entirely dependent on your particular tileset.

Once you’ve drawn your map, you’ll have something like this:

![Grid](/assets/img/2014-07-25-grid-4.png)

Finally, export your map ready for import into Corona. Go to File > Export As… and choose the .lua file format.

In the next tutorial we’ll look at importing and displaying your isometric map in Corona.
