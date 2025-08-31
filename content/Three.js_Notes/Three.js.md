[their website](https://threejs.org/)

## VSCode tips

`SHIFT + ALT + F` --- format code

`npm run dev` to launch on localhost
`nvm use node` if it throws error to force last node version


## Typescript tips


```ts
const meshes = [
  new THREE.Mesh(geometry, new THREE.MeshBasicMaterial({ color: data.color })),
  new THREE.Mesh(geometry, /*...*/),
  new THREE.Mesh(geometry, /*...*/),
  new THREE.Mesh(geometry, /*...*/),
]

scene.add(...meshes) // <--------- SHORTCUT!!

```


## Resources

- [PolyHaven](https://polyhaven.com/) - a public 3D asset library
	- HDR images
	- textures
	- models

---


# Projects init 

[[Vite]]

[github page for boilerplate](https://github.com/Sean-Bradley/Three.js-Boilerplate-TS-Vite)

`npm run dev`
can update files without shutting down then restart



**Adding Three**:

`npm install three --save-dev`
`npm install @types/three --save-dev`
	this is to make it work with TypeScript (this package contains `ts` type definitions)

The types package might be behind a version, *if* there are issues just downgrade the other package

---

## Three modules

`node_modules/three/examples/jsm/...`

to import:
```
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls.js'
```
The `.js` is a necessity of Vite, other *bundlers* don't need it


#### Aliases

In recent versions of Three, you can use aliases to shorten import paths; in `node_modules/three/package.json` :
```
"exports": {
	...
	"./addons/*": "./examples/jsm/*"
	...
}
```

which means that now I can import with shortened path:
```
import {OrbitControls} from 'three/addons/controls/OrbitControls.js'
```

These aliases do not work with every *bundler*, but Vite supports it


### `default`

When importing from libraries, if in library code ends with `export default libname` then just `import libname from ...`

Otherwise, if it ends with `export { libname }` then `import { libname } from ...`


---

## Third party libraries


`npm install <libname>@latest --save-dev` will add the third party library ref to `devDependencies` in `package.json`

`npm install @types/<libname>@latest --save-dev` for adding ts type definitions


### Third party GUIs

Three.js official examples currently use [lil-gui](https://github.com/georgealways/lil-gui), but other popular options are [dat.gui](https://github.com/dataarts/dat.gui) (minimal), [Tweakpane](https://tweakpane.github.io/docs/) and [SBEDIT](https://sbedit.net/1e64b476676f6f945af94f278a392aba642d135c).


---
---
---






# Components


## Stats

The stats panel shows the fps and the ms between each frame; on browsers other than firefox there's a third mb info box in red.

`stats.update()` at the end of the `animate()` function shows stats for the whole animation loop; one can also measure stats about a block of code encasing it with `stats.begin()` and `stats.end()`, removing `stats.update()`.


```ts
function animate() {

	requestAnimationFrame(animate)
	renderer.render(scene, camera)
	
	stats.update() // at every frame
}
```


---
---


## Rendering


### Object3D

Parent class that many others derive from; provides a set of properties and methods for manipulating objects in 3D space.

`objectA.add(objectB)` to add object B as a child to object A.

> [!docs] [Object3D docs](https://threejs.org/docs/#api/en/core/Object3D)

---

### Scene

Extends Object3D.

A **Scene** allows you to set up, in 3D coordinates, what is to be rendered by Three.js.

```ts
const scene = new THREE.Scene()

const cube = new THREE.Mesh(geometry, material)
scene.add(cube)
```

> [!docs] [Scene docs](https://threejs.org/docs/#api/en/scenes/Scene)


**One can have multiple scenes.** 

An object cannot be a child of multiple scenes: further assignments overwrite previous ones.

Multiple scenes are in memory, however they can only be rendered one at a time. For example, in the GUI I can add buttons to switch which scene gets rendered.

> [!tip]
> `console.log(scene)` will print out all the scene properties and the objects contained.

##### Properties: 

###### Background

Solid color:
````ts
scene.background = new THREE.Color(0x123456)
````

Image:
```ts
scene.background = new THREE.TextureLoader().load(`img path or url`)
```

Skybox:
```ts
scene.background = new THREE.CubeTextureLoader()
	.setPath('https://sbcode.net/img/')
	.load(['px.png', 'nx.png', 'py.png', 'ny.png', 'pz.png', 'nz.png'])
```
Where `px` = positive x axis, `nx` = negative x axis etc; the images that make up the cube


---

### Camera

There are multiple camera types, notably the *Pespective* and *Orthographic* ones:
- [**`PerspectiveCamera`**](https://threejs.org/docs/index.html#api/en/cameras/PerspectiveCamera): mimics the way the human eye sees. It is the most common projection mode used for rendering a 3D scene;
- [**`OrthographicCamera`**](https://threejs.org/docs/index.html#api/en/cameras/OrthographicCamera): in this projection mode, an object's size in the rendered image stays constant regardless of its distance from the camera. ==This can be useful for rendering 2D scenes and UI elements, amongst other things.==


> [!docs] [Camera docs](https://threejs.org/docs/#api/en/cameras/Camera)


##### Perspective Camera

```ts
const camera = new THREE.PerspectiveCamera( 45, width / height, 1, 1000 );
scene.add( camera );
```

The constructor requires the following arguments to define the camera's properties:
- the field of view (in this case, camera depth along its "tracks")
- the aspect ratio
- the near plane
- the far plane
![Viewing Frustum|300](>publish/quartz/content/Three.js_Notes/images/frustum.png)
This information defines the [viewing frustum](https://en.wikipedia.org/wiki/Viewing_frustum), or the modeled region of space, that is necessary for rendering.



##### Orthographic Camera

![Perspective vs Orthographic view](>publish/quartz/content/Three.js_Notes/images/perspectiveVSortho.png)

An orthographic camera has the following properties:
- left plane
- right plane
- top plane
- bottom plane
- near plane
- far plane
All as numbers relative to camera position: `camera.position.set(x, y, z)`.




#### Automatic resizing

For perspective cameras:

```ts
const camera = new THREE.PerspectiveCamera(75, 
	window.innerWidth / window.innerHeight, // <--- aspect ratio tied to window
	0.1, 1000)

const renderer = new THREE.WebGLRenderer()
renderer.setSize(window.innerWidth, window.innerHeight)  // ... same for render
document.body.appendChild(renderer.domElement)

// here the properties affected by resizing get updated
window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight
  camera.updateProjectionMatrix() // REQUIRED EVERY TIME A PROPERTY IS UPDATED
  renderer.setSize(window.innerWidth, window.innerHeight)
})
```


For orthographic cameras it is harder to do; the resize function will be more complex.

---

### Renderer

The most commonly used is the `WebGLRenderer`; it is quite fast and smooth (does not use js). It uses the HTML element `<canvas>`.

> [!docs] [WebGLRenderer docs](https://threejs.org/docs/#api/en/renderers/WebGLRenderer)

```ts
const renderer = new THREE.WebGLRenderer()
renderer.setSize(window.innerWidth, window.innerHeight) // browser fullscreen
document.body.appendChild(renderer.domElement)
```

> Usually, camera and renderer should have the same aspect ratio.
> Same goes for the 'resize' listener, update them both.

When creating a `WebGLRenderer` without arguments, it automatically creates the `<canvas>` HTML element that is used for rendering. This must then be appended to the `<body>`.


Otherwise, one can hard code a `<canvas>` element into the `index.html` of the project:
```html
<canvas id="canvas"></canvas>
```
```ts
const canvas = document.getElementById('canvas') as HTMLCanvasElement
const renderer = new THREE.WebGLRenderer({ canvas: canvas })

```


=============================


```ts
function animate() {
  requestAnimationFrame(animate)

  renderer.render(scene, camera)
}
```

`renderer.render(scene, camera)` = Render a scene or an object using a camera.


============================

> [!tip] 
> Antialiasing is off by default. To set to true:
> ```ts
> const renderer = new THREE.WebGLRenderer({ antialias: true })
> ```



---
---


## Animation loop

in `main.ts`, there's an animation loop:

```ts
function animate() {

	requestAnimationFrame(animate)
	
	// insert changes here
	
	renderer.render(scene, camera)
}

animate() // call once to kickstart loop
```


`requestAnimationFrame` queues the `animate()` function to be racalled as fast as the browser allows to; for example, when running on second monitor the stats panel shows 60fps, while on the new monitor it goes up to 180fps.

> `requestAnimationFrame(animate)` is a browser function, not a Three.js function.


##### Refresh-rate independent animation

It is a good idea to not tie an animation to the user's screen's refresh rate, in order to deliver the same experience to all end users; for example, the following code:

```ts
function animate() {
  requestAnimationFrame(animate)

  cube.rotation.x += 0.01
  cube.rotation.y += 0.01

  renderer.render(scene, camera)
}
```
Makes the cube rotate slower for lower refresh rate, and faster for higher ones.

To make the cubo rotate at the same speed for all FPS:

```ts
const clock = new THREE.Clock()
let delta

function animate() {
  requestAnimationFrame(animate)

  delta = clock.getDelta() // time passed since last loop = last delta call

  cube.rotation.x += 0.01 * delta  // value scaled to refresh rate
  cube.rotation.y += 0.01 * delta

  renderer.render(scene, camera)
}
```


### On-demand rendering

To save up on resources, do not include animations that trigger upon interaction in the animation loop: simply add `renderer.render(scene, camera)` to event listeners.

For example, `OrbitControls` allows to rotate the cube by click-and-drag. It does work by including rendering inside the animation loop, but a more efficient way is trigger the re-rendering only upon interaction:

```ts
const orbit = new OrbitControls(camera, renderer.domElement)
orbit.addEventListener('change', () => {
  renderer.render(scene, camera)
})
```

Also, if removing rendering from the animation loop, must include it in the window resize listener.


---
---


## Object3D


> [!docs] [Object3D docs](https://threejs.org/docs/?q=object3d#api/en/core/Object3D)

Base class for most Three.js objects; the main properties are:
- position
- rotation
- scale
- visible

`object3D.position = N`


Add an object3D as a child of another object3D with the `parent.add(child)` method:
- an object can have at most one parent (last one overwrites previous);
- any changes  to the parent's properties (scale, position etc) will affect the children;
	- ==children's properties are relative to parent's properties==: for example, object A has posx = 4 and the child B has "local position" = 6; child B's "world position" will be 10.
- it is possible to nest objects.


```ts
object1.position.set(4, 0, 0)
object2.position.set(6, 0, 0)
object1.add(object2)

const object2WorldPosition = new THREE.Vector3()
object2.getWorldPosition(object2WorldPosition) 
// object2WorldPosition = (10, 0, 0)
```


---
---
---



# Materials


> [!docs] [Materials docs](https://threejs.org/manual/#en/materials)

Materials define how objects will appear in the scene.

Material objects have properties that can be changed after creation.
This requires to update the rendering.
```ts
material.needsUpdate = true
```

> [!info] From the docs
> "This topic rarely affects most three.js apps but just as an FYI... Three.js applies material settings when a material is used where "used" means "something is rendered that uses the material". Some material settings are only applied once as changing them requires lots of work by three.js. In those cases you need to set `material.needsUpdate = true` to tell three.js to apply your material changes."

> [!tip] Update material only after trying without.


The `side` property allows to select whether to render the *front* side, meaning the external-facing planes of the object, the *back* side, meaning the internal planes, or both. The default is to render the front planes, as that's less resource intensive.
- There are certain objects, like planes, that turn invisible if the camera is placed on the back side of the object.
- The back side rendering of a cube could be useful for not rendering the camera-facing wall of a room for example, in order to view the interiors.


The most commonly used materials are:
- [MeshBasicMaterial](https://threejs.org/docs/#api/en/materials/MeshBasicMaterial) - does not react to light (! easily blends into background )
- [MeshNormalMaterial](https://threejs.org/docs/#api/en/materials/MeshNormalMaterial) - shading relative to camera
- [MeshPhongMaterial](https://threejs.org/docs/#api/en/materials/MeshPhongMaterial) - absolute shading, easier on performance
- [MeshStandardMaterial](https://threejs.org/docs/#api/en/materials/MeshStandardMaterial) - absolute shading, more realistic
	- uses *physically based rendering* (PBR), a recent standard in 3D applications

*Phong* and *Standard* need a light in the scene, alternatively *Standard* can be lit with and *environment map* (most quality solution).


## Lights and shadows



> [!docs] 
> [Lights](https://threejs.org/manual/#en/lights) and [Shadows](https://threejs.org/manual/?q=shadow#en/shadows) docs


The main light that can be added to a scene:
- [AmbientLight](https://threejs.org/docs/?q=ambient#api/en/lights/AmbientLight) illuminates the whole scene in all directions.
- [DirectionalLight](https://threejs.org/docs/#api/en/lights/DirectionalLight) illuminates the whole scene in 1 direction.
- [PointLight](https://threejs.org/docs/#api/en/lights/PointLight) illuminates in all directions from a 3D position. Distance and decay can be managed.
- [SpotLight](https://threejs.org/docs/#api/en/lights/SpotLight) illuminates in 1 direction from a 3D position. Distance, decay, angle, penumbra and target can be managed.

Shadows cast by objects need to be added separately.
Lights manage their own camera frustums when they are set with:
```ts
light.castShadow = true
```
One can set the desired camera frustum for lights, like for cameras. Use the helpers to visualize the bounding boxes.
> [!tip] 
> Remember to update the camera for lights as well in order to render a change in light properties, just like for normal cameras.
> ```ts
> light.shadow.camera.updateProjectionMatrix()
> lightHelper.update()
> ```


From the docs, "Three.js by default uses _shadow maps_. The way a shadow map works is, _for every light that casts shadows all objects marked to cast shadows are rendered from the point of view of the light_."

The `shadowMap` render property needs to be enabled:
```ts
renderer.shadowMap.enabled = true
```

Then scene objects can then be set to cast or receive shadows:
```ts
mesh.castShadow = true  // materials can cast or receive shadows
ground.receiveShadow = true
```


> [!tip] 
> The [bias](https://threejs.org/docs/?q=shadow#api/en/lights/shadows/LightShadow.bias) light property is used to tweak surface depth when doing shadow calculations; The default is `0`. Very tiny adjustments here (in the order of `0.0001`) may help reduce artifacts in shadows.



## Environment Maps

Usually better results than general lights, best results with PBR materials (`MeshStandardMaterial` and `MeshPhysicalMaterial`).

The environment map is set witht the property `scene.environment`.

And environment map takes longer for the initial rendering, but is more realistic and actually renders faster on updates.



Simple environment maps can be created from the skybox seen in [[#Background]]:
```ts
const environmentTexture = new THREE.CubeTextureLoader().setPath(/*img path*/).load(['px.png', 'nx.png', 'py.png', 'ny.png', 'pz.png', 'nz.png'])
scene.environment = environmentTexture
scene.background = environmentTexture
```
The shades and colors on the objects are calculated from the background.



More detailed environment maps are created from HDR images, which are much larger. As such, a loader is used to trigger operations after the image has been "converted" into the ready to use texture:
```ts
const hdr = 'https://sbcode.net/img/rustig_koppie_puresky_1k.hdr'

let environmentTexture: THREE.DataTexture // ts

new RGBELoader().load(hdr, (texture) => {
   environmentTexture = texture
   environmentTexture.mapping = THREE.EquirectangularReflectionMapping
   scene.environment = environmentTexture
   scene.background = environmentTexture
   scene.environmentIntensity = 1
})

```

Also, a *tone mapping* slider can be used to adjust the intensity of reflection colors and light:
```ts
renderer.toneMappingExposure = // a, b, c
scene.environmentIntensity = // a, b, c
```
From [MeshStandardMaterial](https://threejs.org/docs/?q=mesh#api/en/materials/MeshStandardMaterial.envMapIntensity).


---
---
---

# Loading resources


> [!docs] [Loading 3D Models - docs](https://threejs.org/manual/?q=load#en/loading-3d-models)

There are many loaders to choose from; the differences are mainly loading speed, number of supported features, standardization.

Three.js suggests using [GLTFLoader](https://threejs.org/docs/?q=loader#examples/en/loaders/GLTFLoader).



All loaders extend a base `Loader` class that contains a [`LoadingManager`](https://threejs.org/docs/#api/en/loaders/managers/LoadingManager). This can be used to track resource loading progress, as well as override resource URLs during loading.



`loader.load(...)` methods are asynchronous, thus in case of loading multiple assets beware of that. It is possible to nest loading assets inside another loading callback.
	`loader.loadAsync()` to work with promises, as well as use `await` to block all code execution waiting for resource loading.
	Otherwise, use ==`async function`== to execute loading code.



---
---
---

# OrbitControls



> [!docs] [OrbitControls docs](https://threejs.org/docs/?q=orbit#examples/en/controls/OrbitControls)

`OrbitControls` allow the camera to orbit around a target.

It includes event listeners for click and drag movements, as well as keyboard keys.