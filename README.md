# EchoScope

Acoustic 3D scanner research MVP that runs entirely in the browser. A phone or laptop speaker sends inaudible chirps, the microphone records the echoes, and each echo's delay gives a distance. Version 2.0 adds **room capture**: the sonar measures the room while the selfie camera photographs it, and the two are combined into a textured 3D "dollhouse" (in the spirit of Matterport) that you can orbit, export, or open in AR on an iPhone.

**Open it:** https://raw.githack.com/martyhlavacek/echoscope/main/index.html
(or https://martyhlavacek.github.io/echoscope/ once GitHub Pages is enabled for this repo). The previous version is kept at `v1.4.html`.

Microphone, camera and motion sensors only work over HTTPS, localhost or a local file. They don't work inside sandboxed previews. Nothing is uploaded; all processing happens on the device.

## Room capture (v2.0)
1. Choose **Room capture** (the default on phones). Tap **Run room-capture demo** first to see the whole flow on a synthetic room.
2. Tap **Start room capture** and allow **Camera & Microphone** and **Motion & Orientation**.
3. Hold the phone upright against your chest with the **screen facing out**: on iPhone the earpiece (the sonar) and the selfie camera both look into the room.
4. Hold still a moment, then **turn slowly on the spot** through a full circle (about 30 s). Each tick is a photo.
5. At the chime, **point the screen at the floor**, then **at the ceiling**. A final chime builds the dollhouse. You can press Stop at any time to build from what was captured.

Afterwards:
- **Dollhouse / Floor plan / Inside** views. Inside is the stitched photo sphere seen from where you stood.
- **Adjust room**: wall distances, floor and ceiling height and rotation. The photos are re-stitched onto the edited box.
- **Export**: `.glb` (any 3D viewer or Blender), `.usdz` (1:10 table-top scale; **View in AR** opens it in AR Quick Look on iPhone/iPad), and a 360° equirectangular `.png`.
- **Save raw capture**: a `.zip` with the photos, their orientations and the sonar data. **Open a saved raw capture…** rebuilds it later, including with newer versions, and it's the most useful thing to share when a room comes out wrong.

How it works:
- Every chirp's range profile is recorded with its aim.
- Echoes that are the same at every heading (your body, the phone itself) are removed using a heading-balanced turn average. The remaining heading-dependent echoes are mostly walls.
- A wide sonar beam hears a flat wall at its perpendicular distance, so each wall appears as a plateau of constant range. A rectangular room is fitted to these plateaus. The fit accounts for the phone travelling on a circle in front of your body axis (the "phone distance from your body axis" setting).
- In a furnished room the sonar can get the room's **rotation** badly wrong (30° in the first real capture). So the room is also lined up with the photos (v2.1): nearly every horizontal edge in a box-shaped room runs along one of two perpendicular directions. Voting with the edges of the stitched panorama gives the rotation without needing any distances. The sonar then re-measures the walls along those directions, and the ceiling and floor lines in the photos refine them when that clearly helps. Untick **align to photos** to see the sonar-only box.
- Floor and ceiling heights come from the chirps recorded while pointing down and up.
- The low-resolution photos are placed by the motion sensors, projected from where the phone actually was onto the fitted box (parallax-corrected), blended with exposure matching, and the gaps are inpainted.

Limitations:
- Rooms are modelled as a single rectangular box.
- Furniture, open doorways and soft furnishings weaken or confuse wall echoes. When the sonar hears few clear walls, the app says so and the box can be corrected by hand.
- Photo placement relies on the motion sensors only (no image alignment), so expect small seams.

## iPhone
- Open the link in Safari. Allow the microphone, camera and **Motion & Orientation** access when asked.
- While the mic is on, Safari plays sound through the **earpiece**, so the screen side is the sonar side (the default on iPhone).
- If the chirp never locks, switch the band to **15–20 kHz**. To hear which speaker is in use, try **1–4 kHz**.
- Turn off silent mode if there's no sound at all.

## Other modes
- **Phone motion sensors**: 3D point cloud. Each echo is placed along the measured aim. Tilt is now measured from gravity; only the heading is zeroed.
- **Laptop stereo speakers**: 2D top-down map from a stationary laptop. The left speaker plays an up-chirp and the right a down-chirp; the mic position is learned automatically.
- **Manual yaw/pitch**: places echoes along an assumed direction, which gives one line.
- **Simulation**: synthetic scenes run through the same DSP pipeline.

Research prototype. It is not a calibrated depth sensor.
