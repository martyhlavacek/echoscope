# EchoScope

Room scanner research MVP that runs entirely in the browser. It started as an acoustic scanner: a phone or laptop speaker sends inaudible chirps, the microphone records the echoes, and each echo's delay gives a distance. Version 3.0 replaces the sonar in **room capture** with the camera and an **AI depth model running on the phone**. You photograph the room from one or more spots, and the photos become a textured 3D model of any room shape (in the spirit of Matterport) that you can orbit, export, or open in AR on an iPhone.

**Open it:** https://raw.githack.com/martyhlavacek/echoscope/room-capture/index.html (this branch).
`v2.1.html` keeps the previous sonar + selfie-camera room capture, and `v1.4.html` the acoustic scanner before room capture.

Camera, microphone and motion sensors only work over HTTPS, localhost or a local file. They don't work inside sandboxed previews. Photos are processed on the device and never uploaded.

## Room capture (v3.0)
1. Choose **Room capture** (the default on phones). Tap **Run room-capture demo** first to see the whole flow on a synthetic L-shaped room.
2. Set **Phone height above the floor**: how high you hold the phone, about chest height. Every distance is scaled from this, so measuring it once is worth it.
3. Tap **Start room capture** and allow **Camera** and **Motion & Orientation**. Hold the phone in front of you, upright, **screen facing you**. The back camera looks into the room and the screen shows what it sees.
4. Hold still a moment, then **turn slowly on the spot** through a full circle (about 30 s). Each tick is a photo.
5. At the chime, **tilt down to the floor and turn around once more**, then **look up at the ceiling** in a few directions (optional).
6. **Walk 2–3 m** to another part of the room, tap **Next spot**, and repeat. One spot is enough for a simple room; L-shapes and furnished rooms need 2–3. Neighbouring spots should see at least two of the same walls.
7. Tap **Build room**.

The first build downloads the AI depth model once: about 50 MB when the phone's GPU can run it, or 27 MB for the slower CPU version. The download starts while you capture, and the browser keeps it afterwards. **Get the AI depth model now** fetches it ahead of time (Wi-Fi recommended).

Afterwards:
- **Dollhouse / Floor plan / Inside** views. Tap **Inside** again to jump to the next spot.
- **straight walls** squares the floor plan along the room's two main directions. Untick it to see the plan as measured.
- **depth points** overlays the calibrated depth of every photo as a coloured point cloud.
- **Adjust room**:
  - **Phone height during capture** rescales the whole room (use it if everything came out uniformly too big or small).
  - **Ceiling height**.
  - **Rebuild** with changed camera settings.
- **Export**:
  - `.glb` for any 3D viewer or Blender.
  - `.usdz` at 1:10 table-top scale; **View in AR** opens it in AR Quick Look on iPhone/iPad.
  - A 360° panorama per spot.
- **Save raw capture**: a `.zip` with the photos, their orientations and the AI depth maps. **Open a saved raw capture…** rebuilds it later, including with newer versions, without running the AI again. It's the most useful thing to share when a room comes out wrong. If a build fails, the room capture settings offer **Save the last raw capture** and **Rebuild the last capture**. Raw captures saved by v2.x open too and are rebuilt from their photos.

How it works:
- **Depth per photo.** [Depth Anything V2 Small](https://huggingface.co/onnx-community/depth-anything-v2-small) (Apache-2.0) runs in the page through [onnxruntime-web](https://onnxruntime.ai) on WebGPU (half precision), or WebAssembly (8-bit) when the GPU can't run it. For each photo it gives relative depth: which things are nearer and farther, but not by how much in metres. That unknown scale and offset is different for every photo.
- **Calibration against the floor.**
  - In each photo aimed down, the floor must be a flat plane exactly your phone height below the camera. A robust search finds the scale and offset that put the most of the photo on that plane with nothing below it, since nothing can be below the floor.
  - A sofa seat or bed filling the view can still pass that test. So each anchor is also checked against the photos taken from the same place, and anchors that disagree are dropped.
- **Calibration against neighbours.** Overlapping photos must agree about the depth of what they both see. The phone travels on a circle in front of your body, so near furniture shifts between photos; that parallax is corrected, and occluded samples are skipped. Everything is solved together.
- **Aligning spots.** The motion sensors give each spot's heading. The spots' positions come from matching the walls each spot sees: an FFT cross-correlation of the wall maps, refined by 2-D ICP.
- **Floor plan.** It is the union of the free space each spot sees up to its walls. Walls are measured just under the ceiling, so furniture doesn't dent the plan. The outline is traced and, optionally, straightened.
- **Ceiling height** comes from the photos aimed up.
- **Textures.** Each spot's photos are stitched using their own depth. Every wall, floor and ceiling texel takes its colour from the spot that sees it most squarely and closest without anything in the way.

On the synthetic L-shaped test room (±3% depth error standing in for the AI), the rebuilt plan is 19.0 m² against 18.8 m² true, with all 6 corners. The ceiling comes out at 2.54 m against 2.50 m, and the second spot is placed within 3 cm. Real accuracy depends on how well the AI judges your room. That is what real captures will show.

Limitations:
- **Scale.** The scale comes from the phone height you enter: a 10 cm error is about an 8% error in every distance.
- **Difficult surfaces.** Mirrors, windows, glossy or blank walls, dim light and very large rooms (beyond about 8 m) can mislead the depth model.
- **Furniture** is painted onto the walls and floor rather than modelled.
- **Seams.** Photo placement relies on the motion sensors, so expect small seams.

## iPhone
- Open the link in Safari. Allow the camera, and **Motion & Orientation** access when asked (the sonar modes also need the microphone).
- In the sonar modes Safari plays sound through the **earpiece** while the mic is on, so the screen side is the sonar side (the default on iPhone). If the chirp never locks, switch the band to **15–20 kHz**.
- If the stitched panorama shows doubled edges, adjust **Camera field of view** (the iPhone main camera is about 68° across the long side of a photo).

## Other modes (sonar)
- **Phone motion sensors**: 3D point cloud. Each echo is placed along the measured aim. Tilt is measured from gravity; only the heading is zeroed.
- **Laptop stereo speakers**: 2D top-down map from a stationary laptop. The left speaker plays an up-chirp and the right a down-chirp; the mic position is learned automatically.
- **Manual yaw/pitch**: places echoes along an assumed direction, which gives one line.
- **Simulation**: synthetic scenes run through the same DSP pipeline.

Research prototype. It is not a calibrated depth sensor.
