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

![mesh_img](assets/meshy.png?raw=true#mid)
*Example Mesh From MakeHuman*

Surface meshes, such as the one above, are typically composed of several thousand vertices. Rendering entails positioning objects in the scene coordinate system, positioning the camera and the lights, and rendering from the point of view of the camera. Animating the scene calls for knowledge of the position of all of the vertices of all of the objects in all the frames, which is a lot of information for an animator to sanely handle.

Additionally, in real life you don't have bits and pieces of your flesh moving about independently, most moving loosely-firmly attached to your skeleton. The degrees of freedom of the system are constrained (to a value wayyy lower than the number of vertices in the object/mesh), and is achieved by defining a set of additional rigid bodies (bones), connected via joints as a kinematic chain. Each vertex is then bound to one or more of these rigid bodies (bones) and transformations of all vertices can now be specified by just specifying the transformations of these bones.

![linkage](assets/rig.png?raw=true#mid)
*Custom Skeleton for the MakeHuman Mesh Above*

#### Rigging
Riggging begins by defining the bones and joints that would form the skeleton system which would dictate how the character can move. Specifying a skeleton system calls for defining the lengths of the rigid bodies and the connectivity of the 

> Recommended reading [Character Rigging, Deformations, and Simulations in Film and Game Production](http://webstaff.itn.liu.se/~perla/Siggraph2011/content/courses/mclaughlin.pdf), SIGGRAPH 2011 Course on the types of rigs, designing control systems on top of rigs, deforming meshes attached to rigs to fake muscles and so on. Infact, go read that and then come back to this post. It'd make more sense.


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

