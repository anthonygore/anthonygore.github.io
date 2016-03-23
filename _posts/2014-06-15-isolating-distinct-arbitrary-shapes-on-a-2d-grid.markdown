---
layout: post
title:  "Isolating Distinct, Arbitrary Shapes On A 2D Grid"
date:   2014-06-15 00:00:00 +1100
tags: corona game_development lua
permalink: /isolating-distinct-arbitrary-shapes-on-a-2d-grid/
-------------------------------------------------------------

Let’s say you have a 2D grid with distinct, arbitrary shapes on it (polygons of any sort), and you’d like to isolate the distinct shapes. For example, this 2D grid could represent a game’s map with a system of different shaped buildings. If your player is on top of one building, the game logic requires the application to distinguish the isolated shapes for pathfinding purposes etc.

E.g.:

    {0,0,0,0,0,0}
    {0,1,1,0,0,0}
    {0,1,1,0,0,1}
    {1,1,0,1,0,1}
    {0,0,0,1,0,1}
    {0,0,1,1,1,1}

You can see two distinct shapes on this grid, one on the top left, the other in the bottom right (we ignore touching diagonals).

To isolate the shapes, iterate the grid left-to-right, top-to-bottom. For each cell (x<sub>i</sub>, y<sub>i</sub>) that has a value vi where i > 0, check the cell directly above i.e. (x<sub>i</sub>, y<sub>i-1</sub>), and directly behind i.e. (x<sub>i-1</sub>, y<sub>i</sub>), where i-1 > 0. If either of these has a value v<sub>i</sub> where i > 0, assign (x<sub>i</sub>, y<sub>i</sub>) the value v<sub>i</sub>. If they don’t, this cell must be part of a new shape. Assign (x<sub>i</sub>, y<sub>i</sub>) the value v<sub>i+1</sub>.

The output will be:

    {0,0,0,0,0,0}
    {0,1,1,0,0,0}
    {0,1,1,0,0,2}
    {1,1,0,2,0,2}
    {0,0,0,2,0,2}
    {0,0,2,2,2,2}
 
As mentioned, this algorithm ignores touching diagonals. In the example, if we considered (x<sub>i</sub>, y<sub>i</sub>) and (x<sub>4</sub>, y<sub>4</sub>) to be touching, then there’d just be a single shape on this grid.

Here’s an implementation in Lua:

    -- og is "old gird"
    -- ng is "new grid"
    local function createPolyObjects (og)
        local objNum = 0
        local ng = {}
        for y = 1, #og, 1 do
            ng[y] = {}
            for x = 1, #og[y], 1 do
                local tile = {}
                tile.val = og[y][x];
                tile.x = x
                tile.y = y
                if tile.val == 1 then
                    --Tile above
                    local tileAbove = {}
                    tileAbove.x = x
                    tileAbove.y = y-1
                    if tileAbove.y > 0 then
                        tileAbove.val = ng[tileAbove.y][tileAbove.x]
                    else
                        tileAbove = nil
                    end
     
                    --Tile behind
                    local tileBehind = {}
                    tileBehind.x = x-1
                    tileBehind.y = y
                    if tileBehind.x > 0 then
                        tileBehind.val = ng[tileBehind.y][tileBehind.x]
                    else
                        tileBehind = nil
                    end
     
                    --Determine the correct objNum to assign
                    if tileAbove == nil or tileAbove.val == 0 then
                        if tileBehind == nil or tileBehind.val == 0 then
                            objNum = objNum + 1
                        end
                    else
                        if tileBehind.val > tileAbove.val then
                            ng = obj.changeObjNums(ng, x, y, objNum, tileAbove.val)
                            objNum = tileAbove.val
                        else
                            objNum = tileAbove.val
                        end
                    end
     
                    --Populate ng
                    ng[y][x] = objNum
                else
                    ng[y][x] = 0
                end
            end
        end
        obj.max = objNum
        return ng
    end
     
    --Helper function:
    --Changes the "object number" used on the grid
    local function changeObjNums (grid, x, y, oldNum, newNum)
        for j = y, 1, -1 do
            for i = x, 1, -1 do
                if grid[j][i] == oldNum then
                    grid[j][i] = newNum
                end
            end
        end
        return grid
    end

This code is part of my [Lua Grid Tools](https://bitbucket.org/anthonygore/luagridtools) repo on Bitbucket.

Next I’ll explain how to break up the polygons into distinct rectangles. This is useful if you’re aiming to add each shape to Box2D, as convex shapes cannot be used in Box2D. Check that out [here]().
