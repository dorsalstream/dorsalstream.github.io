---
layout: post
title: Driving Animations With Kinect
tags: 
- kinect 
- motion retargeting 
- xml3d
- animation
---

Welcome to **Dorsal Stream**. We'll kick off the blog with a post about using Kinect as a motion capture device to drive a humanoid mesh. The main purpose of the post is to talk a bit about the math of *skeletal animations*, document some quirks of *Kinect* and introduce a naive method for *motion retargeting*.
We'll skip over the creation of Humanoid Meshes since there exist a multitude of [tools](http://www.makehuman.org/) and tutorials to get you started with meshes, and dive directly into the preparation required to get your mesh moving about. This would be the rigging step, and we'll go a bit into the math of rigging. While this is not necessary for animating with Blender, it would come in quite handy when retargeting *Kinect* poses onto the mesh.
We'll then discuss about the Kinect tracking data format and the typical *motion retargeting* method, after which we'll present a naive solution to retarget Kinect poses onto a mesh with a skeleton morphologically similar to the *Kinect* skeleton.

-----

### Skeletal Animations (A Blender Perspective)
The typical animation pipeline involves creation of a surface mesh with your favorite tool set (*Blender*, *Maya*, whatnot) and attachment of textures and materials to it. Once that is fine and dandy, you are ready to make your mesh move about, and we arrive at the rigging step.

![mesh_img](assets/mesh.PNG?raw=true)

Stuff about skeletons and rigging
Mesh space and armature space, converting skeletal deformations to mesh space and deforming meshes



### Kinect v2 Skeletal Tracking
How to get data out of it


### Motion Retargeting
How it typically works
Constraints
Naive Method
Drawbacks of the naive method
Drawbacks of retargeting with Kinect


### Download

