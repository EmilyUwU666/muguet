# FivePort (iOS, based on Muguet)

FivePort is a custom iOS build based on [Muguet](https://github.com/c22dev/muguet) by Constantin Clerc (c22dev). The original open-source license and contributors' credits are retained. This fork preserves the native Mach-O/Metal port and multiplayer support.

## Performance and low-memory support (iPhone 15 Pro and iPad Air M3)

FivePort has **automatic Low Memory Mode** for iPhone/iPad models with approximately 8 GB of RAM or less, and devices with limited app memory. This includes the iPhone 15 Pro and iPad Air M3. The launcher also lets you toggle the mode manually if you want to experiment.

When enabled, it uses:
- **30% rendering resolution** (minimum output size and aspect-ratio rules still apply)
- **Normal textures** (texture-quality setting 0)
- **30 fps** frame limit
- **MSAA/TXAA disabled**
- **Conservative startup settings** even if older `config.json` values were left behind

Higher-memory devices retain the existing Balanced/Quality graphics defaults. To turn off Low Memory Mode and select your own graphics preset, use the toggle on the launcher. Low Memory Mode makes the app less demanding but cannot guarantee that the game runs within iOS's process memory limit; if you experience a crash, check the on-device `muguet-debug.log` and iOS analytics.

The TestFlight version still uses an **8 GiB virtual Wasmtime memory reservation** that must match its embedded precompiled game code. Virtual reservation is not the same as physical RAM consumption. The reservation is deliberately left unchanged; selecting Low Memory Mode adjusts rendering workload and in-game textures, not that engine reservation.

## Set up your game files on iPhone or iPad

FivePort does **not** download or provide GTA V game assets. You must supply files you have the rights to use, prepared for the compatible WebAssembly game port.

**Recommended: import a game ZIP**

1. In Files, find a ZIP containing `data/` and `b/<build-id>/shaders/`. The `data/` folder must contain `manifest.json` and all the assets referenced by that manifest; `shaders/` must contain `index.json` and its required shader packs.
2. Open FivePort's launcher and choose **Import game data .zip**. Select the ZIP from Files and wait until the import completes. Keep the app open during import.
3. If the file picker cannot access your ZIP, copy the ZIP to **Files → On My iPhone → FivePort**, reopen the launcher, and select the newly listed ZIP.
4. When the launcher says the game data is ready, choose **Play**. For first-time troubleshooting on devices with 8 GB RAM or less, leave **Low Memory Mode** on and enable **Show frame rate**.

**Alternative: copy the extracted folders manually**

In **Files → On My iPhone → FivePort → game**, create the following layout:

```text
game/
├── data/
│   ├── manifest.json
│   ├── common/
│   ├── x64/
│   └── ...other assets listed in manifest.json
└── b/
    └── <build-id>/
        └── shaders/
            ├── index.json
            └── ...shader pack files
```

The build ID is whatever your compatible game export uses. The `data` and `b` directories must be immediately inside `game`, **not** inside an additional nested folder. Do not rename the build ID. The FivePort TestFlight build embeds precompiled game code, so you normally do **not** need to copy `game.wasm` into the iPhone app; this differs from ordinary Muguet source builds.

**If Play is unavailable:** verify both `data/manifest.json` and `b/<build-id>/shaders/index.json` exist. **If Play opens a black screen:** ensure the entire asset export is present, not just the manifest and shaders. To see engine errors, open `muguet-debug.log` in the app's Files folder after trying to launch.

The app's **How to set up game files** button displays these instructions on-device. If you previously installed this app under the old Muguet label, its Home Screen/Files name will change to FivePort after installing the updated TestFlight build.


## How does this work ?
this whole port relies on the WebAssembly compilation of the game. in a browser, that `game.wasm` runs inside the JS engine, behind WebGPU, Web Audio, web workers...
Muguet throws that layer away and runs the same `game.wasm` as a regular native app, with [wasmtime](https://wasmtime.dev) for the code and [wgpu](https://wgpu.rs) on Metal for the graphics

On macOS, the game code is compiled to native arm64 the first time you launch it, then cached. On iOS, we can't properly do that (i dont rlly want you to deal with the hassle of enabling JIT), so the game code is compiled when you build the app and baked into it

On top of that, Muguet adds a small UDP multiplayer (for other people who have the same port to play with you)

## Game files
Because I care about me not getting sued, you will need your own copy as a `.zip` : the folder holding `data/` and `b/<build>/` (with `game.wasm` and `shaders/`)

It's basically the same source as the web port.

Muguet itself will then asks for that zip

## Requirements
- Apple Silicon Mac, macOS 14 or later
- Rust toolchain
- Xcode 26 or later

For iOS, also :
- `rustup` with `rustup toolchain install 1.98.1` and `rustup target add aarch64-apple-ios --toolchain 1.98.1`
- `xcodegen` (`brew install xcodegen`)
- an Apple developer account
- a device on iOS 15 or later, ideally 8 GB of RAM

## Building
The easy way, paste this in Terminal :
```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/c22dev/muguet/main/build.sh)"
```
It downloads Muguet into `/tmp/muguet`, installs what's missing (Homebrew, Rust...), then asks whether you want the Mac or the iOS version and walks you through the rest. The `.app` / `.ipa` end up in `~/Downloads/muguetOutput`. Already have the repo ? Double-click `build.command`, or run `./build.sh`

You need Xcode 26 or later from the App Store. To install on your iPhone / iPad, sign in to Xcode with your Apple ID (Settings > Accounts), a free one works

By hand, macOS :
```
./make-app.sh
```
gives `build/Muguet.app`

iOS :
```
TEAM=<your team id> ./make-ios.sh game.zip --install
```
`--ipa` makes `build/Muguet.ipa` instead. With a free account, add `FREE=1` (+ a `BUNDLE_ID` of your own)

Please install this app with AltStore. This probably won't work well with Sideloadly (or even SideStore, as per [this issue])(https://github.com/SideStore/SideStore/issues/1616)

The app also works with a paid developer account.

## Multiplayer
One player hosts from the launcher (or runs `muguet-server` on any machine), the others join with that address. UDP port 47520

## Limitations
- iOS needs the increased memory limit entitlement
- Story missions are single player only

## Credits
- [divyanshugarg36](https://github.com/divyanshugarg36) for lots of iOS fixes and improvements
- [Kagander](https://github.com/Kagander) for fixing the minimap
- [inj-src](https://github.com/inj-src) for decoding bc textures on all cores
- [Dbhardwaj99](https://github.com/Dbhardwaj99) for reporting real free disk space
- [dre4moff](https://github.com/dre4moff) for releasing controller input when the app loses focus

Contributions are welcome !
