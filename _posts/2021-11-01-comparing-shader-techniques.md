---
layout: post
title: Comparing shader techniques
lead: Bachelor's thesis
img: /assets/jpg/heatmap.png
---

For my Bachelor's thesis I implemented and compared different shading techniques, starting with basic forward and deferred shading and then adding tiled and clustered light culling
as outlined in the [Efficient Real-Time Shading with Many Lights](http://newq.net/publications/more/sa2014-many-lights-course) course at SIGGRAPH Asia 2014.
(Done in OpenGL 4.5+ using compute shaders)

![basiccomparison](/assets/jpg/basicVsCulled.png)
![tiledebug](/assets/jpg/tileDebug.png)

I also investigated more accurate culling methods for the tiled approach, such as HalfZ & Modified HalfZ, and 2.5D culling.

![cullingcompare](/assets/jpg/cullingCompare.png)

Lastly the memory usage of the common "light grid + index list" data structure was compared to a bitmask based approach as presented by [Michal Drobot](https://advances.realtimerendering.com/s2017/index.html#:~:text=Slides%20(With%20Notes)-,Improved%20Culling%20for%20Tiled%20and%20Clustered%20Rendering,-Abstract%3A%20This).