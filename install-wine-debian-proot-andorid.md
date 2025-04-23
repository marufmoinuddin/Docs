Running 32-bit and 64-bit Windows Programs on Android with Termux, Box86, Box64, and Wine 7.20
Date: April 23, 2025
This guide explains how to run 32-bit and 64-bit Windows programs (.exe files) on an Android device using Termux, Box86, Box64, and Wine 7.20 (x86 and amd64) in a proot environment. No root access is required. The setup ensures wineserver and other components work correctly on ARM-based Android devices.
Overview

Termux: A terminal emulator for running Linux commands on Android.
Proot: Runs a Debian Linux environment inside Termux without root.
Box86: Translates 32-bit (x86) instructions to ARM instructions.
Box64: Translates 64-bit (x64) instructions to ARM instructions.
Wine 7.20 (x86 and amd64): A compatibility layer for running 32-bit and 64-bit Windows programs on Linux.
Winetricks: A tool to install Windows libraries for better compatibility.

Prerequisites

Android Device: ARM64 architecture (most modern phones), 4GB+ free storage.
Internet Connection: Stable, for downloading packages.
Termux App: Install from F-Droid or GitHub (avoid Play Store).
Basic Terminal Knowledge: Familiarity with typing commands.

Setup Instructions
Step 1: Install Termux and Proot

Install Termux:

Download and install Termux from F-Droid or GitHub.
Open Termux and update packages:pkg update && pkg upgrade
pkg install wget




Install Proot and Debian:

Install proot-distro to set up a Debian environment:pkg install proot-distro
proot-distro install debian


Log into the Debian environment:proot-distro login debian


If prompted, create a user (e.g., user) and password.


Set Up Termux-X11:

Install Termux-X11 to display graphical windows:pkg install termux-x11


Start Termux-X11 in a separate Termux session:termux-x11 :0


In your main session, set the display:export DISPLAY=:0





Step 2: Install Box86 and Box64
Box86 and Box64 allow 32-bit and 64-bit Windows programs to run on ARM devices.

Enable 32-bit Support:

Add support for 32-bit (armhf) packages:sudo dpkg --add-architecture armhf
sudo apt update




Install Box86:

Add the Box86 repository:sudo apt install gpg
sudo wget https://ryanfortner.github.io/box86-debs/box86.list -O /etc/apt/sources.list.d/box86.list
wget -qO- https://ryanfortner.github.io/box86-debs/KEY.gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/box86-debs-archive-keyring.gpg
sudo apt update


Install Box86:sudo apt install box86-android




Install Box64:

Add the Box64 repository:sudo wget https://ryanfortner.github.io/box64-debs/box64.list -O /etc/apt/sources.list.d/box64.list
wget -qO- https://ryanfortner.github.io/box64-debs/KEY.gpg | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/box64-debs-archive-keyring.gpg
sudo apt update


Install Box64:sudo apt install box64-android





Step 3: Install Wine 7.20 (x86 and amd64)
Wine 7.20 (x86) runs 32-bit programs, and Wine 7.20 (amd64) runs 64-bit programs. We’ll use pre-built binaries for ARM compatibility.

Install Wine Dependencies:

Install required libraries for both 32-bit and 64-bit Wine:sudo apt install nano cabextract libfreetype6 libfreetype6:armhf libfontconfig1 libfontconfig1:armhf libxext6 libxext6:armhf libxinerama1 libxinerama1:armhf libxxf86vm1 libxxf86vm1:armhf libxrender1 libxrender1:armhf libxcomposite1 libxcomposite1:armhf libxrandr2 libxrandr2:armhf libxi6 libxi6:armhf libxcursor1 libxcursor1:armhf libvulkan1 libvulkan1:armhf zenity




Download Wine 7.20 (x86):

Download the 32-bit tarball:cd ~
wget https://github.com/Kron4ek/Wine-Builds/releases/download/7.20/wine-7.20-x86.tar.xz




Download Wine 7.20 (amd64):

Download the 64-bit tarball:wget https://github.com/Kron4ek/Wine-Builds/releases/download/7.20/wine-7.20-amd64.tar.xz




Extract and Organize:

Unzip and rename for clarity:tar xvf wine-7.20-x86.tar.xz
mv wine-7.20-x86 wine
tar xvf wine-7.20-amd64.tar.xz
mv wine-7.20-amd64 wine64
rm wine-7.20-x86.tar.xz wine-7.20-amd64.tar.xz




Set Up Wine Scripts:

Create a script for 32-bit Wine with Box86:echo '#!/bin/bash
export WINEPREFIX=~/.wine32
export WINESERVER=~/wine/bin/wineserver
box86 ~/wine/bin/wine "$@"' | sudo tee /usr/local/bin/wine
sudo chmod +x /usr/local/bin/wine


Create a script for 64-bit Wine with Box64:echo '#!/bin/bash
export WINEPREFIX=~/.wine64
export WINESERVER=~/wine64/bin/wineserver
box64 ~/wine64/bin/wine64 "$@"' | sudo tee /usr/local/bin/wine64
sudo chmod +x /usr/local/bin/wine64




Configure Environment Variables:

Add Box86 and Box64 paths to ~/.bashrc:echo 'export DISPLAY=:0
export BOX86_PATH=~/wine/bin/
export BOX86_LD_LIBRARY_PATH=~/wine/lib/wine/i386-unix/:/lib/i386-linux-gnu/:/lib/arm-linux-gnueabihf/:/usr/lib/i386-linux-gnu/:/usr/lib/arm-linux-gnueabihf/
export BOX64_PATH=~/wine64/bin/
export BOX64_LD_LIBRARY_PATH=~/wine64/lib/wine/i386-unix/:~/wine64/lib/wine/x86_64-unix/:/lib/i386-linux-gnu/:/lib/x86_64-linux-gnu/:/lib/arm-linux-gnueabihf/:/usr/lib/i386-linux-gnu/:/usr/lib/x86_64-linux-gnu/' >> ~/.bashrc
source ~/.bashrc





Step 4: Install Winetricks
Winetricks simplifies installing Windows libraries for both 32-bit and 64-bit programs.

Download Winetricks:
cd ~
wget https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
chmod +x winetricks
sudo mv winetricks /usr/local/bin/


Create Winetricks Scripts:

For 32-bit Wine:echo '#!/bin/bash
export BOX86_NOBANNER=1 WINE=wine WINEPREFIX=~/.wine32 WINESERVER=~/wine/bin/wineserver
wine /usr/local/bin/winetricks "$@"' | sudo tee /usr/local/bin/winetricks32
sudo chmod +x /usr/local/bin/winetricks32


For 64-bit Wine:echo '#!/bin/bash
export BOX64_NOBANNER=1 WINE=wine64 WINEPREFIX=~/.wine64 WINESERVER=~/wine64/bin/wineserver
wine64 /usr/local/bin/winetricks "$@"' | sudo tee /usr/local/bin/winetricks64
sudo chmod +x /usr/local/bin/winetricks64





Step 5: Test the Setup

Initialize Wine Prefixes:

For 32-bit Wine:wine wineboot

This creates ~/.wine32. You may see prompts for “Wine Mono” or “Wine Gecko”; cancel or install as needed.
For 64-bit Wine:wine64 wineboot

This creates ~/.wine64.


Test Wineserver:

For 32-bit:box86 ~/wine/bin/wineserver --version

Expected output: wineserver-7.20.
For 64-bit:box64 ~/wine64/bin/wineserver --version

Expected output: wineserver-7.20.


Run Test Programs:

32-bit Test (Notepad++):cd ~
wget https://github.com/notepad-plus-plus/notepad-plus-plus/releases/download/v8.6.9/npp.8.6.9.portable.x86.zip
unzip npp.8.6.9.portable.x86.zip
wine notepad++.exe

If Notepad++ opens, the 32-bit setup is successful.
64-bit Test (Notepad++):wget https://github.com/notepad-plus-plus/notepad-plus-plus/releases/download/v8.6.9/npp.8.6.9.portable.x64.zip
unzip npp.8.6.9.portable.x64.zip
wine64 notepad++.exe

If Notepad++ opens, the 64-bit setup is successful.



Step 6: Running Other Programs

Check Compatibility:

Visit the Wine Application Database to check if your program is supported.
Install libraries with winetricks:winetricks32 vcrun2019  # For 32-bit
winetricks64 dotnet48   # For 64-bit




Determine Program Architecture:

Check if your .exe is 32-bit or 64-bit (check documentation or download page).


Run a Program:

Place your .exe in a folder (e.g., ~/programs/):cd ~/programs


For 32-bit:wine your_program.exe


For 64-bit:wine64 your_program.exe




Use Custom Prefix (Optional):

To isolate a program’s settings:export WINEPREFIX=~/.custom_prefix
export WINEARCH=win32  # For 32-bit
wine wineboot
wine your_program.exe

Or:export WINEPREFIX=~/.custom_prefix
export WINEARCH=win64  # For 64-bit
wine64 wineboot
wine64 your_program.exe





Troubleshooting

Error: wine: could not exec wineserver:

Run with debug logging:BOX86_LOG=1 wine wineboot    # For 32-bit
BOX64_LOG=1 wine64 wineboot  # For 64-bit


Check for missing libraries:sudo apt install -f


Verify wineserver path:ls ~/wine/bin/wineserver    # For 32-bit
ls ~/wine64/bin/wineserver  # For 64-bit




No Window Appears:

Ensure Termux-X11 is running:termux-x11 :0
export DISPLAY=:0


Confirm the program’s architecture (32-bit or 64-bit).


Program Crashes:

Install required libraries with winetricks32 or winetricks64.
Check compatibility on the Wine Application Database.
Reinitialize the prefix:rm -rf ~/.wine32
wine wineboot

Or:rm -rf ~/.wine64
wine64 wineboot




Performance Issues:

Ensure sufficient RAM and CPU power.
For games, consider installing DXVK (requires additional setup; see DXVK Releases).



Notes

Wine 7.20: This guide uses Wine 7.20 (x86 and amd64). For better compatibility, consider newer versions from Kron4ek’s Wine-Builds.
Separate Prefixes: Use ~/.wine32 for 32-bit and ~/.wine64 for 64-bit to avoid conflicts.
Backup: Back up important files before running programs to avoid data loss.
Limitations: Some programs may not work due to Wine’s compatibility or ARM translation limitations.

References

Wine User’s Guide
Box86 Usage
Box64 Usage
Kron4ek’s Wine-Builds
Winetricks

