---
layout: post
title: Fluid Simulation
lead: Initially implemented for the "Animation and Simulation" course at the University of Koblenz
img: /assets/jpg/fluidImg.png
---

I implemented a [3D eulerian Fluid Simulation](https://github.com/OlesenJonas/Fluid-Sim/tree/main) in OpenGL compute shaders that can be rendered either in real-time (if the quality settings allow for it), or with a fixed timestep.
The result of the latter can be seen here:
<!-- <iframe width="100%" src="https://user-images.githubusercontent.com/45714731/190432458-3c7ecffb-7e99-4087-aaec-62e8f3362b19.mp4" frameborder="0" allowfullscreen> </iframe> -->
<video width="100%" preload="auto" muted controls>
    <source src="https://user-images.githubusercontent.com/45714731/190432458-3c7ecffb-7e99-4087-aaec-62e8f3362b19.mp4" type="video/mp4"/>
</video>

The simulation has various features, such as:

### Advection
- Semi-Lagrangian (reverse) advection
  - Euler or 4th order Runge-Kutte
- Optionally: Back and forth error compensation and correction
![advectSettings](/assets/jpg/advectSettings.png)

### Buoyancy and Gravity

### Pressure Projection
- Basic Jacobi
- Basic Red-Black Gauss-Seidel
![projectSettings](/assets/jpg/projectSettings.png)

A [multigrid](https://github.com/OlesenJonas/Fluid-Sim/tree/multigrid) branch exists that contains the attempt of afterwards adding support for using a multigrid approach to sovle the pressure projection. But im not sure about it's correctness.

### Rendering
is done simply through brute force raymarching following [Ryan Brucks' blog](https://shaderbits.com/blog/creating-volumetric-ray-marcher).