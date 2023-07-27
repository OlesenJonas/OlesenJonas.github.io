---
layout: post
title: Procedural Paint Strokes
lead: Hobby Project
# img: /assets/jpg/fluidImg.png
img: https://cdna.artstation.com/p/assets/images/images/049/304/104/large/jonas-olesen-render1.jpg?1652188190
---

Inspired by Arcane I wanted to explore the painterly direction of stylized surfacing.
![stillExample](https://cdna.artstation.com/p/assets/images/images/049/304/104/large/jonas-olesen-render1.jpg?1652188190)
> **_NOTE:_**  Base character from [Blender Studios](https://studio.blender.org/characters/snow/v2/)

The tool is based on a (Blender) Geometry Nodes setup that distributes "Brush Particles" using a give flowmap.
Those particles are then exported as a pointcloud and read by a custom C++/OpenGL program.
![particles](https://cdna.artstation.com/p/assets/images/images/049/304/422/large/jonas-olesen-strokegen.jpg?1652188886)

That program then fills the output texture with those brush strokes using the supplied input textures.
![particleApply](https://cdnb.artstation.com/p/assets/images/images/049/304/427/large/jonas-olesen-colorsample.jpg?1652188895)

The brush stroke geometry can also be used to generate a normal map:
<video width="100%" preload="auto" muted controls autoplay loop>
    <source src="https://cdn.artstation.com/p/video_sources/000/778/670/normalinfluence.mp4" type="video/mp4"/>
</video>

[More Pictures](https://www.artstation.com/artwork/r9Boa6)

Im quite happy with the result already, although it's currently still unclear to me how to best scale this to larger assets without ruining the texel density, as its using unique texture sets at the moment.

The code is private for now, as it is relatively messy, and im planning on porting it inside my (currently still WIP) Vulkan renderer before I continue experimenting with this.