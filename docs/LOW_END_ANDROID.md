# Building for low-end Android devices (Samsung Galaxy Ace 4 Lite, SM-G313M)

This fork adds a `LITE_BUILD` profile aimed at very old, low-memory Android phones.

## Target hardware (SM-G313M)

| Spec | Value |
| --- | --- |
| Model | Samsung Galaxy Ace 4 Lite (SM-G313M) |
| Android | 4.4.2 KitKat (API 19) |
| CPU | Dual-core ARMv7 (32-bit) |
| GPU | Mali-400, OpenGL ES 2.0 only |
| RAM | ~512 MB |
| Screen | 480x800 |

## Build command

Follow [COMPILING_MOBILE.md](./COMPILING_MOBILE.md) to set up the Android toolchain, then:

```bash
lime build android -release -DLITE_BUILD
```

What `-DLITE_BUILD` does (see `project.hxp`):

- Compiles **ARMv7 only** (ARM64 is excluded): smaller APK, faster builds.
- **Disables ASTC textures.** The Mali-400 cannot decode ASTC, so PNG textures are used.
- **Disables** ads, in-app purchases, in-app review, webview and age signals (they need Google Play Services / newer Android).
- Sets `android.minimum-sdk-version` to **19**.
- Strips all `trace()` calls and removes hxcpp stack-tracking (`HXCPP_STACK_LINE`, `HXCPP_STACK_TRACE`, `HXCPP_CHECK_POINTER`), which add overhead to every function call. Crash reports will have less detail.
- Uses `-dce std` to drop unused Haxe standard library code (smaller binary). Mods that rely on unused Flixel/OpenFL classes may not work.
- Turns off sprite smoothing (`FlxSprite.defaultAntialiasing = false`) to save GPU fill-rate.
- Ignores sprite filters (blur/glow, etc.), which need an extra render texture per sprite.
- Doesn't permanently cache all UI textures at startup (`FunkinMemory.initialCache`), they load on demand to save RAM.

## Known limitations (please read)

- **This was not compiled or tested on a device.** Some of the code changes (`#if LITE_BUILD` blocks in `InitState.hx`, `FunkinSprite.hx`, `FunkinMemory.hx`) are untested. The changes were made without a Haxe/Lime toolchain available.
- The upstream game targets modern Android. Dependencies such as AndroidX, `extension-androidtools` and `hxvlc`
  may require a minimum SDK above 19. If the build fails with a manifest merge error, that dependency is the blocker.
- With ~512 MB of RAM and an OpenGL ES 2.0 GPU, the full game (large textures, HD audio, video cutscenes)
  may still run out of memory or run slowly even when it builds and installs.
- On mobile the game targets the display refresh rate with a floor of 60 FPS (`Preferences.framerate`),
  so there is no 30 FPS cap in this profile.

## Ideas if it still doesn't fit in memory

- Downscale large atlases/backgrounds (`scripts/optimizePNG.sh` is a starting point).
- Convert audio to lower-bitrate OGG/MP3 (`scripts/convertOGGToMP3.sh`).
- Disable video playback with `-DNO_FEATURE_VIDEO_PLAYBACK` to drop `hxvlc` (also removes a minimum-SDK blocker).
