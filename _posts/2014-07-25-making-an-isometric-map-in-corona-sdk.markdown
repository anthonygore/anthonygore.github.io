---
layout: post
title:  "Making An Isometric (2.5D) Map In Corona SDK"
date:   2014-07-25 00:00:00 +1100
tags: game_development, corona, isometric
permalink: /making-an-isometric-2-5d-map-in-corona-sdk/
-------------------------------------------------------

Think of an isometric (aka 2.5D) map as a regular, top-down map (aka orthogonal map), only it’s tilted at a 45deg angle and slightly squashed to provide that fake-3D effect that makes many games look awesome.

In fact, each point on an orthogonal map has an equivalent point on an isometric map.

This insight is important because it demonstrates the key to working with isometric maps: you can deal with them the same way you deal with orthogonal maps, you’ll just have to have a “conversion” layer between your game logic and the drawing of the map.

Let’s say you have a particular tile (2,2) on a 5 x 5 grid:

    {0,0,0,0,0}
    {0,0,0,0,0}
    {0,0,1,0,0}
    {0,0,0,0,0}
    {0,0,0,0,0}

This same grid can be represented orthogonally:

Or isometrically:

Assuming we have isometric map tiles drawn up, how do we position them on the isometric grid? Let’s compare the calculation of the top-left corner of the tile in both perspectives.

Orthogonal:

    local tileWidth, tileHeight = 30, 30
    local selectedTile = {gridX=2,gridY=2}
    local topCorner = {}
    topCorner.x = selectedTile.gridX * tileWidth
    topCorner.y = selectedTile.gridY * tileHeight
    print("Screen pos of top corner of tile (" .. selectedTile.gridX .. "," .. selectedTile.gridY .. ") is [" .. topCorner.x .. "," .. topCorner.y .. "]")
    
**Output**: Screen pos of top corner of tile (2,2) is [60,60]

Isometric:

    local tileWidth, tileHeight = 60, 30
    local selectedTile = {gridX=2,gridY=2}
    local topCorner = {}
    topCorner.x = (selectedTile.gridX - selectedTile.gridY) * tileWidth/2
    topCorner.y = (selectedTile.gridX + selectedTile.gridY) * tileHeight/2
    print("Screen pos of top corner of tile (" .. selectedTile.gridX .. "," .. selectedTile.gridY .. ") is [" .. topCorner.x .. "," .. topCorner.y .. "]")
    
**Output**: Screen pos of top corner of tile (2,2) is [0,60]

There’s a few things to notice:

* The tile width and height are different in isometric. Commonly, though not necessarily, the width is double the height.
* It might seem odd that tile (2,2) has an x co-ordinate of 0. In this style of isometric projection, the x-axis goes straight down the centre of the grid. That would mean tiles on the left side of the grid will have negative x values. You could add an offset to the map to adjust the map so there are no negative x values.

How is the conversion between ortho/iso points done?

You can read more about it [here](http://clintbellanger.net/articles/isometric_math/), but in summary, the formula is:

    isoX = (orthX - orthY) * tileWidth/2
    isoY = (orthX + orthY) * tileHeight/2
    
If you use Tiled Map Editor to create your maps, you can import them into Corona SDK with [this tool](https://github.com/anthonygore/Isometric-Tiled-Map-Loader).

Also, here’s a simple isometric puzzle game I’ve made in Corona and put on [Bitbucket](https://bitbucket.org/anthonygore/corona-isometric-map-template).
