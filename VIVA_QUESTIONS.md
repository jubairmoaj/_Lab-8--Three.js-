# Three.js Car Road Project Viva Notes

Use these answers as a guide, then explain them in your own words.

## Quick Project Summary

This project is a Three.js scene where a real Ferrari GLB model is placed on an animated road. The road moves backward/forward to create the feeling that the car is driving. Buildings and trees are added beside the road and move with the road. The sky is a procedural night sky with gradient color, moving clouds, blinking stars, and a moon. The mouse controls a point light source, keyboard keys control the camera, and the mouse wheel controls the light height.

## Important Files

- `texture.html`: Main project file. It creates the scene, camera, lights, road, model loader, controls, and animation loop.
- `assets/models/ferrari.glb`: The real car model used in the scene.
- `assets/models/README.md`: Credit/source information for the car model.
- `js/three.js`: Main Three.js library.
- `js/GLTFLoader.js`: Loads `.gltf` and `.glb` 3D model files.
- `js/DRACOLoader.js`: Helps load Draco-compressed GLB models.
- `js/libs/draco/gltf/draco_wasm_wrapper.js`: Draco decoder wrapper.
- `js/libs/draco/gltf/draco_decoder.wasm`: Draco decoder binary.

## Common Viva Questions And Easy Answers

### 1. What is this project about?

It is a 3D car-on-road scene made with Three.js. I load a real Ferrari car model, place it on a road, move the road segments to create motion, rotate the wheels, and control the light with the mouse.

### 2. Which file is the main file?

The main file is `texture.html`. Almost all my project logic is inside this file.

### 3. How did you get/download the car model?

I used an open-source Ferrari GLB model from the three.js examples project. The model source is mentioned in `assets/models/README.md`. I saved the model inside my project as `assets/models/ferrari.glb`.

### 4. Which car model file did you add?

I added:

```text
assets/models/ferrari.glb
```

This is the 3D car model file.

### 5. What is a `.glb` file?

`.glb` is a binary glTF file. It can contain 3D geometry, materials, textures, and scene information in one file. It is commonly used for 3D models on the web.

### 6. How do you show the car on the road?

In `texture.html`, I use `THREE.GLTFLoader()` to load `assets/models/ferrari.glb`. After loading, I get the model from `gltf.scene`, scale it with `fitImportedCarToRoad()`, prepare the materials, collect the wheel meshes, and add it to `importedCarGroup`.

Key code area:

```js
carLoader.load("assets/models/ferrari.glb", function (gltf) {
    const importedCar = gltf.scene;
    fitImportedCarToRoad(importedCar);
    importedCarGroup.add(importedCar);
});
```

### 7. Why do you use `GLTFLoader.js`?

Three.js by itself does not directly load GLB files with the core library. `GLTFLoader.js` is an extra Three.js loader that understands `.gltf` and `.glb` models.

### 8. Why do you use `DRACOLoader.js`?

The Ferrari model is Draco-compressed, which means the mesh data is compressed to reduce file size. `DRACOLoader.js` decodes that compressed geometry so Three.js can display the model.

### 9. Which Draco files are required?

These two decoder files are used:

```text
js/libs/draco/gltf/draco_wasm_wrapper.js
js/libs/draco/gltf/draco_decoder.wasm
```

The loader path is set here:

```js
dracoLoader.setDecoderPath("js/libs/draco/gltf/");
```

### 10. Where is the road code?

The road code is in `texture.html`:

- `createRoadTexture()` creates the asphalt texture, side lines, and center dashed line.
- The `roadMaterial` uses that texture with a custom shader.
- The `for` loop with `roadSegmentCount` creates several road planes.
- The `animate()` function moves those road segments every frame.

### 11. How is the road made?

The road is made from flat plane geometries:

```js
new THREE.PlaneGeometry(30, roadSegmentLength, 1, 1)
```

Each plane is rotated flat using:

```js
road.rotation.x = -Math.PI / 2;
```

Then the segments are placed one after another along the Z axis.

### 12. How does the road look endless?

There are multiple road segments. In every frame, each segment moves along the Z axis. When a segment goes too far, it is moved back to the other side. This recycling creates an endless-road effect.

Key idea:

```js
segment.position.z += roadMoveSpeed * delta;
if (segment.position.z > roadHalfSpan) {
    segment.position.z -= roadTrackLength;
}
```

### 13. Is the car actually moving?

The car stays near the center of the scene. The road moves under it, and the wheels rotate. This creates the visual feeling that the car is moving.

### 14. How do the wheels rotate?

After loading the model, I collect meshes whose names look like wheels or rims in `collectImportedCarWheels()`. Then in the `animate()` loop, I rotate those meshes every frame.

```js
importedWheelMeshes[i].rotation.x -= wheelSpin * delta;
```

### 15. How does the mouse control the light?

Mouse position is 2D screen data, but the scene is 3D. I use `THREE.Raycaster()` to cast a ray from the camera through the mouse position. The ray hits the road plane, and the point light moves above that hit point.

### 16. Why do you use `Raycaster`?

Raycaster converts mouse screen position into a 3D position in the scene. Without it, the mouse only gives X/Y screen pixels, not a real 3D world position.

### 17. What does the mouse wheel do?

The mouse wheel changes the height of the point light. The height is clamped between `minHeight` and `maxHeight` so it does not go too low or too high.

### 18. What lights are used?

I use three lights:

- `AmbientLight`: soft overall light.
- `PointLight`: main light controlled by the mouse.
- `HemisphereLight`: extra sky/ground fill light.

### 19. Where is the camera code?

The camera is created with:

```js
new THREE.PerspectiveCamera(55, window.innerWidth / window.innerHeight, 0.1, 500)
```

The `updateCamera()` function places the camera around the car using `Math.cos()` and `Math.sin()`.

### 20. How does keyboard control work?

The program stores pressed keys in `keyState`. In every animation frame, `handleKeyboard(delta)` checks which keys are pressed and updates camera angle, height, or zoom distance.

Controls:

- `A/D` or `Left/Right`: rotate camera around car.
- `W/S` or `Up/Down`: change camera height.
- `Q/E`: zoom out/in.

### 21. What is the animation loop?

The animation loop is the `animate()` function. It runs continuously using `requestAnimationFrame(animate)`. It updates camera controls, light position, road movement, wheel rotation, and finally renders the scene.

### 22. What is `delta` time?

`delta` is the time between the current frame and the previous frame. I use it so movement speed stays smooth even if the frame rate changes.

### 23. Why is there a custom shader?

The shader is used mainly for the road material. It lets the road texture react to the mouse-controlled point light with diffuse and specular lighting.

### 24. What is `uLightPos`?

`uLightPos` is a shader uniform. It sends the point light position from JavaScript to the shader so the road can be shaded based on the light position.

### 25. Why do you need a local server?

Browsers often block loading local model files directly from `file://`. A local server like Live Server serves the files with `http://127.0.0.1:5500/`, so `GLTFLoader` can load the GLB and decoder files correctly.

### 26. What old/extra code was removed?

The old handmade/toy car code was removed. The project now uses the real Ferrari GLB model. Unused visual helper lines/circles for the light were also removed.

### 27. What part did AI help with?

AI helped me modify and organize the code, but I understand the important parts: model loading, road creation, mouse light control, camera control, and animation.

### 28. Where is the building and tree code?

The building and tree code is in `texture.html` near the road creation code. The important functions are:

- `createBuilding()`: creates a building from box geometry and adds small window boxes.
- `createTree()`: creates a tree using a cylinder trunk and cone-shaped leaves.
- `createRoadsideSegment()`: creates buildings and trees for one road segment.

### 29. Did you import the buildings and trees?

No. I made them using basic Three.js geometry. Buildings use `BoxGeometry`, tree trunks use `CylinderGeometry`, and tree tops use `ConeGeometry`. This keeps the project simple and easier to explain.

### 30. How do the buildings and trees move with the road?

For every road segment, I also create one matching roadside group and store it in `roadsideSegments`. In the animation loop, when a road segment moves, the matching roadside group gets the same Z position.

```js
roadsideSegments[i].position.z = segment.position.z;
```

So buildings and trees move together with the road and recycle with it.

### 31. How did you make the sky realistic?

I made the sky procedurally in `texture.html`. The function `createSkyTexture()` creates a canvas texture with a dark-to-light gradient and horizon glow. Then I put that texture on a large sphere using `SphereGeometry`. The stars and clouds are separate animated objects.

### 32. Why is the sky sphere using `THREE.BackSide`?

The camera is inside the sky sphere. Normally a sphere shows its outside surface, but I need to see the inside surface. `THREE.BackSide` makes the inside of the sphere visible.

### 33. Did you import the sky image?

No. The sky is generated by JavaScript canvas, so no extra sky image is needed. The moon is also created using Three.js geometry, and the glow is created with a small canvas texture.

### 34. How do the clouds move?

The clouds are `THREE.Sprite` objects using a transparent canvas cloud texture. In the animation loop, `updateSkyAnimation()` changes each cloud's Z position slowly. When a cloud goes too far, it is moved back to the starting side.

### 35. How do the stars blink?

The stars are created as a `THREE.Points` star field. Each star has a random phase and speed attribute. A small shader uses `sin(time)` to change each star's brightness, which creates a twinkling/blinking effect. I also keep a clear area near the moon so the moon glow looks more natural.

## Short Explanation To Say In Viva

My project uses Three.js to create a 3D scene. I create a camera, renderer, lights, and road. The road is made from multiple plane segments with a canvas-generated road texture. I load a real Ferrari model from `assets/models/ferrari.glb` using `GLTFLoader`. Because the model is compressed, I use `DRACOLoader` with the Draco WASM decoder files. I scale and position the model on the road, then collect its wheel meshes so I can rotate them in the animation loop. The car stays mostly fixed, while the road moves and the wheels rotate to create the driving effect. I also create procedural buildings and trees using basic geometry, and they move with the road segments. For the sky, I generate a canvas texture and apply it to the inside of a large sphere, with a moon and glow. I use sprites for moving clouds and a shader-based point field for blinking stars. The mouse controls a point light using Raycaster, and keyboard keys control the camera.
