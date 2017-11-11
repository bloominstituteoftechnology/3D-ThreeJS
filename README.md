# 3D with three.js

https://threejs.org/

https://threejs.org/examples/

## Audience

JavaScript developers interested in 3D.

## Demo quick links

* [Scene graph](examples/scenegraph/index.html)
* [Lighting and shadows](examples/lighting/index.html).

These are elaborated on further below.


## three.js vs. WebGL vs. an Engine

WebGL is a low-level interface for doing 3D. It's concerned with the
most basic types of operations that can be done. For example, you can
use it to draw a triangle. It will not do something as complex as
loading a 3D model of car.

A full-blown game engine will include all kinds of library functions to
ease 3D development. It also will include *tooling* for helping to build
assets for the game. If you are writing a complete project, choose a
good 3D engine. Web-based engines will use WebGL deep down for rendering.

three.js is a compromise. It takes away a lot of the boilerplate that's
necessary with WebGL, and includes a lot of helpful functions for
importing models, generating geometry, and so on. As such, it's a
lightweight engine that's great for small demos and learning. (Or for
bulding a full-blown engine on top of!)

[JS Game Engine comparison](https://html5gameengine.com/)


## 3D in General

Though it seems mysterious, it's actually conceptually straightforward.

Take out your phone, start up the camera app, and pan the phone around
the room. Notice how the phone is taking a 3D space and turing it into a
2D image. We want to do the same thing except with a mathematical,
virtual world.

Fortuntely, it's possible to go pretty far without knowing 3D math, but
it will become a requirement eventually.

For now, think of everything as existing in 3-space, with 3D `<x,y,z>`
coordinates being used to represent positions of objects.

What are the coordinate units that are used in the 3D world? Feet?
Meters? It doesn't actually matter. A 300-meter tall building looks just
as tall to a 2-meter tall person as a 300 millimeter tall building looks
to a 2-millimeter tall person.

But you should choose a unit and stick with it. Lots of places choose
meters.


## 3D Math

Usually there are libaries to perform the operations you need to do. But
sometimes, you need to do some 3D computation that involves manipulating
positions and directions in 3-space, and that's where 3D math comes in
handy.

There are two basic data types you need to know: *vector* and *matrix*.

Vectors typically hold positions or direction, and for 3D are a
collection of 3 coordinates: `<x, y, z>`.

Vectors that are length `1` are called *unit vectors*.

Vectors can be made into length 1 by *normalizing* them.

Matrixes typically hold rotation, position, and scaling information or
camera projection information. These are typically 4×4 matrixes:

```
[
  [a, b, c, d],
  [e, f, g, h],
  [i, j, k, l],
  [m, n, o, p]
]
```

The *model matrix* holds rotation/position information that can be
used to *transform* 3-space points of a model to other points.

Importantly, this is how you cause movable things things to be
positioned and rotated in the world.

The *view matrix* holds rotation/position information for the virtual
camera, where it is and where it's facing.

The *projection matrix* holds projection information about the virtual
camera, e.g. field of view, zoom level.

Two common projections are *perspective* (regular normal 3D) or
*orthogonal*, which flattens out the perspective (often used for CAD or
user interfaces).

To *transform* a vector by a matrix, multiply the matrix by the vector.
A transformed vector will result.

More 3D math is given in the [3D Math Details](#3d-math-details)
section.


## 3D Objects in the World

Objects are defined by geometry, materials, position, rotation, and
other attributes.

### Geometry

The *geometry* is the overall shape of the object, e.g. box, cylinder,
goat.

### Matrial

The *material* describes the visual qualities of the object, e.g. what
color it is, what texture it is, how it reflects light.

### Mesh

The geometry plus the material makes a *mesh* in three.js. You add
meshes to the world held in the *scene graph*.


## Scene Graph

Objects in the world are held in a tree structure that signifies the
parent/child relationship between objects in the world.

For example, the root node (that holds the whole world) might have 3
children that are "cars". Each "car" might have 4 "wheel" children.

The advantage of this kind of hierarchy is that when you move a car (by
changing its model matrix), all its wheels automatically move with it.

### three.js: Scene Graph

```javascript
// Create the scene graph (empty)
const scene = new THREE.Scene();

// Make cars
function makeCar() {
  const carObj = makeCarMesh(); // Exercise for the reader
  const wheelObj = [];
  
  for (let i = 0; i < 4; i++) {
	  let w = makeWheelMesh(); // Also exercise for the reader
	  wheelObj.push(w);

      // Add wheels as children to car on the scenegraph
      carObj.add(w);
  }

  // Position wheels RELATIVE to the car
  wheelObj[0].position.set(-1, 0, -1);
  wheelObj[1].position.set( 1, 0, -1);
  wheelObj[2].position.set(-1, 0,  1);
  wheelObj[3].position.set( 1, 0,  1);

  return carObj;
}

// Add a bunch of cars to the world
let cars = [];

for (let i = 0; i < 10; i++) {
	let c = makeCar();

	cars.push(c);

	scene.add(c);
}

// Position cars
// (Wheels automatically move with the cars because they're children
// of the cars on the scene graph)
cars[0].position.set(20, 30, 40);
```

[Running example with two connected crates](examples/scenegraph/index.html)


## Camera and viewport

The *viewport* is the 2D portion of the screen that will show the 3D scene.

```javascript
// Make a viewport and renderer, and attach them to the DOM

let viewportWidth = window.innerWidth; // Browser window size
let viewportHeight = window.innerHeight;

const renderer = new THREE.WebGLRenderer(); // Create new WebGL-based renderer
renderer.setSize(viewportWidth, viewportHeight); // Set render size
document.body.appendChild(renderer.domElement); // Add to DOM
```

The *camera* describes how the scene will appear. Cameras come in two
main varietys, *perspective* (typical 3D) and *orthogonal* (flattened). 

For a perspective camera, we need to specify field of view, viewport
aspect ratio, and the near and far *clipping planes*.

Clipping planes cause the renderer to not display anything closer to the
camera than the *near clipping plane*, and nothing farther from the
camera than the *far clipping plane*.

```javascript
// Make a perspective camera

const fov = 45; // Field of view, degrees
const aspect = viewportWidth / viewportHeight; // Screen aspect ratio
const nearClip = 1;
const farClip = 500;

// Create the camera
const camera = new THREE.PerspectiveCamera(fov, asp[ect, nearClip, farClip);

// Set the camera's X, Y, Z position
camera.position.set(0, 0, 80);

// Look at the origin, <0,0,0>
camera.lookAt(new THREE.Vector3(0, 0, 0));

// ... farther down the code ...

// Render the scene with this camera
renderer.render(scene, camera);
```

## Building a Cube

In order to build a 3D object in three.js, you need a mesh. In order to
get a mesh, you need a geometry and a material.

```javascript
// Making a yellow cube

const cubeGeometry = new THREE.BoxGeometry(1, 1, 1);
const cubeMaterial = new THREE.MeshBasicMaterial({ color: 0xffff00 });

const cube = new THREE.Mesh(cubeGeometry, cubeMaterial);

// Add to the scene
scene.add(cube);
```

## Building a Textured Cube

It's just like a flat-shaded cube, but it uses a texture material
instead of the flat color material.

```javascript
// Making a textured cube

const crateTexture = new THREE.TextureLoader().load('img/crate.jpg');

const cubeGeometry = new THREE.BoxGeometry(1, 1, 1);
const cubeMaterial = new THREE.MeshBasicMaterial({ map: crateTexture });

const cube = new THREE.Mesh(cubeGeometry, cubeMaterial);

// Add to the scene
scene.add(cube);
```

## Position and Rotation

Objects in 3-space can be moved by changing their position. They can be
rotated by changing their rotation. (Rotation values are specified in
[radians](https://en.wikipedia.org/wiki/Radian).)

```javascript
// Set the cube position, X, Y, Z:

cube.position.set(10, 20, 30);

// Or set specific coordinates
cube.position.y = 30;


// Set the cube rotation around X, Y, and Z axes:

cube.rotation.set(0, Math.PI * 0.25, 0); 

// Or set rotation around a specific axis
cube.rotation.x = Math.PI/3; // rotation in radians
```


## Animation in three.js

Use
[requestAnimationFrame()](https://developer.mozilla.org/en-US/docs/Web/API/window/requestAnimationFrame)
to schedule animation updates.

```javascript
function animate() {
  requestAnimationFrame(animate);

  cube.rotation.y += 0.05;

  renderer.render(scene, camera);
}

animate();
```

## Drawing lines

Lines work like meshes, except they're lines, not meshes. But they're
still made up of geometry and material.

```javascript
const lineGeometry = new THREE.Geometry();
lineGeometry.vertices.push(new THREE.Vector3(-10, 0, 0));
lineGeometry.vertices.push(new THREE.Vector3(0, 10, 0));
lineGeometry.vertices.push(new THREE.Vector3(10, 0, 0));

const lineMaterial = new THREE.LineBasicMaterial({ color: 0x0000ff });

const line = new THREE.Line(lineGeometry, lineMaterial);

scene.add(line);
```


## Lighting and Shadows

### Light Types

*Point Lights*: These are infinitely small and cast light in all
directions, like light bulbs.

*Directional Light*: Casts light in a specific direction, but is
infinitely far away, like the Sun.

*Spot Lights*: Cast a cone of light from a specific point.

(There are more, but these are common.)

You can have multiple lights of different types in a scene.

So far we're been using `MeshBasicMaterial()` which doesn't help us with shadows because it makes the objects a specific color regardless of the lights. Instead we'll use `MeshPhongMaterial()`.

> [Phong shading](https://en.wikipedia.org/wiki/Phong_shading) is a way
> of adding shadows and highlights to surfaces. Think of it as dealing
> with how shiny or reflective a surface is.

The steps are:

1. When you make a renderer, you have to tell it to use shadow maps:

    ```javascript
    // New renderer
    const renderer = new THREE.WebGLRenderer();
    renderer.shadowMap.enabled = true; // ENABLE SHADOWS 
	```

2. Add a light (or multiple lights) to the scene, and tell it to cast shadows:

    ```javascript
    // Add a light source
    const light = new THREE.PointLight(0xffffffff, 1, 0, 1);
    light.position.set(40, 60, 20);
    light.castShadow = true; // CAST SHADOWS
    scene.add(light);
	```

	Note that only some light types cast shadows.

3. When you create meshes that need to be shaded by lights, use `MeshPhongMaterial()`:

    ```javascript
    const cubeMaterial = new THREE.MeshPhongMaterial({
        color: 0xaaaaaa,    // Intrinsic color of mesh
        shininess: 100,     // How shiny
        specular: 0x111111  // What color the shiny highlight is
    });

	```
    Instead of `color` in the material settings, you can also use `map` if you have a texture map.

4. Mark the objects that need to cast shadows:

    ```javascript
    // Make a crate to rotate
    const crate = makeCrate(20);
    crate.castShadow = true; // MAKE THE CRATE CAST SHADOWS
	```

5. Mark the objects that will receive shadows, i.e. have shadows cast upon them:

    ```javascript
    const plane = makeCheckerboard();
    plane.receiveShadow = true; // MAKE THE PLANE RECEIVE SHADOWS
	````

6. (Optional) Add an ambient light to the scene so that things in shadow
   aren't pitch black:

    ```javascript
    // Add an ambient light to the scene
    const ambientLight = new THREE.AmbientLight(0x222222); // soft white light
	scene.add(ambientLight);
	```

[Running example of lighting and shadows](examples/lighting/index.html).


## Shaders

TODO
https://aerotwist.com/tutorials/an-introduction-to-shaders-part-1/


## Bump Maps

TODO


## 3D Math Details

Again, most math can be done with libraries. But occasionally you'll need something to reflect in a weird way off a surface and you'll have to roll your own.


### Vectors

This is a set of `<x,y,z>` corrdinates that represent a point in space.
Typically X is horizontal across your screen, Y is vertical, and Z is
"into" the screen.

For mathematical convenience, we often use 4D vectors `<x,y,z,w>` for
the actual math, but you don't need to start with those details.

Sometimes a vector is used to represent a *position* in 3-space. Think
of a thing floating in space at that position.

Sometimes a vector is used to represent a *direction* in 3-space. Think
of an arrow pointing from the origin `<0,0,0>` to the position in the
vector.

We'll use these in the following examples:

```javascript
// Our vectors
v1 = { x: 10, y: 20, z: 30 };
v2 = { x: 40, y: 50, z: 60 };
```

#### Vector Length (or *magnitude*)

To compute the length of a 3D vector, use the [distance
formula](https://en.wikipedia.org/wiki/Distance#Geometry).

Sometimes all you need is the square of the distance, which saves you a
square root operation, as seen below.

```javascript
// Compute the difference for each component
dx = v2.x - v1.x;
dy = v2.y - v1.y;
dz = v2.z - v1.z;

// Compute the distance
dist = Math.sqrt(dx*dx + dy*dy + dz*dz);

// Compute the square of the distance, if that's all you need
distSquared = dx*dx + dy*dy + dz*dz;
```

Magnitude of a vector *v*  is often written `|v|` or `‖v‖`.

#### Vector Uniform Scale

To scale a vector uniformly, multiply each component by a constant value:

```javascript
// Scale v1 by 10, store in v2:
v3.x = v1.x * 10;
v3.y = v1.y * 10;
v3.z = v1.z * 10;
```

#### Vector Normalize

Normalizing makes the length of a vector `1`. A vector of length 1 is
called a *unit vector*.

It is common to normalize a vector when you're simply interested in its
direction, and you want to use that direction in subsequent
calculations.

To normalize a vector, scale it uniformly by the inverse of it's length.
That is, divide every component by the length of the vector.

```javascript
// Normalize v1

length = magnitude(v1);

v1.x /= length;
v1.y /= length;
v1.z /= length;
```

Clearly you cannot normalize a vector of length `0`.


#### Vector Dot Product

This gives you an indication on how parallel or how perpendicular two vectors are.

It also helps you if you need to *project* a vector onto another vector.

```javascript
// Dot Product Formula 1

dotProduct = (v1.x * v2.x) + (v1.y * v2.y) + (v1.z * v2.z);
```

If the dot product is zero, it means the vectors are perpendicular.

If the dot product is negative, it means the vectors are generally
pointed opposite directions.

If the dot product is positve, it means the vectors are generally
pointed the same direction.

How do you get the exact angle?

The end result of a vector dot product is the length of one, times the
length of the other, times the
[cosine](https://en.wikipedia.org/wiki/Trigonometric_functions#cosine)
of the angle between them.

```
// Dot Product Formula 2

dotProduct = |v1| × |v2| × cos(θ)
```

To get the angle, we'll use Dot Product Formula 1, above, and then we'll
divide the vector magnitudes out so we get just the cosine part. Then
we'll take the arccosine to get the angle.

```javascript
// Remember this is equivalent to |v1||v2|cos(θ):
dotProduct = (v1.x * v2.x) + (v1.y * v2.y) + (v1.z * v2.z);

// Divide the magnitude (length) of v1 out of the result:
dotProduct /= magnitude(v1);

// Now dotProduct is equivalent to |v2|cos(θ)

// Divide the magnitude of v2 out of the result:
dotProduct /= magnitude(v2);

// Now dotProduct is equivalent to cos(θ)

// Use arccos to get the angle
angle = Math.acos(dotProduct)
```

#### Vector Cross Product

The cross product will produce another vector that is perpendicular to the two vectors given.

If the input vectors are parallel, the cross product will be zero length.

TODO


### Matrixes
TODO

#### Orientation matrix
TODO

#### Inverse/Transpose
TODO

#### Projection matrix
TODO


## References

TODO


## Assignment

### Procedural 3D Landscape

TODO

### Simple FPS

TODO

## TODO
order of matrix ops
