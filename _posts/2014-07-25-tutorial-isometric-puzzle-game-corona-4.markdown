---
layout: post
title:  "Tutorial: Isometric (2.5D) Puzzle Game With Corona (Part 4)"
date:   2014-07-25 00:03:00 +1100
tags: game_development, corona, lua
permalink: /tutorial-isometric-2-5d-puzzle-game-with-corona-part-4/
---

This tutorial assumes knowledge of:

* Corona SDK
* Tiled Map Editor

Here’s a short video of the game:

[![Corona isometric puzzle game](http://img.youtube.com/vi/C_JbHhReU6o/0.jpg)](http://www.youtube.com/watch?v=C_JbHhReU6o "Corona isometric puzzle game")

A critical aspect of isometric games is the ordering within the display group. For example, if one block is “farther away” in terms of map perspective, another block that overlaps this block should be displayed in front of it, to maintain the 3D illusion. You can do this by changing the ordering of display group objects. This concept is explained further in the Corona docs

In isometric perspective, the concept is this: objects that are “nearer” are lower on the screen, and should be at the front of the display group. Object that are “farther” are higher on the screen, and should be at the back of the display group.

Let’s assume your blocks are in a table block. The below function will re-order them (not in the table, but the display group) by calling the method `toFront()` on each object:

    function reorderDisplay()
     
        --Put the blocks into an ordered table
        local ordered = {}
        for i = 1, #block do
            table.insert(ordered, block[i])
        end
     
        --Arranges the blocks based on the smallest y value
        table.sort(ordered,
        function(a, b)
            return a.y < b.y
        end)
     
        --Inserts the blocks into the display group in that order i.e. the
        --smallest goes in first, the largest goes in last.
        for i = 1, #ordered do
            ordered[i]:toFront()
        end
    end

The function firstly puts the blocks into a new table _ordered_. It then sorts the table with the Lua function `table.sort()`, which can have a sorting function passed as it’s second parameter. The function I’ve provided compares the _y_ property of each object to order them smallest to largest.

Once sorted, simply call `toFront()` on each object. Since they’re sorted smallest to largest by their _y_ property, the nearer objects will be displayed in front

In this game, I call the `reorderDisplay()` function in the game loop, but only if a block is moving. That way, any time a block passes another, Corona can figure out which order they should be in. If you don’t call this function, the 3D illusion is lost.

    function onEveryFrame (event)     
        --Check if it needs to reorder the block display
        for i = 1, #block do
            if block[i].moving == true then
                reorderDisplay()
            end
        end
    end
