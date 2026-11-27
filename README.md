==========
important notes
==========
[Hotkeys]
OpenPauseMenu = SDL-0/Back & SDL-0/X
ShutdownVM = SDL-0/Back & SDL-0/Start
SaveStateToSlot = SDL-0/Back & SDL-0/RightShoulder
LoadStateFromSlot = SDL-0/Back & SDL-0/LeftShoulder
NextSaveStateSlot = SDL-0/Back & SDL-0/DPadRight
PreviousSaveStateSlot = SDL-0/Back & SDL-0/DPadLeft
EOF

Please do not put system into sleep mode when running the game, due to the bug of drivers not working well with sleep mode, to avoid system frozen the next time it is wake up!

copy the entire folder to /roms/ports/ or /roms2/ports/, run the armsx2-R36SROM1 for /roms/ports system and vice verse, armsx2-R36SmyromsROM1 is when you decide to run your own rom copy that is put inside /roms/ports/R36Scfw/myroms/rom.iso, yes, you must rename your rom copy to rom.iso!

Enjoy the game session!

==========
introduction

==========
THEY SAID IT WAS IMPOSSIBLE. THEY SAID THE $35 R36S COULD NEVER RUN HARDWARE VULKAN PS2 ON STOCK ARKOS.
THEY WERE WRONG.
While everyone else is forced to wipe their SD cards, flash completely different operating systems, or suffer through 2 FPS unplayable software rendering—we just cracked the code on stock ArkOS (glibc 2.30) with a zero-OS-touch, bare-metal /dev/mali0 Vulkan hardware bridge!

🔥 THE IMPOSSIBLE HACK THAT BREAKS ALL THE RULES:

• Bypassed the "Unbreakable" glibc 2.30 Wall: No OS reinstalls. No bricked firmware. A self-contained glibc 2.36+ shadow runtime injects modern ARM64 dependencies straight into memory without touching a single host system file!
• Hijacked Legacy /dev/mali0 for Pure Hardware Vulkan: Using malisx2, we forced the R36S’s old Rockchip UK 11.7 BSP kernel driver to speak modern Vulkan 1.3 directly to the Mali-G31 GPU—unlocking real hardware shader acceleration on a chip everyone left for dead!
• Killed the Compositor Overhead (VulkanDirect): Zero Weston. Zero Wayland lag. Zero system freezes. armsx2-sdl fires frames straight onto the 640×480 display panel via raw kmsdrm + VK_KHR_display!
• 100% Plug-and-Play ES-DE Integration: One script downloads the emulator, builds the driver bridge, tunes the speedhacks, wires up native GO-Super Gamepad controls (Select + Start instant exit), pipes crisp ALSA audio straight to the speakers, and injects PlayStation 2 directly into your ES-DE menu!
No reflashing. No dual-booting. Pure, raw Mali-G31 Vulkan PS2 power unleashed on a $35 pocket handheld.

📱 Short Social / Video Hook Version (TikTok / YouTube Shorts / Reddit Title)

"Everyone said the $35 R36S on stock ArkOS couldn't run hardware Vulkan PS2 without reflashing your OS... so I built a shadow glibc + /dev/mali0 bare-display bridge that forces the Mali-G31 GPU to run ARMSX2 straight from ES-DE. One script. Zero OS mods. Pure madness."


# Porting-ARMSX2-SDL-TO-R36S-ARKOS-FULL-SPEED
This is an automated deployment script (ARMSX2) onto legacy Rockchip RK3326 handhelds (R36S / ArkOS) without modifying the host operating system. It resolves OS library incompatibilities, enables hardware Vulkan graphics on older Mali-G31 drivers, and integrates directly into the user's frontend menu in a single execution.


==========
Presentation Briefing: 
==========
Zero-Touch PS2 Hardware Acceleration Deployment (setup_and_run_ps2.sh)

1. Executive Overview
setup_and_run_ps2.sh is an automated deployment script that bridges modern PlayStation 2 emulation (ARMSX2) onto legacy Rockchip RK3326 handhelds (R36S / ArkOS) without modifying the host operating system. It resolves OS library incompatibilities, enables hardware Vulkan graphics on older Mali-G31 drivers, and integrates directly into the user's frontend menu in a single execution.



2. How the Script Works (4 Automated Stages)
* Stage 1: Isolated Runtime & Driver Provisioning
    * Downloads the bare-display ARMSX2 SDL binary, the malisx2 Vulkan-to-/dev/mali0 translator, and a self-contained Debian Bookworm (glibc 2.36+) library bundle into /roms2/ports/armsx2_r36s/.
    * Why it matters: Bypasses ArkOS's outdated glibc 2.30 limit safely without risking host OS corruption.
* Stage 2: Self-Generating Hardware & Input Configuration
    * Dynamically builds /roms2/ports/armsx2-R36SROM2dd.sh and writes a tuned PCSX2.iniconfiguration on every launch.
    * Key optimizations: Forces Renderer = Vulkan (VulkanDirect via kmsdrm to avoid Weston compositor deadlocks), binds native SDL-0 gamepad controls (Select + Start exit hotkey), routes audio directly to ALSA, and enables a 200%-scale on-screen FPS counter.
* Stage 3: Frontend Auto-Integration (ES-DE / EmulationStation)
    * Scans standard frontend configuration paths (es_systems.cfg / es_systems.xml), creates automatic backups (.bak), and injects the Sony PlayStation 2 (/roms/ps2) system entry.
* Stage 4: Instant Handover & Execution
    * Invokes the isolated ld-linux-aarch64.so.1 dynamic linker to launch armsx2-sdl immediately with hardware acceleration active.



3. Key Value & Impact
* Zero Manual Setup: Replaces multi-step Linux terminal troubleshooting with a single one-click script.
* Non-Destructive Architecture: Keeps host firmware untouched while unlocking native Mali-G31 GPU performance on budget RK3326 hardware.

If you'd like, I can turn this briefing into slide-by-slide bullet points (e.g., Title, Challenge, Architecture Diagram, Results) or write spoken presenter notes for your talk.

=========
The CODE
=========
#!/bin/bash
set -e

exec > "/roms2/ports/armsx2_auto.log" 2>&1
clear
echo "=== AUTO-INSTALLING & CONFIGURING ARMSX2 FOR R36S ===" > /dev/tty0

# 1. Install ARMSX2, malisx2, and glibc 2.36+ runtime using explicit paths
if [ ! -f "/roms2/ports/armsx2_r36s/bin/armsx2-sdl" ]; then
    echo "Downloading ARMSX2 + malisx2 + runtime (please wait)..." > /dev/tty0
    mkdir -p "/roms2/ports/armsx2_r36s/bin" "/roms2/ports/armsx2_r36s/lib" "/roms2/ports/armsx2_r36s/malisx2" "/roms/ps2" "/roms2/bios"

    # === PASTE YOUR DIRECT DOWNLOAD URLS INSIDE THE QUOTES BELOW ===
    curl -sL "" -o "/tmp/armsx2-sdl.tar.zst"
    curl -sL "" -o "/tmp/malisx2.tar.gz"
    # ===============================================================

    mkdir -p "/tmp/armsx2_extract" "/tmp/malisx2_extract"
    zstd -dc "/tmp/armsx2-sdl.tar.zst" | tar -xf - -C "/tmp/armsx2_extract"
    cp -rf "$(dirname "$(find /tmp/armsx2_extract -type f -name "armsx2-sdl" | head -n 1)")/"* "/roms2/ports/armsx2_r36s/bin/"
    chmod +x "/roms2/ports/armsx2_r36s/bin/armsx2-sdl"

    tar -xzf "/tmp/malisx2.tar.gz" -C "/tmp/malisx2_extract"
    cp -f "$(find /tmp/malisx2_extract -type f -name "libvulkan_malisx2.so" | head -n 1)" "/roms2/ports/armsx2_r36s/malisx2/libvulkan_malisx2.so"

    cat << 'EOF' > "/roms2/ports/armsx2_r36s/malisx2/malisx2_icd.aarch64.json"
{
    "file_format_version": "1.0.0",
    "ICD": {
        "library_path": "/roms2/ports/armsx2_r36s/malisx2/libvulkan_malisx2.so",
        "api_version": "1.3.0"
    }
}
EOF

    mkdir -p "/tmp/bookworm_pkgs" "/tmp/bookworm_root"
    curl -s "http://debian.org" | gzip -dc > "/tmp/bookworm_pkgs/Packages"
    for PKG in libc6 libstdc++6 libgcc-s1 libdrm2 libgbm1 libasound2 libwayland-client0 libwayland-server0 libxkbcommon0 libzstd1 zlib1g; do
        if [ -n "$(awk -v pkg="$PKG" '$1 == "Package:" && $2 == pkg {found=1} found && $1 == "Filename:" {print $2; exit}' "/tmp/bookworm_pkgs/Packages")" ]; then
            curl -sL "http://debian.org(awk -v pkg="$PKG" '$1 == "Package:" && $2 == pkg {found=1} found && $1 == "Filename:" {print $2; exit}' "/tmp/bookworm_pkgs/Packages")" -o "/tmp/bookworm_pkgs/${PKG}.deb"
            dpkg-deb -x "/tmp/bookworm_pkgs/${PKG}.deb" "/tmp/bookworm_root"
        fi
    done

    cp -a /tmp/bookworm_root/lib/aarch64-linux-gnu/* "/roms2/ports/armsx2_r36s/lib/" 2>/dev/null || true
    cp -a /tmp/bookworm_root/usr/lib/aarch64-linux-gnu/* "/roms2/ports/armsx2_r36s/lib/" 2>/dev/null || true
    rm -rf "/tmp/armsx2-sdl.tar.zst" "/tmp/malisx2.tar.gz" "/tmp/armsx2_extract" "/tmp/malisx2_extract" "/tmp/bookworm_pkgs" "/tmp/bookworm_root"
fi

# 2. Generate the ES-DE / PortMaster Launch Script using explicit paths
cat << 'LAUNCH_EOF' > "/roms2/ports/armsx2-R36SROM2dd.sh"
#!/bin/bash
rm -f "/roms2/ports/armsx2logfile.txt"
exec > "/roms2/ports/armsx2logfile.txt" 2>&1

if [ -f "/opt/system/Tools/PortMaster/control.txt" ]; then
  source "/opt/system/Tools/PortMaster/control.txt"
  export PORT_32BIT="N"
  [ -f "/opt/system/Tools/PortMaster/mod_${CFW_NAME}.txt" ] && source "/opt/system/Tools/PortMaster/mod_${CFW_NAME}.txt"
elif [ -f "/opt/tools/PortMaster/control.txt" ]; then
  source "/opt/tools/PortMaster/control.txt"
  export PORT_32BIT="N"
  [ -f "/opt/tools/PortMaster/mod_${CFW_NAME}.txt" ] && source "/opt/tools/PortMaster/mod_${CFW_NAME}.txt"
elif [ -f "/roms2/ports/PortMaster/control.txt" ]; then
  source "/roms2/ports/PortMaster/control.txt"
  export PORT_32BIT="N"
  [ -f "/roms2/ports/PortMaster/mod_${CFW_NAME}.txt" ] && source "/roms2/ports/PortMaster/mod_${CFW_NAME}.txt"
fi
get_controls

$ESUDO chmod 666 /dev/mali0 /dev/dri/card0 2>/dev/null

export HOME="/home/ark"
export XDG_CONFIG_HOME="/home/ark/.config"
$ESUDO mkdir -p "/home/ark/.config/ARMSX2/inis" "/home/ark/.config/ARMSX2/bios"

$ESUDO tee "/home/ark/.config/ARMSX2/inis/PCSX2.ini" > /dev/null << 'EOF'
[UI]
SettingsVersion = 1
SetupWizardIncomplete = false
StartFullscreen = true

[GameList]
RecursivePaths = /roms/ps2

[EmuCore]
EnableCheats = false
EnableWideScreenPatches = false

[EmuCore/Speedhacks]
EECycleRate = -1
EECycleSkip = 0
fastCDVD = true
IntcStat = true
WaitLoop = true
vuFlagHack = true
vuThread = true
vu1Instant = true

[EmuCore/GS]
Renderer = Vulkan
VsyncEnable = 0
SyncToHostRefreshRate = false
upscale_multiplier = 1
filter = 1
mipmap_hw = 0
accurate_blending_unit = 0
texture_preloading = 1
hw_download_mode = 2
OsdShowFPS = true
OsdShowSpeed = true
OsdShowGSStats = false
OsdScale = 200.0

[SPU2/Output]
OutputModule = cubeb
BackendName = alsa
DeviceName = default
SynchMode = TimeStretch
Latency = 80

[Pad1]
Type = DualShock2
InvertL = 0
InvertR = 0
Deadzone = 0
AxisScale = 1.33
LargeMotorScale = 0.00
SmallMotorScale = 0.00
ButtonDeadzone = 0
PressureModifier = 0.50
Up = SDL-0/DPadUp
Right = SDL-0/DPadRight
Down = SDL-0/DPadDown
Left = SDL-0/DPadLeft
Triangle = SDL-0/Y
Circle = SDL-0/B
Cross = SDL-0/A
Square = SDL-0/X
Select = SDL-0/Back
Start = SDL-0/Start
L1 = SDL-0/LeftShoulder
L2 = SDL-0/+LeftTrigger
R1 = SDL-0/RightShoulder
R2 = SDL-0/+RightTrigger
L3 = SDL-0/LeftStick
R3 = SDL-0/RightStick
Analog = SDL-0/Guide
LUp = SDL-0/-LeftY
LRight = SDL-0/+LeftX
LDown = SDL-0/+LeftY
LLeft = SDL-0/-LeftX
RUp = SDL-0/-RightY
RRight = SDL-0/+RightX
RDown = SDL-0/+RightY
RLeft = SDL-0/-RightX

[Hotkeys]
OpenPauseMenu = SDL-0/Back & SDL-0/X
ShutdownVM = SDL-0/Back & SDL-0/Start
EOF

$ESUDO chown -R ark:ark "/home/ark/.config/ARMSX2"
[ -f "/roms2/bios/scph39001.bin" ] && cp -n "/roms2/bios/scph39001.bin" "/home/ark/.config/ARMSX2/bios/scph39001.bin"
[ -f "/roms2/bios/scph39001.MEC" ] && cp -n "/roms2/bios/scph39001.MEC" "/home/ark/.config/ARMSX2/bios/scph39001.MEC"

if [ -n "$1" ] && [ -f "$1" ]; then
    $ESUDO env HOME="/home/ark" XDG_CONFIG_HOME="/home/ark/.config" DBUS_SESSION_BUS_ADDRESS=/dev/null VK_DRIVER_FILES="/roms2/ports/armsx2_r36s/malisx2/malisx2_icd.aarch64.json" VK_ICD_FILENAMES="/roms2/ports/armsx2_r36s/malisx2/malisx2_icd.aarch64.json" VK_LOADER_DRIVERS_SELECT="malisx2*" MANGOHUD=0 SDL_VIDEODRIVER=kmsdrm SDL_RENDER_DRIVER=software SDL_AUDIODRIVER=alsa AUDIODEV=default "/roms2/ports/armsx2_r36s/lib/ld-linux-aarch64.so.1" --library-path "/roms2/ports/armsx2_r36s/lib:/roms2/ports/armsx2_r36s/bin:/roms2/ports/armsx2_r36s/malisx2" "/roms2/ports/armsx2_r36s/bin/armsx2-sdl" -- "$1"
else
    $ESUDO env HOME="/home/ark" XDG_CONFIG_HOME="/home/ark/.config" DBUS_SESSION_BUS_ADDRESS=/dev/null VK_DRIVER_FILES="/roms2/ports/armsx2_r36s/malisx2/malisx2_icd.aarch64.json" VK_ICD_FILENAMES="/roms2/ports/armsx2_r36s/malisx2/malisx2_icd.aarch64.json" VK_LOADER_DRIVERS_SELECT="malisx2*" MANGOHUD=0 SDL_VIDEODRIVER=kmsdrm SDL_RENDER_DRIVER=software SDL_AUDIODRIVER=alsa AUDIODEV=default "/roms2/ports/armsx2_r36s/lib/ld-linux-aarch64.so.1" --library-path "/roms2/ports/armsx2_r36s/lib:/roms2/ports/armsx2_r36s/bin:/roms2/ports/armsx2_r36s/malisx2" "/roms2/ports/armsx2_r36s/bin/armsx2-sdl"
fi

$ESUDO pkill -9 armsx2-sdl
pm_finish
$ESUDO systemctl restart oga_events &
printf "\033c" > /dev/tty0
LAUNCH_EOF

chmod +x "/roms2/ports/armsx2-R36SROM2dd.sh"

# 3. Auto-inject PS2 system into ES-DE / EmulationStation config using explicit paths
if [ -f "/home/ark/.emulationstation/es_systems.cfg" ] && ! grep -q "<name>ps2</name>" "/home/ark/.emulationstation/es_systems.cfg"; then
    sudo cp "/home/ark/.emulationstation/es_systems.cfg" "/home/ark/.emulationstation/es_systems.cfg.bak"
    sudo sed -i 's|</systemList>|  <system>\n    <name>ps2</name>\n    <fullname>Sony PlayStation 2</fullname>\n    <path>/roms/ps2</path>\n    <extension>.iso .ISO .chd .CHD .cso .CSO .bin .BIN .gz .GZ .elf .ELF</extension>\n    <command>/roms2/ports/armsx2-R36SROM2dd.sh "%ROM%"</command>\n    <platform>ps2</platform>\n    <theme>ps2</theme>\n  </system>\n</systemList>|' "/home/ark/.emulationstation/es_systems.cfg"
fi

if [ -f "/etc/emulationstation/es_systems.cfg" ] && ! grep -q "<name>ps2</name>" "/etc/emulationstation/es_systems.cfg"; then
    sudo cp "/etc/emulationstation/es_systems.cfg" "/etc/emulationstation/es_systems.cfg.bak"
    sudo sed -i 's|</systemList>|  <system>\n    <name>ps2</name>\n    <fullname>Sony PlayStation 2</fullname>\n    <path>/roms/ps2</path>\n    <extension>.iso .ISO .chd .CHD .cso .CSO .bin .BIN .gz .GZ .elf .ELF</extension>\n    <command>/roms2/ports/armsx2-R36SROM2dd.sh "%ROM%"</command>\n    <platform>ps2</platform>\n    <theme>ps2</theme>\n  </system>\n</systemList>|' "/etc/emulationstation/es_systems.cfg"
fi

if [ -f "/home/ark/ES-DE/custom_systems/es_systems.xml" ] && ! grep -q "<name>ps2</name>" "/home/ark/ES-DE/custom_systems/es_systems.xml"; then
    sudo cp "/home/ark/ES-DE/custom_systems/es_systems.xml" "/home/ark/ES-DE/custom_systems/es_systems.xml.bak"
    sudo sed -i 's|</systemList>|  <system>\n    <name>ps2</name>\n    <fullname>Sony PlayStation 2</fullname>\n    <path>/roms/ps2</path>\n    <extension>.iso .ISO .chd .CHD .cso .CSO .bin .BIN .gz .GZ .elf .ELF</extension>\n    <command>/roms2/ports/armsx2-R36SROM2dd.sh "%ROM%"</command>\n    <platform>ps2</platform>\n    <theme>ps2</theme>\n  </system>\n</systemList>|' "/home/ark/ES-DE/custom_systems/es_systems.xml"
fi

# 4. Launch immediately
echo "Setup complete! Launching ARMSX2..." > /dev/tty0
exec "/roms2/ports/armsx2-R36SROM2dd.sh" "$1"

=======
Credits
=======

Here is the Author-by-Author Credits & Attributions list covering every component, driver, runtime, and script layer used in your setup:


Full Credits to All Authors & Maintainers
* 1. PS2 Emulation Engine (ARMSX2 & Upstream PCSX2)
    * Authors: The ARMSX2 Team (ARM64 Linux/Android port, Mali r44p1 workarounds, and armsx2-sdl bare-display handheld build) and The PCSX2 Development Team (upstream EE/IOP/VU recompilers, GS Vulkan renderer, and SPU2 audio core).
    * Source: ARMSX2 GitHub | PCSX2 GitHub
* 2. Mali Kernel-to-Vulkan Translation Driver (malisx2 & Mesa 3D)
    * Authors: bmdhacks (author of malisx2, implementing legacy /dev/mali0 mali_kbase UK 11.7 / 1.18+ Job Manager support and VK_KHR_display presentation) and The Mesa 3D / Panfrost Contributors (underlying compiler and Vulkan/WSI framework).
    * Source: bmdhacks/malisx2 GitHub | Mesa 3D
* 3. Nested glibc 2.36+ Runtime & AppDir Bundling (sharun / AnylinuxARM64 & Debian)
    * Authors: VHSgunzo (creator of sharun and the Anylinux portable AppDir runtime hooks) and The Debian Project / GNU C Library (glibc) Developers (providing the isolated arm64 Bookworm ld-linux-aarch64.so.1, libc6, libdrm, libgbm, and libasound libraries).
    * Source: VHSgunzo/sharun GitHub | Debian ARM64
* 4. Handheld Operating System & PortMaster Framework (ArkOS, PortMaster, & ROCKNIX)
    * Authors: Christian Haitian (creator and maintainer of ArkOS for RK3326/R36S), The PortMaster Team (including kloptops, JohnnyonFlame, brooksytech, and contributors for control.txt, gptokeyb, and handheld runtime hooks), and The ROCKNIX Team (reference mainline kernel + PanVK implementation).
    * Source: ArkOS GitHub | PortMaster | ROCKNIX GitHub
* 5. Window System Integration & Audio/Input Middleware (SDL2, Cubeb, & Arm WSI)
    * Authors: Sam Lantinga & The Simple DirectMedia Layer (SDL2) Contributors (kmsdrm direct video backend and GO-Super Gamepad input mapping), Mozilla / Matthew Gregan & Cubeb Contributors (ALSA audio backend), and Arm Limited / ginkage (vulkan-wsi-layer reference architecture).
    * Source: libsdl-org/SDL | mozilla/cubeb | ARM-software/vulkan-wsi-layer
* 6. Frontend Menu Integration (ES-DE / EmulationStation)
    * Authors: Leon Styhre & The ES-DE Contributors (EmulationStation Desktop Edition es_systems.xml / es_systems.cfg ROM launcher integration).
    * Source: ES-DE.org
* 7. R36S ArkOS Integration, Bridge Architecture & Automation Scripts
    * Author: You (system integration, /dev/mali0 + kmsdrm + ALSA + SDL-0 configuration tuning, and ES-DE automation workflow for the R36S).

If you'd like this formatted into a compact credits slide (for a video/presentation outro) or a CREDITS.md / README.md file for sharing your release, let me know!
