# EchoScope

Room scanner that runs entirely in the browser. You photograph a room with your phone from one or more spots. An **AI depth model running on the phone** turns the photos into distances, and they become a textured 3D model of any room shape (in the spirit of Matterport). You can orbit it, read the wall lengths off the floor plan, export it, or open it in AR on an iPhone.

**Open it:** https://raw.githack.com/martyhlavacek/echoscope/room-capture/index.html (this branch).

EchoScope started as an acoustic scanner (speaker chirps and echoes). Version 3 replaced the sonar with the camera. The acoustic versions are kept: `v2.1.html` (sonar + selfie-camera room capture) and `v1.4.html` (the acoustic 3D scanner).

The camera and motion sensors only work over HTTPS. Photos are processed on the device and never uploaded.

## Capturing a room
1. Set **Phone height above the floor** on the start screen: how high you hold the phone, about chest height. Every distance is scaled from it, so measure it once. The app remembers it, along with the other settings.
2. Tap **Start room capture** and allow **Camera** and **Motion & Orientation**. The capture runs full screen with the camera view.
3. Hold the phone in front of you, upright, **screen facing you**. Hold still a moment, then **turn slowly on the spot** through a full circle (about 30 s). Each tick is a photo.
4. At the chime, **tilt down to the floor and turn around again**, then **look up at the ceiling** in a few directions (optional).
5. **Walk 2–3 m** to another part of the room and tap **Next spot**, then repeat. One spot is enough for a simple room; L-shapes and furnished rooms need 2–3. Neighbouring spots should see at least two of the same walls.
6. Tap **Build room**.

Next spot and Build room stay on screen the whole time:
- Tapping **Next spot** before a spot is finished (full turn + floor) explains what's missing; a second tap skips that spot.
- **✕** cancels the capture but keeps the photos, which can then be rebuilt or saved under Capture files.

The first build downloads the AI depth model once: about 50 MB when the phone's GPU can run it, or 27 MB for the slower CPU version. The download starts while you capture, and the browser keeps it afterwards. **AI depth model → Get the AI depth model now** fetches it ahead of time (Wi-Fi recommended).

Tip: on iPhone, **Share → Add to Home Screen** runs EchoScope full screen without Safari's toolbars.

## The result
- **Views**:
  - **Dollhouse**.
  - **Floor plan**, with the length of every wall.
  - **Inside**. Tap Inside again to jump to the next spot.
- **straight walls** squares the plan along the room's two main directions. Untick it to see the plan as measured.
- **depth points** overlays the calibrated depth of every photo.
- **Adjust room**:
  - **Phone height during capture** rescales the whole room (use it if everything came out uniformly too big or small).
  - **Ceiling height**.
  - **Rebuild** with changed camera settings.
- **Export**:
  - `.glb` for any 3D viewer or Blender.
  - `.usdz` at 1:10 table-top scale; **View in AR** opens it in AR Quick Look on iPhone/iPad.
  - A 360° panorama per spot.
- **Save raw capture**: a `.zip` with the photos, their orientations and the AI depth maps. **Open a saved raw capture…** rebuilds it later, including with newer versions, without running the AI again. It's the most useful thing to share when a room comes out wrong. If a build fails, the error comes with **Save raw capture** and **Try the build again** buttons. Raw captures from v2.x open too and are rebuilt from their photos.

## When something goes wrong
- **Diagnostics log** (in the settings): the app records every capture and build step as it happens, with timings. The log also includes the depth engine's attempts (GPU or CPU, download, start-up), warnings and errors, and the device's details (no photos). **Copy log** puts it on the clipboard to paste into a message; **Download log** / **Share log** save it as a text file.
- **The log survives a crash.** It is written to the browser's storage as it goes, so it's still there if iOS kills the tab, which happens when the page runs out of memory.
- **The capture is saved before every build.** It is saved again once the depth maps exist, so a crash loses nothing. After a crashed build the page shows what it was doing when it stopped, with:
  - **Rebuild the saved capture**;
  - **Copy log**;
  - **Download log**;
  - **Download capture**.
- **Lighter settings for the next build.** If the last build crashed, the next one uses settings that need less memory. If it crashed while the GPU was computing depth, the next build uses the CPU instead.
- **Cancel build.** During a build, **Copy log** and **Cancel build** stay on screen. A step that takes unusually long is noted in the log.

## How it works
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

If the stitched panorama shows doubled edges, adjust **Camera → Field of view** (the iPhone main camera is about 68° across the long side of a photo).

Research prototype. It is not a calibrated measuring device.
