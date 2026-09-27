# RezTank

A community rebuild of the Virindi Tank plugin for Asheron's Call (Decal). It is a
drop-in replacement for `utank2-i.dll`: same file name, same plugin identity, same
dependencies, so Decal, VTClassic, UtilityBelt and VirindiHUDs keep working unchanged.

Current test build: **reztank.test.v2** (pre-release). Next builds use the `RezTank.Test.v2.01` format. No public release yet.

Build names: test builds are `RezTank.Test.v2.01`, `v2.02`, … (pre-releases, update channel
`latest-test.txt`); release builds are `RezTank.v1`, `v1.01`, … (update channel `latest.txt`). Test builds only ever update to
newer test builds, release builds only to newer releases.

## Install

1. Close Asheron's Call.
2. Open your Virindi Tank folder (the one containing `utank2-i.dll`, usually
   `C:\Games\VirindiPlugins\VirindiTank\`).
3. Rename the existing `utank2-i.dll` to `utank2-i.dll.orig` (your backup).
4. Download the newest `RezTank.*.dll` from the [releases page](https://github.com/ResistanceLab/RezTank/releases)
   into that folder and rename it to `utank2-i.dll` (the file name Decal is registered to load).
   From then on RezTank installs updates under their own versioned names (`RezTank.Test.v2.05.dll`, ...)
   and re-points Decal at them, as long as the game runs with administrator rights; without those
   rights it simply replaces the file in place.
5. Start the game and make sure Virindi Tank is enabled in the Decal window.

The VTank window title will read `Virindi Tank v.1.0.0.0 [reztank.test.v1]`, and after login chat
shows `Virindi Tank reztank.test.v1 (community rebuild) loaded. Profiles: <folder>`.

## Updates

RezTank checks this repository at login. When a newer build is published it downloads it,
verifies it, and installs it for the next game start (the previous build is kept next to it
as `<old name>.prev`; delete that once you are happy with the new build). Right-click the
installed DLL -> Properties -> Details to see which build it is. To receive only the chat notice and install by hand, create an empty
file named `noautoupdate.txt` in the Virindi Tank folder.

Virindi Automatic Updates Filter will not overwrite RezTank with stock Virindi Tank.

## Changes vs. stock Virindi Tank

### reztank.test.v1
- Startup no longer crashes when Decal's `ProfilePath` registry value is missing. Profiles
  are read from `ProfilePath` if it is set and the folder exists, otherwise from the folder
  the DLL is in.
- Built-in update channel (announce + self-install) pointing at this repository.
- Plugin exceptions are still logged locally to `Documents\Decal Plugins\uTank2\errors.txt`
  but are no longer uploaded to virindi.net.

## Restore stock

Delete `utank2-i.dll` and rename `utank2-i.dll.orig` back to `utank2-i.dll`. If stock VTank
then fails to start with a NullReferenceException in `PluginCore.Startup`, add a String
value `ProfilePath` = `<your VirindiTank folder>\` under
`HKLM\SOFTWARE\WOW6432Node\Decal\Plugins\{642F1F48-16BE-48BF-B1D4-286652C4533E}`.

## Releasing a new build (maintainer notes)

### Scripted (recommended)

One-time setup in Git Bash:

    winget install GitHub.cli          # or download from https://cli.github.com
    gh auth login                      # browser login, pick HTTPS
    git clone https://github.com/ResistanceLab/RezTank.git
    cd RezTank

Each release:

    ./release.sh test 2.03 "/c/path/to/RezTank.Test.v2.03.dll" "What changed"   # pre-release RezTank.Test.v2.03
    ./release.sh release 1 "/c/path/to/RezTank.v1.dll" "First release"         # release RezTank.v1

The script computes the checksum, creates the release with the DLL attached, rewrites the
channel manifest and pushes it. Players pick it up at their next login.

### Manual (what the script does)

1. Build the new DLL with the build id bumped (`RezTank.Test.v2.01` for a pre-release,
   `RezTank.v1` for a release).
2. Create a GitHub release tagged exactly with the build id, mark test builds as pre-release,
   and attach the DLL named after the build (`RezTank.Test.v2.03.dll`) as an asset.
3. Edit the channel manifest on `main` — `latest-test.txt` for test builds, `latest.txt` for
   releases:
   - line 1: the new build id
   - line 2: `https://github.com/ResistanceLab/RezTank/releases/download/<build id>/<build id>.dll`
     (tag-specific link; the `releases/latest` permalink skips pre-releases, so don't use it)
   - line 3: SHA-256 of the DLL (optional but recommended)
   - remaining lines: short notes shown in chat
4. Players on older builds see the notice at their next login and the DLL installs itself.

Note: the repository must be **public** for the update check and downloads to work from
inside the game (the plugin fetches without any credentials).
