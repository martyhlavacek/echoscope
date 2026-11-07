# EchoScope

Acoustic 3D scanner research MVP that runs entirely in the browser. A phone or laptop speaker sends inaudible chirps, the microphone records the echoes, and each echo's delay gives a distance. Echo directions come from the phone's motion sensors (3D) or, on a laptop, from triangulating between the two speakers (2D).

**Open it:** https://raw.githack.com/martyhlavacek/echoscope/main/index.html
(or https://martyhlavacek.github.io/echoscope/ once GitHub Pages is enabled for this repo)

Microphone and motion sensors only work over HTTPS, localhost or a local file. They don't work inside sandboxed previews. Nothing is uploaded; all processing happens on the device.

## iPhone
- Open the link in Safari and tap **Start**. Allow the microphone and **Motion & Orientation** access when asked.
- While the mic is on, Safari plays sound through the **earpiece**, so aim with the **screen side** (this is the default on iPhone). Hold still for a second while the chirp locks, then sweep slowly.
- If the chirp never locks, switch the band to **15–20 kHz**. To hear which speaker is in use, try **1–4 kHz**.
- Turn off silent mode if there's no sound at all.

## Modes
- **Phone motion sensors**: 3D cloud. Each echo is placed along the measured aim.
- **Laptop stereo speakers**: 2D top-down map from a stationary laptop. The left speaker plays an up-chirp and the right a down-chirp; the mic position is learned automatically.
- **Manual yaw/pitch**: places echoes along an assumed direction, which gives one line.
- **Simulation**: synthetic room run through the same DSP pipeline.

Research prototype. It is not a calibrated depth sensor.
