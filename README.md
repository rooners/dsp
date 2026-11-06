# Dyson Sphere Program - Nebula Multiplayer Mod (AMP Template)

CubeCoders AMP Generic Module template for running a headless dedicated server for **Dyson Sphere Program** using **Wine**, **BepInEx 5 (x64)**, and the **Nebula Multiplayer Mod**.

---

## 1. Deploying the Template to AMP

### Option A: Via AMP Web UI (Recommended)
1. Go to your AMP ADS web panel (`http://<your_server>:<port>/`).
2. Navigate to **Configuration** -> **Instance Deployment** -> **Template Repositories**.
3. Add `https://github.com/rooners/dsp` as a new template repository.
4. Go to **Instances** -> **Create Instance** -> select **Dyson Sphere Program (Nebula Mod)**.

### Option B: Direct Copy to ADS Container
If your ADS instance runs inside Docker (`amp-ads`):
```bash
docker exec -i amp-ads bash -c "mkdir -p /home/amp/.ampdata/instances/ADS01/Plugins/ADSModule/GenericTemplates/dsp-nebula"
docker cp . amp-ads:/home/amp/.ampdata/instances/ADS01/Plugins/ADSModule/GenericTemplates/dsp-nebula/
```

---

## 2. Server Files Setup (SFTP Upload)

Because Dyson Sphere Program is a commercial Steam game without an anonymous SteamCMD server build, you must upload your game files into the instance's **`GenericApplication`** folder via SFTP.

### Exact Upload Path:
`/DATA/AppData/amp/.ampdata/instances/DysonSphereProgramNebulaMod01/GenericApplication/`

### Upload Directory Layout Inside `GenericApplication/`:
```text
/GenericApplication/
├── DSPGAME.exe
├── DSPGAME_Data/
├── UnityCrashHandler64.exe
├── UnityPlayer.dll
├── steam_appid.txt             (Included in this repo with ID 1366540)
├── winhttp.dll                 (From BepInEx 5 x64)
├── doorstop_config.ini         (From BepInEx 5 x64)
└── BepInEx/
    ├── core/
    ├── config/
    │   ├── BepInEx.cfg         (Ensure [Logging.Console] Enabled = true)
    │   └── nebula.cfg          (Included in this repo)
    ├── patchers/
    │   └── NebulaPatcher.dll   (From Nebula Mod)
    └── plugins/
        └── NebulaMultiplayerMod/
            └── NebulaMultiplayerMod.dll
```

### Critical Steps
1. **BepInEx Architecture:** Ensure you install **BepInEx 5 x64** (64-bit).
2. **DLL Override:** The template automatically sets `WINEDLLOVERRIDES="winhttp=n,b"` so Wine loads BepInEx.
3. **Console Logging:** In `BepInEx/config/BepInEx.cfg`, change:
   ```ini
   [Logging.Console]
   Enabled = true
   ```
   This allows AMP's web console to display live server status and logs.
4. **Saves:** Place existing `.dsv` game saves into `GenericApplication/DSPGAME_Data/Save/` (or the Wine save directory).

---

## 3. Ports & Networking

* **Port 8469 (TCP & UDP):** The primary port used by Nebula. Ensure port 8469 is forwarded on your router if players are connecting over the internet.
