---
layout: post
title: Terrain and Fog Rendering
lead: Created for the "Real-Time Rendering" course at the University of Koblenz
img: /assets/jpg/terrainAndFog.png
---

![terrainAndFog](/assets/jpg/terrainAndFog.png)

# Terrain

I was responsible for [implementing](https://github.com/OlesenJonas/OpenGLFramework/tree/Terrain) the [Concurrent Binary Tree](https://onrendering.com/data/papers/cbt/ConcurrentBinaryTrees.pdf) data structure to manage/update the terrain mesh, as well as the material/shader setup for rendering it.
All of the logic for updating the terrain mesh is implemented in a series of compute shaders following the paper.
![cbtCompute](/assets/jpg/CBTcompute.png)

> **_NOTE:_**  The sum reduction pass is far from optimized!

<iframe width="640" height="360" src="https://www.youtube.com/embed/H6MFzpOfDAo?rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

<iframe width="640" height="360" src="https://www.youtube.com/embed/Tj3brwUCezw?rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

For the terrain shader I wanted to try out an index-Map based approach inspired by a GDC Presentation about the [terain in Ghost Recon Wildlands](https://666uille.files.wordpress.com/2017/03/gdc2017_ghostreconwildlands_terrainandtechnologytools-onlinevideos1.pdf) where each texel in an unsigned integer texture holds an index into a material array. The most significant bit indicates whether or not the area represented by the texel is sloped enough to require biplanar mapping.

![matIndices](/assets/jpg/matIndices.png)
Sampling the material textures according to that index then results in:
![matPatchSingle](/assets/jpg/materialPatchesSingle.png)
And interpolating materials based on the nearest four indices: 
![matPatchBlend](/assets/jpg/materialPatchesBlend.png)
![matPatchBlend](/assets/jpg/materialPatchesShaded.png)

Since there was no virtual texturing solution in place, and the results of this blend couldnt be cached at a high enough resolution without it, the full logic needed to be run for every pixel of the terrain every frame. Since thats obviously quite expensive some architectural changes needed to be made.
Simply switching from forward to deferred rendering wasnt enough since shading was less of a bottleneck compared to overdraw resulting in lots of wasted material logic.
Because of that I decided to implement a "UV-Buffer"-like solution, storing the non-displaced vertex positions (since that is what was used for mapping the textures) aswell as their quad derivatives (packed into a 1 additional texture).
{% highlight glsl %}
vec3 dPdx = dFdx(worldPosNoDisplacement);
vec3 dPdy = dFdy(worldPosNoDisplacement);

uint packeddX = packHalf2x16(vec2(dPdx.x, dPdy.x));
uint packeddY = packHalf2x16(vec2(dPdx.y, dPdy.y));
uint packeddZ = packHalf2x16(vec2(dPdx.z, dPdy.z));
{% endhighlight %}
That guarantees that not just the shading but also the material texture fetches only happen for visible fragments.

I also tried to be smart and sort the pixels into groups depending on how many texture fetches need to happen, in an attempt to reduce divergence inside the workgroups.
But that turned out to be overengineering as the lack of high frequency detail in the index-map meant that most pixels only needed to go through one of the two fast passes anyways and divergence wasnt really a problem to begin with:
{% highlight glsl %}
MaterialAttributes attributes;
//Check if pixel requires just a single material lookup
if(materialIDs[0] == materialIDs[1] && materialIDs[1] == materialIDs[2] && materialIDs[2] == materialIDs[3])
{
    if((materialIDs[0] & 128u) == 128u)
    {
        attributes = biplanarSampleOfMaterialAttributesFromID(materialIDs[0] & 0x7F, ...);
    }
    else
    {
        attributes = flatSampleOfMaterialAttributesFromID(materialIDs[0] & 0x7F, ...);
    }
}
{% endhighlight %}

Nevertheless, it was a massive performance win. Especially with small triangle sizes and view-angles where the binary tree structure resulted in mostly back-to-front sorted triangles.

<iframe width="640" height="360" src="https://www.youtube.com/embed/LW_2yzsLCcM?rel=0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
<br/><br/>

# Fog
The main feature is a multiple-scattering approximation on top of an analytic exponential height fog.
It works by storing any light that is "lost" due to out-scattering in an additional rendertarget which is then blurred in the same way as bloom in [Call of Duty Advanced Warfare](https://www.iryoku.com/next-generation-post-processing-in-call-of-duty-advanced-warfare) and finally added back to the "basic" fog, weighted by the amount of fog along the view vector.\
The result of this can best be seen in a test environment with more depth variation:

without in-scattering:
![fog1_1](/assets/jpg/fog1_1.png)
with in-scattering:
![fog1_2](/assets/jpg/fog1_2.png)
without in-scattering:
![fog2_1](/assets/jpg/fog2_1.png)
with in-scattering:
![fog2_2](/assets/jpg/fog2_2.png)