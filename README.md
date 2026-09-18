# TurboWarp Photogrammetry App

**English** | [日本語](README.ja.md)

An app that reconstructs a space incrementally with WebGPU as you walk around carrying a calibrated camera.

## What's included

This is the initial scaffold generated from turbowarp-app-template. Use-case-specific features are not implemented.

- Mode selection, guidance, and error display built on the shared app-shell.
- Unpacked SB3 sources plus a startup-check script that updates a state variable when the green flag is clicked.
- Builds for the SB3 and the distribution page, SHA-256 recording, and CI.

The distribution page does not embed the TurboWarp player; it offers the startup-check SB3 for download. Run `build:sb3` before downloading from the dev server.

## Planned

- Load the lens calibration profile from a file produced by camera-calibration-app. Reject profiles that do not match the capture conditions, or whose match cannot be determined, before they are applied.
- In standalone mode, start from a fixed stereo rig and update depth, point cloud, and mesh incrementally while moving. Add monocular support in a later stage, once the initialization conditions and the basis for real-world scale have been confirmed.
- In cluster mode, complete pairing, time correspondence, and placement calibration before starting reconstruction. Never report insufficient quality, missing calibration, or a disconnection as success.
- Use the placement calibration result for the relative pose of the fixed rig, and let visual-tracking handle pose while moving. Redo placement calibration whenever the rig configuration changes.
- Guide the user through starting and stopping capture on the camera rig, tracking loss and recovery, and capture coverage and quality.
- Save keyframes along with calibration and timing information, and export the reconstruction result.
- Verify shape error, trajectory error, latency, and GPU memory on real hardware. 30 Hz tracking and 5–10 Hz shape updates are provisional targets.

## Modes

- **Standalone**: Runs entirely on a single PC. Requires no pairing or time synchronization; reconstruction starts given only lens calibration.
- **Cluster**: Uses cameras across multiple PCs. Reconstruction starts only after QR-carried pairing, time correspondence, and placement calibration against a common reference.

## Dependencies and responsibilities

- camera-source: the contract for images, capture settings, and the intrinsic calibration profile.
- camera-calibration-app: produces the intrinsic calibration profile. This app receives it as a file and contains no calibration procedure and no OpenCV.
- time-space-sync: time correspondence and fixed-rig placement. Used in cluster mode.
- webrtc-qrcode-pairing / webrtc: pairing and connection for cluster mode. Unused in standalone mode.
- visual-tracking: pose while moving, and the sparse map.
- photogrammetry: depth estimation, fusion, and shape generation.
- aframe: pose and projection settings, and rendering of the reconstruction result.

The only actual dependency is turbowarp-app-shell 0.2.0 in package.json. The use-case-specific connections above are planned, and do not rely on any unreleased early extension. When one is added, its exact version, artifact hash, API manifest, and evaluation order will be pinned.

## Layout and development

Node.js >=22.18.0, pnpm 11.11.0.

```bash
corepack enable
pnpm install --frozen-lockfile
pnpm check
pnpm dev
```

- `config/app.json`: name, modes, description, and planned work.
- `config/feature-flags.ts`: experimental feature flags, fixed at startup and OFF by default.
- `scripts/project.ts`: the source of truth for the startup-check SB3.
- `apps/main/source`: the unpacked SB3 sources, generated at build time (not tracked by Git).
- `src`: the distribution page built on the shared shell.
- `public/downloads`: the generated SB3 and release.json.
- `dist`: build output for the distribution page and downloads.

`pnpm build` generates `apps/main/source` from `project.ts`, then packs it with sb3-toolchain. The generated sources, SB3 files and `dist` are not tracked by Git.

## Staged rollout and acceptance criteria

1. In the related GitHub Issue, settle what to extract from the existing implementation, its dependencies, the DoD, and the rollback path.
2. Add the use-case-specific path behind a flag that is OFF by default, and replace the existing path with delegation.
3. Record error, latency, stalls, and recovery in hardware integration testing.
4. Do not reimplement the core extension's algorithms inside the app.

The DoD for the initial scaffold is: `pnpm check` passes, the SB3 updates its state on the green flag, and the distribution page shows the description, mode selection, and SB3 download. Real-device verification of camera-based features has not been performed.

## Rollback and task management

New paths are stopped by turning their flag OFF in `config/feature-flags.ts`, and compatibility reads for the old app path are kept during migration. Turning the initial flags ON does not implement any use-case-specific feature.

GitHub Issues are the source of truth for progress, recording start/done/blocked.

## Origin

The shared structure is extracted from the kamishibai (picture-story) app and realtime-motion-capture-app. See the [extraction notes](docs/extraction.md) (Japanese) for details.

## License

MPL-2.0. The package is private in its initial state.
