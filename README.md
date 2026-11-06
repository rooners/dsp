# Dyson Sphere Program - Nebula Multiplayer Mod (AMP Template)

CubeCoders AMP Generic Module template for running a headless dedicated server for **Dyson Sphere Program** using **Wine**, **BepInEx 5 (x64)**, and the **Nebula Multiplayer Mod**.

---

## 1. Deploying the Template to AMP

### Option A: Via AMP Web UI (Recommended)
1. In your AMP ADS web panel (`http://<your_server>:8080/`):
2. Navigate to **Configuration** -> **Instance Deployment** -> **Template Repositories**.
3. Add `https://github.com/rooners/dsp` as a template repository.
4. Go to **Instances** -> **Create Instance** -> select **Dyson Sphere Program (Nebula Mod)**.

### Option B: Direct Copy to ADS Container
If your ADS instance runs inside Docker (`amp-ads`):
```bash
docker exec -i amp-ads bash -c "mkdir -p /home/amp/.ampdata/instances/ADS01/Plugins/ADSModule/GenericTemplates/dsp-nebula"
docker cp . amp-ads:/home/amp/.ampdata/instances/ADS01/Plugins/ADSModule/GenericTemplates/dsp-nebula/
```

---

## 2. Server Files Setup (SFTP Upload)

Because Dyson Sphere Program is a commercial Steam game without an anonymous SteamCMD server build, copy your game files and BepInEx installation into the instance root directory via SFTP (Port `2225` by default in AMP).

### Instance Root Directory:
`/DATA/AppData/amp/.ampdata/instances/<InstanceName>/`

### Directory Layout Inside the Instance:
```text
<InstanceName>/
├── DSPGAME.exe
├── DSPGAME_Data/
│   ├── Plugins/
│   │   └── x86_64/
│   │       └── steam_api64.dll     (Replaced with Detanup01/gbe_fork build)
├── MonoBleedingEdge/
├── UnityCrashHandler64.exe
├── UnityPlayer.dll
├── steam_appid.txt                 (1366540 - included in this repo)
├── steam_api64.dll                 (Replaced with Detanup01/gbe_fork build)
├── winhttp.dll                     (From BepInEx 5 x64)
├── doorstop_config.ini             (From BepInEx 5 x64)
└── BepInEx/
    ├── core/
    ├── config/
    │   ├── BepInEx.cfg             (Ensure [Logging.Console] Enabled = true)
    │   └── nebula.cfg              (Multiplayer server config)
    ├── patchers/
    │   └── NebulaPatcher.dll
    └── plugins/
        ├── NebulaMultiplayerMod/
        │   └── NebulaMultiplayerMod.dll
        └── ...
```

---

## 3. Critical Configuration Steps

### 1. Steam Emulation (Headless Requirement)
Because headless Linux servers do not run the Steam desktop client (`Steam.exe`), Dyson Sphere Program's `SteamAPI.Init()` will fail and abort without an emulator.
- Download the latest 64-bit `steam_api64.dll` from **[Detanup01/gbe_fork Releases](https://github.com/Detanup01/gbe_fork/releases)** (asset `emu-win-release-vs22.7z` or `emu-win-release.7z`).
- Replace `steam_api64.dll` in both:
  1. The instance root directory
  2. `DSPGAME_Data/Plugins/x86_64/`
> **Important:** Use the **Detanup01 fork** of Goldberg emulator. The older original Goldberg build lacks modern `ISteamUserStats` exports required by Dyson Sphere Program, which causes an `EntryPointNotFoundException` and stops Nebula from binding port 8469.

### 2. Wine DLL Override
The template automatically injects `WINEDLLOVERRIDES="winhttp=n,b"` so Wine loads BepInEx's `winhttp.dll` hook instead of Wine's internal builtin library.

### 3. Live Console Logging
In `BepInEx/config/BepInEx.cfg`, verify:
```ini
[Logging.Console]
Enabled = true
```
This enables Unity and Nebula server output to stream directly to AMP's web console.

### 4. Startup Mode (New Game vs Existing Save)
- **First Launch (Generate Galaxy):** Launch with `-newgame-default` in the command line arguments to create the universe.
- **Subsequent Launches (Resume Save):** In `GenericModule.kvp` (or via AMP settings), use `-load-latest` to ensure restarts resume your progress without generating a new game:
  ```ini
  App.LinuxCommandLineArgs=-a wine "./DSPGAME.exe" -batchmode -nographics -nebula-server -load-latest -ups 60
  ```

---

## 4. Ports & Networking

* **Port 8469 (TCP & UDP):** The default port used by Nebula. Ensure port 8469 is forwarded or accessible via VPN (e.g. ZeroTier).
* **Auto-Pause:** In `BepInEx/config/nebula.cfg`, `AutoPause = true` pauses simulation when all players disconnect, keeping idle CPU and memory consumption minimal.
