---
title: Morph Targets
image:
description: Learn all about morph targets in Babylon.js.
keywords: diving deeper, meshes, morph targets, blend shapes
further-reading:
video-overview:
video-content:
---

## Morph targets

Morph targets are a new feature introduced with Babylon.js v3.0.

![Morph Target Before and After](/img/how_to/morphtargets.jpg)

## Basics

Meshes can be deformed by using morph targets. A morph target must be built from a mesh with the **EXACT** same number of vertices as the original mesh.
Morph targets are used by the GPU to create the final geometry by applying the following formula:

final mesh = original mesh + sum((morph targets - original mesh) \* morph targets influences)

For instance, you can use morph targets to simulate the opening of a mouth. The initial mesh has a closed mouth. The morph target can be the same mesh but with an opened mouth. Then by changing the influence of the morph target (from 0 to 1) you can display either a closed or an opened mouth or a mix of both.

You can find live examples here:
<Playground id="#HPV2TZ#8" title="Animated Morph Targets" description="Simple example of animated morph targets."/>

The following two examples are best seen in the full Playground where sliders can be used to change the influencers
<Playground id="#HPV2TZ#2" title="Animated Morph Targets with Standard Material" description="Simple example of animated morph targets with standard material."/>  
<Playground id="#HPV2TZ#4" title="Animated Morph Targets with PBR Material" description="Simple example of animated morph targets with PBR material."/>

## How to Use Morph Targets

To use morph targets, you first have to create a `MorphTargetManager` and affect it to a mesh:

```javascript
const manager = new BABYLON.MorphTargetManager();
sphere.morphTargetManager = manager;
```

Then you can create `MorphTarget` either with the `FromMesh` static function:

```javascript
const target = BABYLON.MorphTarget.FromMesh(sphereTarget, "target", 0.25);
```

or simply by creating a target and specifying positions and normals:

```javascript
const target = new BABYLON.MorphTarget(name, influence);
target.setPositions(...);
target.setNormals(...);
```

Once done, you can specify the influence of a specific target with `target.influence = 0.25`

Targets with influence = 0 are disabled.

Here is a complete example with 4 targets:

```javascript
const scramble = function (data) {
  for (index = 0; index < data.length; index++) {
    data[index] += 0.1 * Math.random();
  }
};

// Main sphere
const sphere = BABYLON.MeshBuilder.CreateSphere("sphere1", { segments: 16, diameter: 2 }, scene);

// Let's create some targets
const sphere2 = BABYLON.MeshBuilder.CreateSphere("sphere2", { segments: 16, diameter: 2 }, scene);
sphere2.setEnabled(false);
sphere2.updateMeshPositions(scramble);

const sphere3 = BABYLON.MeshBuilder.CreateSphere("sphere3", { segments: 16, diameter: 2 }, scene);
sphere3.setEnabled(false);

sphere3.scaling = new BABYLON.Vector3(2.1, 3.5, 1.0);
sphere3.bakeCurrentTransformIntoVertices();

const sphere4 = BABYLON.MeshBuilder.CreateSphere("sphere4", { segments: 16, diameter: 2 }, scene);
sphere4.setEnabled(false);
sphere4.updateMeshPositions(scramble);

const sphere5 = BABYLON.MeshBuilder.CreateSphere("sphere5", { segments: 16, diameter: 2 }, scene);
sphere5.setEnabled(false);

sphere5.scaling = new BABYLON.Vector3(1.0, 0.1, 1.0);
sphere5.bakeCurrentTransformIntoVertices();

// Create a manager and affect it to the sphere
const manager = new BABYLON.MorphTargetManager();
sphere.morphTargetManager = manager;

// Add the targets
const target0 = BABYLON.MorphTarget.FromMesh(sphere2, "sphere2", 0.25);
manager.addTarget(target0);

const target1 = BABYLON.MorphTarget.FromMesh(sphere3, "sphere3", 0.25);
manager.addTarget(target1);

const target2 = BABYLON.MorphTarget.FromMesh(sphere4, "sphere4", 0.25);
manager.addTarget(target2);

const target3 = BABYLON.MorphTarget.FromMesh(sphere5, "sphere5", 0.25);
manager.addTarget(target3);
```

At any time, you can remove a target with `manager.removeTarget(target)`

## How to Access Morph Targets in a glTF File

You can access a morph target influence on a mesh in a glTF file through the [morphTargetManager](/typedoc/classes/babylon.morphtargetmanager#gettarget) which is automatically created for a loaded glTF file containing morph targets. You can see how many influences are present on the mesh by writing to the console.

```javascript
console.log(mesh.morphTargetManager);
```

If you want to view or change the value of a morph target influence, it can be accessed by getting the influence from the array in the morphTargetManager by key value.

```javascript
myInfluence = mesh.morphTargetManager.getTarget(key);
```

See the following example for a full playground using morph targets from a glTF file.

- <Playground id="#9CLJEF" title="Morph Targets From a .glTF File" description="Simple example of using morph targets from a .glTF file."/>

## List of Morphable Properties

You can morph the following mesh attributes:

- position
- normal (can be turned of by calling `manager.enableNormalMorphing = false`)
- tangents (can be turned of by calling `manager.enableTangentMorphing = false`)
- uvs (can be turned of by calling `manager.enableUVMorphing = false`)

## Animating Morph Targets

You can animate any morph target influence by creating a [BABYLON.Animation](https://doc.babylonjs.com/features/featuresDeepDive/animation/animation_method) targeting the influence you wish to control. When creating your animation, you need to list "influence" as the property to be animated. This property will always be of type FLOAT.


```javascript
const myAnim = new BABYLON.Animation(name, "influence", frames_per_second, BABYLON.Animation.ANIMATIONTYPE_FLOAT, loop_mode);
```

To play the morph target animation, add the animation to the mesh that owns the [MorphTargetManager](https://doc.babylonjs.com/typedoc/classes/BABYLON.MorphTargetManager) so that the animation is easy to access later. You can then call `scene.beginAnimation` while targeting the specific morph target you want to animate.

```javascript
mesh.animations.push(myAnim);
scene.beginAnimation(morphTarget, from, to, true);
```

You can animate multiple morphs on the same mesh by creating a new animation for each morph and targeting the appropriate influence. If you want to play several morph animations simultaneously - such as playing blink animations on morph targets driving each individual eye - you can add these animations to an [animation group](https://doc.babylonjs.com/features/featuresDeepDive/animation/groupAnimations).

- <Playground id="#019CR7#2" title="Animating Morph Targets" description="Simple example of adding animation to a morph target"/>

## Use Morph Targets with Node Material

The [Node Material](/features/featuresDeepDive/materials/node_material) is a powerful tool that allows creating shaders without having to write GLSL. To use a Node Material in a mesh with Morph Targets, you need to add the Morph Target node to it so that vertex positions, normals and uvs properly respond to the effects of the morph:

![Use Morph Target with Node Material](/img/how_to/morphtargetnode.png)

<Playground id="#HPV2TZ#299" title="Using Node Material with Morph Targets" description="Use Morph Target node on the Node Material"/>

## Limitations

On WebGL2+ Babylon.js will use textures to store the targets so the only limit is the GPU memory. You can find an example of pretty large numbers of targets here: - <Playground id="#1PD3Q7#2" title="Lots of Morph Targets" description="Example showing a large number of morph targets."/>

On WebGL1, the system cannot store inside textures (it requires float textures) so as a fallback it will store the targets inside mesh vertices.

- Please be aware that most of the browsers are limited to 16 attributes per mesh. Adding a single morph target to a mesh add up to 4 new attributes (position + normal + tangents + uvs). This could quickly go beyond the max attributes limitation.
- All targets within a same manager must have the same vertices count
- A mesh and its MorphTargetManager must have the same vertices count

You can find a video explaining how morph targets work here:

<Youtube id="LBPRmGgU0PE"/>
