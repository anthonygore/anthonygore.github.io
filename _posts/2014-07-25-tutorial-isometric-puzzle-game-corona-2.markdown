---
layout: post
title:  "Tutorial: Isometric (2.5D) Puzzle Game With Corona (Part 2)"
date:   2014-07-25 00:01:00 +1100
tags: game_development, corona, lua
permalink: /tutorial-isometric-2-5d-puzzle-game-with-corona-part-2/
---

This tutorial assumes knowledge of:

* Corona SDK
* Tiled Map Editor

Here’s a short video of the game:

[![Corona isometric puzzle game](http://img.youtube.com/vi/C_JbHhReU6o/0.jpg)](http://www.youtube.com/watch?v=C_JbHhReU6o "Corona isometric puzzle game")

In [part 1](/tutorial-isometric-2-5d-puzzle-game-with-corona-part-1) of the tutorial I showed how to make an isometric map with Tiled Map Editor. In part 2, we’ll look at loading your map into a Corona project.

You’ll need to have your map exported as a .lua file from Tiled Map Editor. You’ll also need to use a module I’ve worked on called Isometric Tiled Map Editor which you can grab on Github: [https://github.com/anthonygore/Isometric-Tiled-Map-Loader](https://github.com/anthonygore/Isometric-Tiled-Map-Loader). To give credit where it’s due, most of that code was done by Caleb P [here](http://developer.coronalabs.com/code/gridmap), I’ve just modified it to work with isometric maps.

I won’t explain how the module works here, just how to use it. You need to require the module as well as the exported map, load the map via the gridmap tool and insert it into a display group:

    gridmap = require("gridmap")
    mapData = require "map1" --assumes file is in root dir and named map1.lua
    map = gridmap:createMap(mapData)
    local group = display.newGroup()
    group:insert(1, map)
    
Since an isometric map is displayed around the x-axis, you may need to offset the map so it’s centred on the screen. Here’s a simple function to do the job:

    local function mapOffset(x, y)
        for i = 1, #mapData.layers do
            if mapData.layers[i].type == "tilelayer" then
                mapData.layers[i].x = mapData.layers[i].x - x
                mapData.layers[i].y = mapData.layers[i].y - y
            end
        end
        --Remove the map and redraw 
        map:removeSelf() 
        map = gridmap:createMap(mapData) 
        group:insert(1, map)
    end
     
    mapOffset(-5,5)

If you try the code or watch the video, you can see I’ve linked the right D-pad to the map offset function, meaning the user can move the map around during the game. In this game it’s a pretty useless feature, but I’ve left it in there as it may be useful for anyone expanding on the game.

In the next tutorial we’ll look at the game logic.
