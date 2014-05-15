---
layout: post
title:  "Isometric deferred shading"
date:   2009-07-06 19:59:34
categories: C# Silverlight Shaders Hlsl Graphics
---

One idea I had for using pixel shaders in modern RIA technologies goes as follows. Since we do not have real 3D support for now in either of those technologies ( I do not count Software 3D engines like Papervision since they simply are not up to the task of game 3D graphics ), what can we do with shaders and 2D graphics beyond simple filters ? In an isometric tile engine which are quite popular for RIA game development, we could have for example a second sprite that encodes the relative distance to the camera ( aka depth or z-depth ) for each pixel in the tile. Add a third sprite layer that stores the per pixel normal of the tile, and you can reconstruct the world space pixel position in a post processing shader ( similar to what deferred renderers are doing in 3D ) as well as get the normal vector. And now you can solve the whole lighting equation in any way you want in 3D even though there are only 2D assets used. I have to play with the implementation details, but first results look really promising.
