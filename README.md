# Windows 11 Pro — clean, quiet setup for programming & gaming

**Researched:** 27 September 2026 · **Target:** Windows 11 Pro **25H2** (build 26200.x), local account, no Microsoft 365 / Copilot subscription
**Goal:** minimum bloatware, telemetry, ads and AI features — while **Windows Update, Microsoft Defender and the Microsoft Store keep working**.
**Approach:** native only — Settings, Group Policy (`gpedit.msc`), policy registry keys and built-in PowerShell. No third-party debloat tools.

**Legend:** ✅ Do · ⚙️ Optional · ⛔ Don't · ⚠ could not be fully verified against Microsoft documentation — check it took effect (the audit in §11 does this for you).

---

## Why this guide looks the way it does (read once)

1. **Pro ignores many "privacy" policies.** Lots of widely-copied tweaks only work on Enterprise/Education and are *silently ignored* on Pro — e.g. `AllowTelemetry = 0`, "Turn off Microsoft consumer experiences", almost all Windows Spotlight policies, `DoNotUseWebResults`, "Remove default Microsoft Store packages". This guide only uses policies that work on Pro; everything else is done with Settings toggles.
2. **Diagnostic data can't go below "Required" on Pro.** That's the floor; Windows Update depends on it. Everything *optional* can be switched off.
3. **Keep the security stack on.** Modern anti-cheat (Riot Vanguard, FACEIT, EA Javelin/Battlefield 6, Call of Duty) requires TPM 2.0 + Secure Boot, and increasingly VBS / Memory Integrity (HVCI) and IOMMU. Disabling Defender or VBS "for FPS" will lock you out of games and gains little.
4. **Don't block Microsoft servers.** Hosts-file / firewall blocklists break Update and Store (they share CDNs) and Defender flags telemetry hosts-file edits.
5. **Install 25H2 now.** 24H2 Home/Pro stops getting updates on **13 Oct 2026**. 26H2 (build 26300.x, in Release Preview since 27 Aug 2026, GA expected October) arrives later as a small enablement update — no reason to wait for it. 26H1 exists only on new Arm devices.

---

## 0. Before you start

| | Step |
|---|---|
| ✅ | **Back up** everything you care about. A clean install wipes the system disk. Also note your Wi-Fi password. |
| ✅ | **Firmware (BIOS/UEFI) settings:** enable **TPM 2.0** (Intel PTT / AMD fTPM), **Secure Boot** (UEFI mode, no CSM), **virtualization** (Intel VT-x / AMD SVM) and **IOMMU** (Intel VT-d / AMD-Vi). Enable **XMP/EXPO** for your RAM and **Resizable BAR** (Above 4G decoding) for the GPU. Update the BIOS first if it's old. |
| ✅ | **Download drivers to a USB stick:** at least the **network (LAN/Wi-Fi) driver** for your motherboard/laptop, because setup runs **offline**. Also grab the chipset driver and your GPU driver (NVIDIA / AMD / Intel) to save time. |
| ✅ | **Get official media:** the **Media Creation Tool** or the **25H2 ISO** from `https://www.microsoft.com/software-download/windows11`. Don't use pre-modified ISOs (Tiny11, "lite" builds, MicroWin) — see §12. |
| ⚙️ | If your PC is a laptop that shipped with **Windows 11 Home**, its firmware contains a Home key and setup will silently install Home. The answer file in §1 forces **Pro** via the generic Pro install key. (Activation still needs a real Pro licence / digital licence.) |

---

## 1. Build the answer file (`autounattend.xml`)

**Why:** Microsoft has closed the old local-account tricks. As of Sept 2026:

| Method | Status |
|---|---|
| `oobe\bypassnro` | ⛔ removed from 26200-series builds (2025) |
| `start ms-cxh:localonly` | ⛔ blocked in current builds |
| Registry `BypassNRO` / `HideOnlineAccountScreens` / dev-console tricks | ⚠ unreliable, conflicting reports |
| **`autounattend.xml` with the local account pre-defined in the file** | ✅ **works on retail 25H2 Pro** — it's Microsoft's own deployment mechanism, so it's the most durable option |
| Pro: *Set up for work or school → Sign-in options → Domain join instead* | ✅ works — manual fallback |

Use the free generator at **https://schneegans.de/windows/unattend-generator/** (supports 24H2/25H2/26H2). It changes *settings during setup*; it doesn't modify the Windows image, so Windows Update keeps working normally.

> Labels on the generator page may differ slightly from the wording below — match by meaning.

### ✅ Set / tick these

| Generator section | Setting |
|---|---|
| Region and language | **Windows display language, keyboard/input:** yours. **Home location / GeoLocation: `Ireland`** (see box below). |
| Time zone | Your real time zone (explicit, not "from region"). |
| Setup settings | **Windows edition: Pro — use generic product key.** Leave the "bypass Windows 11 requirements" options **off** (you want TPM/Secure Boot for anti-cheat). |
| Partitioning | **Interactive** — you choose and wipe the disk yourself during setup. (Safer than letting a file auto-wipe "disk 0".) |
| Computer name | Your choice. |
| User accounts | **Add a local account**, member of **Administrators**, with a password. Tick **"log on to the first admin account"** (auto-logon once). Choose **password never expires**. |
| Wi-Fi | **Skip Wi-Fi setup** (install offline). |
| Express settings | **Disable all** (turns off optional diagnostics, location, tailored experiences, ad ID prompts during OOBE). |
| System tweaks | Tick exactly the ✅ items in the **system tweaks list** below (12 items). Leave the ⛔ items unticked. The ⚙️ items are your call. |
| File Explorer | ⚙️ Show file extensions, ⚙️ show hidden files. |
| Start menu & taskbar | ✅ **Do not show Bing results when searching in the Start menu or the search box** (sets the Pro-valid policy `DisableSearchBoxSuggestions`, see §6). ✅ **Disable widgets**. ⚙️ Hide Task View button, left-align the taskbar. |
| Remove bloatware | Tick exactly the ✅ items in the **generator app list** below (38 items). Leave the ⛔ items unticked. The ⚙️ items are your call. |

### ⛔ Do NOT tick these

| Option | Why |
|---|---|
| Remove **Microsoft Store** | Hard to reinstall; the Store also services Notepad, Paint, Terminal, codecs, winget (App Installer). |
| Disable **Windows Defender** | Breaks anti-cheat and removes your AV. |
| Disable **Windows Update** / pause updates | You lose security fixes. |
| Disable **SmartScreen**, **UAC**, **Smart App Control** | Security; Smart App Control can't be turned back on without a reinstall. |
| **"Xbox Apps"** (removes Xbox Identity Provider / Xbox TCUI too) | Some PC games/launchers use Xbox sign-in; those two are tiny and harmless. Remove the Xbox app + Game Bar with §4 instead. |
| Remove **Microsoft Edge WebView2** (if offered) | Many apps (dev tools, launchers, installers) need it. |
| "Bypass TPM / Secure Boot / CPU requirements" | Only needed on unsupported hardware — and it defeats the anti-cheat requirements. |

### System tweaks list, in the generator's order

✅ = tick · ⛔ = leave unticked · ⚙️ = your call (suggestion in the note). Totals: 12 ✅, 7 ⛔, 7 ⚙️.

1. ⛔ **Disable Windows Update** — creates a task that pauses updates every week, so you'd stop getting security fixes
2. ⛔ **Disable User Account Control (UAC)** — UAC is what keeps your admin account safe; programs get admin rights only after you click "Yes"
3. ⛔ **Disable Smart App Control** — can't be turned back on; it usually switches itself off on developer PCs, and you can still turn it off later in Windows Security
4. ⛔ **Disable SmartScreen in Windows and Edge** — security, which you want to keep
5. ✅ **Disable Fast Startup** — avoids stale drivers after shutdown and dual-boot or GPU driver update problems; saves only a second or two on an SSD
6. ⛔ **Disable System Protection / System Restore** — the guide uses restore points (§3)
7. ✅ **Enable long paths** — needed for programming (node_modules, deep Git repos)
8. ⚙️ **Enable Remote Desktop services (RDP)** — leave unticked unless you connect into this PC from another device
9. ⚙️ **Harden ACLs** — small gain on a single-user PC; some old installers or games expect to write to C:\. Suggest leaving it unticked
10. ⚙️ **Delete hidden junction points** — cosmetic; old software may rely on paths like My Documents. Suggest leaving it unticked
11. ✅ **Allow execution of PowerShell script files** — RemoteSigned; you need this for programming
12. ⚙️ **Do not update Last Access Time stamp** — Windows already turns this off on drives over 128 GB. Suggest leaving it unticked
13. ⛔ **Prevent Windows Update from rebooting your computer** — a workaround task; set active hours instead (§8)
14. ⚙️ **Turn off system sounds** — personal taste
15. ✅ **Disable app suggestions / Content Delivery Manager** — stops silent installs of "suggested" apps; Pro ignores the official policy for this. Side effect: greyed-out "Manage devices" under Mobile devices
16. ✅ **Prevent device encryption** — you turn on BitLocker yourself in §3 and save the recovery key
17. ✅ **Hide Edge First Run Experience** — harmless; saves some clicks
18. ✅ **Disable Edge Startup Boost and Background mode** — stops Edge running in the background
19. ⛔ **Make Edge uninstallable** — not needed with the Ireland region; the generator warns it causes Windows Update problems
20. ✅ **Disable the Enhance Pointer Precision mouse setting** — no mouse acceleration; consistent aim in shooters
21. ✅ **Delete empty C:\Windows.old folder** — harmless cleanup
22. ✅ **Disable automatic sign-on of last user after a restart** — you sign in yourself after update restarts; doesn't affect the one-time sign-in at the end of setup
23. ✅ **Disable Windows Platform Binary Table (WPBT) execution** — blocks firmware from injecting vendor programs (e.g. ASUS Armoury Crate) into Windows on every boot
24. ✅ **Prevent download and installation of applications associated with certain hardware devices** — stops vendor apps (LG monitors, Razer/Logitech mice) auto-installing; install what you want yourself
25. ⚙️ **Audit process creation events** — logs every program start; noise for most people. Suggest leaving it unticked
26. ⚙️ **Include command line in log events** — only with the previous option; would log passwords/API tokens from command lines. Leave it unticked

### Generator app list ("Remove bloatware"), in the generator's order

✅ = tick (remove) · ⛔ = don't tick (keep) · ⚙️ = your call. Totals: 38 ✅, 14 ⛔, 7 ⚙️.

1. ✅ **3D Viewer** — retired
2. ✅ **Bing Search** — web results in Start search
3. ⛔ **Calculator**
4. ⛔ **Camera** — tiny; handy for testing the webcam
5. ✅ **Clipchamp**
6. ⚙️ **Clock** — remove if you don't use timers/alarms
7. ✅ **Copilot**
8. ✅ **Cortana** — retired
9. ✅ **Dev Home** — retired in 2025
10. ⚙️ **Facial recognition (Windows Hello)** — keep only if you have an IR camera for face sign-in
11. ✅ **Family** — needs a Microsoft account
12. ✅ **Feedback Hub**
13. ✅ **Game Assist (beta)** — Edge overlay that runs on Game Bar, which you're removing
14. ✅ **Get Help**
15. ⛔ **Handwriting (all languages) (beta)** — small; leave it
16. ✅ **Internet Explorer** — already switched off in Windows 11
17. ✅ **Mail and Calendar** — retired
18. ✅ **Maps**
19. ✅ **Math Input Panel** — old pen tool
20. ⛔ **Media Features** — some games/apps need these media components
21. ⛔ **Microsoft Store** — required
22. ✅ **Mixed Reality** — retired
23. ✅ **Movies & TV** — retired
24. ✅ **News**
25. ⛔ **Notepad (modern)** — its AI features get turned off by policy (§7)
26. ✅ **Office 365** — the Microsoft 365 Copilot app
27. ✅ **OneDrive** — keep only if you want cloud sync
28. ✅ **OneNote**
29. ✅ **OneSync** — mail/contacts sync engine, unused once Mail and People are gone
30. ⛔ **OpenSSH Client** — needed for programming (Git over SSH)
31. ✅ **Outlook for Windows**
32. ⛔ **Paint** — its AI features get turned off by policy (§7)
33. ✅ **Paint 3D** — retired
34. ✅ **People** — retired
35. ⛔ **Photos**
36. ✅ **Power Automate**
37. ✅ **PowerShell 2.0** — obsolete; attackers use it to get around security logging
38. ⚙️ **PowerShell ISE** — remove if you'll use VS Code
39. ✅ **Quick Assist** — keep only if someone helps you remotely
40. ✅ **Recall**
41. ⛔ **Remote Desktop Client**
42. ✅ **Skype** — retired
43. ⛔ **Snipping Tool** — needed for the Win+Shift+S screenshot shortcut
44. ✅ **Solitaire Collection**
45. ⛔ **Speech (all languages) (beta)** — Narrator and text-to-speech need it
46. ✅ **Steps Recorder** — deprecated
47. ⚙️ **Sticky Notes**
48. ✅ **Teams**
49. ✅ **Tips**
50. ✅ **To Do**
51. ⚙️ **Voice Recorder**
52. ✅ **Wallet**
53. ✅ **Weather**
54. ⚙️ **Windows Media Player (classic)** — suggest removing it
55. ⚙️ **Windows Media Player (modern)** — suggest keeping it, unless you install VLC or mpv
56. ⛔ **Windows Terminal** — needed for programming
57. ✅ **WordPad** — retired
58. ⛔ **Xbox Apps** — would also remove the Xbox sign-in parts some games use; §4 removes only the Xbox app and Game Bar
59. ✅ **Your Phone / Phone Link** — keep only if you link your phone

### Put the answer file on the USB stick (choose one method)

**Method A — standard USB stick (Media Creation Tool, or Rufus in plain mode):** copy **`autounattend.xml` to the root of the USB stick**, next to `setup.exe`. Windows Setup finds it there automatically.

**Method B — Ventoy stick:** don't touch the ISO. Ventoy's **auto-install plugin** hands the answer file to Windows Setup when it boots the ISO.

1. On the Ventoy data partition (the big one with your ISOs):
   ```text
   /ISO/Win11_25H2_x64.iso            <- your ISO (any folder / name)
   /ventoy/script/autounattend.xml    <- the generated answer file
   /ventoy/ventoy.json                <- plugin config (create it)
   ```
2. Create `/ventoy/ventoy.json` (paths start at the stick's root, use forward slashes, and must match your real file names exactly):
   ```json
   {
     "auto_install": [
       {
         "image": "/ISO/Win11_25H2_x64.iso",
         "template": "/ventoy/script/autounattend.xml"
       }
     ]
   }
   ```
   If you already have a `ventoy.json`, add the `"auto_install"` block to it instead of creating a second file. No JSON editing wanted? **VentoyPlugson.exe** (included in the Ventoy download) → *Auto Install Plugin*.
3. At boot, select the ISO. Ventoy shows a menu → pick **/ventoy/script/autounattend.xml** (not *Boot without auto installation template*). ⚙️ `"autosel": 1` in the block picks it automatically; keeping the menu lets you do a plain install from the same stick later.
4. **Secure Boot:** update Ventoy to the latest version first — since 1.1.13 you must enroll Ventoy's key once per PC (Microsoft's 2023 UEFI certificate change). On the blue **MOK management** screen: *Enroll key from disk* → **VTOYEFI** partition → **ENROLL_THIS_KEY_IN_MOKMANAGER.cer** → *Continue* → *Yes* → *Reboot*. Some PCs also need *Allow Microsoft 3rd Party UEFI CA* enabled in firmware. If Ventoy still won't boot with Secure Boot on, turn Secure Boot off just for the install and **turn it back on right after** (anti-cheat needs it).
5. With Ventoy the USB stick itself also shows up as a disk in Setup — another reason to keep partitioning **Interactive**. Delete partitions only on your system SSD.

The generator's scripts read the copy of the answer file that Setup saves to `C:\Windows\Panther\`, so they run the same with either method. How to confirm it worked: §3 step 1.

> **Why Ireland? (EEA region, EU Digital Markets Act)**
> The region you pick during setup is stored once as the *device region* (`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Control Panel\DeviceRegion`) and Windows compares it against `IntegratedServicesRegionPolicySet.json`. With an EEA device region you get: **Bing unlinkable from Windows Search**, **Edge uninstallable** from Settings, **Widgets news feed** can be switched off, almost every inbox app uninstallable, and exemption from the automatic **Microsoft 365 Copilot** app install. New AI features also tend to reach the EEA later.
> **Changing Settings → Region later does *not* change the device region** — only a reset/reinstall does. Your **Store/billing region**, formats, language, keyboard and time zone are separate and should be set to your real country after install (§3). Store, Update and Defender don't care about the device region.

---

## 2. Install

1. **Unplug Ethernet / don't connect Wi-Fi.** (The "interactive local account" path is ignored when online on 25H2 retail media; a pre-defined account works either way, but offline is the safe belt-and-braces.)
2. Boot from the USB (one-time boot menu, usually F8/F11/F12). **Method A:** Setup picks up `autounattend.xml` automatically. **Method B (Ventoy):** select the ISO, then choose the `/ventoy/script/autounattend.xml` template in Ventoy's menu.
3. At the disk screen, **delete all partitions on the target disk** → select the unallocated space → **Next**. Leave other disks alone.
4. Setup finishes and logs into your local account.
5. **Fallback** if something goes wrong and you land on the Microsoft-account screen (it means the answer file wasn't used — with Ventoy, check the paths in `ventoy.json`): *Set up for work or school → Sign-in options → Domain join instead* → create the local account.

---

## 3. First boot — do these in order

| # | | Step |
|---|---|---|
| 1 | ✅ | **Check the answer file ran:** you were never shown a Microsoft-account screen, and these logs exist: `C:\Windows\Setup\Scripts\Specialize.log`, `DefaultUser.log`, `FirstLogon.log`. If they're missing, the generator's tweaks and app removals didn't run — §4 and later still cover the essentials. |
| 2 | ✅ | **Restore point baseline:** open PowerShell **as admin**: `Enable-ComputerRestore -Drive "C:\"` then `Checkpoint-Computer -Description "Clean install" -RestorePointType MODIFY_SETTINGS` |
| 3 | ✅ | **Settings → Time & language → Language & region → Country or region = your real country**, Regional format = yours. (Device region stays EEA — that's intended.) |
| 4 | ✅ | Install the **network driver** from your USB if needed, then connect to the internet. |
| 5 | ✅ | **Chipset driver** (AMD/Intel) and **GPU driver** from the vendor (NVIDIA driver/NVIDIA app, AMD Adrenalin, Intel Arc). |
| 6 | ✅ | **Settings → Windows Update → Check for updates** — repeat, rebooting as needed, until nothing is left. Install *Optional updates → driver updates* only if you know you need one. |
| 7 | ✅ | **Microsoft Store → Library → Get updates** (updates App Installer/winget, Notepad, Paint, Terminal, Windows Security UI, codecs). |
| 8 | ✅ | **Windows Security** — check it's all green: *Virus & threat protection* on (real-time + cloud-delivered protection + tamper protection), *App & browser control → Reputation-based protection* on, *Device security → Core isolation → Memory integrity* **on** (Microsoft starts auto-enabling it on eligible PCs from Oct 2026 anyway; anti-cheat increasingly requires it). |
| 9 | ✅ | **BitLocker (Pro):** Control Panel → *BitLocker Drive Encryption* → *Turn on BitLocker* for C:. **Save the recovery key to a USB stick and/or print it** — store it off the PC. |
| 10 | ✅ | **Activation:** Settings → System → Activation should read *active with a digital license*. |

---

## 4. Remove the remaining inbox apps (native PowerShell)

The answer file removes most of these; this catches whatever it missed or whatever a later update re-adds. It removes **exact package names only** — no wildcards that could hit the Store, frameworks or Windows Security.

Open **Windows PowerShell (admin)** — use Windows PowerShell 5.1, not PowerShell 7, for the Appx cmdlets.

```powershell
# Apps to remove (exact package names). Missing ones are simply skipped.
$remove = @(
  'Clipchamp.Clipchamp',
  'Microsoft.BingNews', 'Microsoft.BingWeather', 'Microsoft.BingSearch',
  'Microsoft.MicrosoftOfficeHub',        # "Microsoft 365 Copilot" app
  'Microsoft.Copilot',                   # Copilot app
  'Microsoft.OutlookForWindows',         # new Outlook
  'MSTeams',
  'Microsoft.Todos',
  'Microsoft.PowerAutomateDesktop',
  'Microsoft.MicrosoftSolitaireCollection',
  'Microsoft.GetHelp',
  'Microsoft.Getstarted',                # Tips
  'Microsoft.WindowsFeedbackHub',
  'Microsoft.WindowsMaps',
  'Microsoft.Windows.DevHome',           # retired in 2025
  'Microsoft.GamingApp',                 # Xbox app (you use Steam/Epic)
  'Microsoft.XboxGamingOverlay'          # Game Bar
)

foreach ($name in $remove) {
  Get-AppxPackage -AllUsers -Name $name | Remove-AppxPackage -AllUsers -ErrorAction SilentlyContinue
  Get-AppxProvisionedPackage -Online | Where-Object DisplayName -eq $name |
    Remove-AppxProvisionedPackage -Online -ErrorAction SilentlyContinue | Out-Null
}
```

`Remove-AppxPackage` removes the app for existing users; `Remove-AppxProvisionedPackage` stops it being installed for new users. You need both.

**⚙️ Optional removals** — add to the list only if you don't use them: `MicrosoftCorporationII.QuickAssist` (Quick Assist), `Microsoft.YourPhone` + `MicrosoftWindows.CrossDevice` (Phone Link), `Microsoft.ZuneMusic` (Media Player), `Microsoft.WindowsSoundRecorder`, `Microsoft.WindowsAlarms`, `Microsoft.MicrosoftStickyNotes`.

**OneDrive** (if the answer file didn't already remove it):
```powershell
winget uninstall --id Microsoft.OneDrive
```

**⛔ Keep these** (removing them breaks Store, winget, Windows Security, apps or media):

| Keep | Why |
|---|---|
| `Microsoft.WindowsStore`, `Microsoft.StorePurchaseApp` | Store + purchases/licensing |
| `Microsoft.DesktopAppInstaller` | winget |
| `Microsoft.SecHealthUI` | Windows Security app |
| `Microsoft.VCLibs.*`, `Microsoft.UI.Xaml.*`, `Microsoft.NET.Native.*`, `Microsoft.WindowsAppRuntime.*` | Framework packages other apps depend on |
| `Microsoft.HEIFImageExtension`, `HEVCVideoExtension`, `VP9VideoExtensions`, `WebpImageExtension`, `AV1VideoExtension`, `RawImageExtension` | Codecs |
| `Microsoft.XboxIdentityProvider`, `Microsoft.Xbox.TCUI` | Xbox sign-in used by some PC games |
| `Microsoft.WindowsTerminal`, `Microsoft.WindowsNotepad`, `Microsoft.Paint`, `Microsoft.ScreenSketch`, `Microsoft.Windows.Photos`, `Microsoft.WindowsCalculator` | Useful tools (AI bits get disabled in §7) |
| Microsoft Edge **WebView2 Runtime** | Needed by many apps |

**⚙️ Edge (EEA only):** with the Ireland device region, *Settings → Apps → Installed apps → Microsoft Edge → Uninstall* is available. Install your browser first (e.g. `winget install --id Mozilla.Firefox` or `Google.Chrome`), then set it as default under *Settings → Apps → Default apps*. Keep WebView2. If you keep Edge, see §7 for its AI settings.

**Game Bar removed → "ms-gamingoverlay" pop-up fix.** Some games/keys trigger a "You'll need a new app to open this ms-gamingoverlay link" prompt after Game Bar is removed. Turn off game capture so nothing calls it:

```powershell
reg add "HKCU\System\GameConfigStore" /v GameDVR_Enabled /t REG_DWORD /d 0 /f
reg add "HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\GameDVR" /v AppCaptureEnabled /t REG_DWORD /d 0 /f
```
⚙️ Policy equivalent (machine-wide): *Computer Configuration → Administrative Templates → Windows Components → Windows Game Recording and Broadcasting → Enables or disables Windows Game Recording and Broadcasting = Disabled* (`HKLM\SOFTWARE\Policies\Microsoft\Windows\GameDVR` `AllowGameDVR = 0`). **Game Mode is unaffected.**

---

## How to apply policies: gpedit or `reg add`

Every policy below is given two ways — use **either**:

- **`gpedit.msc`** (Pro has it): navigate to the path, set the value. Visible and easy to undo.
- **`reg add`** in an admin PowerShell/Command Prompt: writes the same registry value the policy writes. Note: `gpedit.msc` will still show such settings as *Not configured* — that's cosmetic.

To undo a `reg add`, delete the value (`reg delete "<key>" /v <name> /f`) or set the policy back to *Not configured*. Most settings need a sign-out or reboot.

`HKLM\SOFTWARE\Policies\…` = *Computer Configuration*; `HKCU\Software\Policies\…` = *User Configuration*. "Admin Templates" below = *Administrative Templates*.

---

## 5. Telemetry & diagnostics

### Settings toggles ✅
- **Privacy & security → Diagnostics & feedback:** *Send optional diagnostic data* **off**; *Improve inking and typing* **off**; *View diagnostic data* **off**; (older builds only: *Tailored experiences* **off** — on current 25H2 it moved to *Recommendations & offers*, see §6); **Delete diagnostic data**; *Feedback frequency* **Never**.
- **Privacy & security → Activity history:** off, and **Clear history**.
- **Privacy & security → Speech:** *Online speech recognition* **off** (voice typing stops working).
- **Privacy & security → Inking & typing personalization:** off.
- **Privacy & security → Search** (called *Search permissions* on older builds): *Cloud content search* (Microsoft account and work/school) **off**; *Search history on this device* off (optional).
- ⚙️ **Privacy & security → Location:** off if you don't need it (breaks automatic time zone, weather location, Find My Device).
- **Privacy & security → Find my device:** off.
- **Privacy & security → App permissions:** review camera, microphone, etc.

### Pro-effective policies ✅

| Policy (gpedit, Computer Config unless noted) | Registry | Pro? |
|---|---|---|
| Admin Templates → Windows Components → Data Collection and Preview Builds → **Allow Diagnostic Data = Enabled, "Send required diagnostic data"** | `HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection` `AllowTelemetry = 1` | ✅ (0 = "off" is **ignored on Pro** and treated as 1) |
| same folder → **Configure diagnostic data opt-in settings user interface = Enabled, disable** | same key, `DisableTelemetryOptInSettingsUx = 1` | ✅ |
| same folder → **Disable diagnostic data viewer = Enabled** | same key, `DisableDiagnosticDataViewer = 1` | ✅ |
| same folder → **Do not show feedback notifications = Enabled** | same key, `DoNotShowFeedbackNotifications = 1` | ✅ |
| *User Config* → Admin Templates → Windows Components → Cloud Content → **Do not use diagnostic data for tailored experiences = Enabled** | `HKCU\Software\Policies\Microsoft\Windows\CloudContent` `DisableTailoredExperiencesWithDiagnosticData = 1` | ✅ |
| Admin Templates → System → OS Policies → **Enables Activity Feed = Disabled**, **Allow publishing of User Activities = Disabled**, **Allow upload of User Activities = Disabled** | `HKLM\SOFTWARE\Policies\Microsoft\Windows\System` `EnableActivityFeed = 0`, `PublishUserActivities = 0`, `UploadUserActivities = 0` | ✅ |
| Admin Templates → Control Panel → Regional and Language Options → **Allow users to enable online speech recognition services = Disabled** | `HKLM\SOFTWARE\Policies\Microsoft\InputPersonalization` `AllowInputPersonalization = 0` | ✅ |
| Admin Templates → Windows Components → Text Input → **Improve inking and typing recognition = Disabled** | `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\TextInput` `AllowLinguisticDataCollection = 0` | ✅ |
| Admin Templates → System → User Profiles → **Turn off the advertising ID = Enabled** | `HKLM\SOFTWARE\Policies\Microsoft\Windows\AdvertisingInfo` `DisabledByGroupPolicy = 1` | ✅ |
| Admin Templates → Windows Components → Search → **Allow Cloud Search = Disabled** | `HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search` `AllowCloudSearch = 0` | ✅ |
| Admin Templates → Windows Components → Find My Device → **Turn On/Off Find My Device = Disabled** | `HKLM\SOFTWARE\Policies\Microsoft\FindMyDevice` `AllowFindMyDevice = 0` | ✅ |
| ⚙️ Admin Templates → Windows Components → Location and Sensors → **Turn off location = Enabled** | `HKLM\SOFTWARE\Policies\Microsoft\Windows\LocationAndSensors` `DisableLocation = 1` | ✅ |
| ⚙️ Admin Templates → Windows Components → Windows Error Reporting → **Disable Windows Error Reporting = Enabled** | `HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Error Reporting` `Disabled = 1` | ✅ (no crash reports sent; Update/Defender unaffected) |

Copy-paste equivalent (admin), required items only:

```powershell
$dc = 'HKLM\SOFTWARE\Policies\Microsoft\Windows\DataCollection'
reg add $dc /v AllowTelemetry /t REG_DWORD /d 1 /f
reg add $dc /v DisableTelemetryOptInSettingsUx /t REG_DWORD /d 1 /f
reg add $dc /v DisableDiagnosticDataViewer /t REG_DWORD /d 1 /f
reg add $dc /v DoNotShowFeedbackNotifications /t REG_DWORD /d 1 /f
reg add "HKCU\Software\Policies\Microsoft\Windows\CloudContent" /v DisableTailoredExperiencesWithDiagnosticData /t REG_DWORD /d 1 /f
$sys = 'HKLM\SOFTWARE\Policies\Microsoft\Windows\System'
reg add $sys /v EnableActivityFeed /t REG_DWORD /d 0 /f
reg add $sys /v PublishUserActivities /t REG_DWORD /d 0 /f
reg add $sys /v UploadUserActivities /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\InputPersonalization" /v AllowInputPersonalization /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\TextInput" /v AllowLinguisticDataCollection /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\AdvertisingInfo" /v DisabledByGroupPolicy /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v AllowCloudSearch /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\FindMyDevice" /v AllowFindMyDevice /t REG_DWORD /d 0 /f
```

⚙️⚠ **DiagTrack service** (*Connected User Experiences and Telemetry*): some people set it to Disabled (`sc.exe config DiagTrack start= disabled`). Not documented by Microsoft, low value once diagnostic data is already at *Required*, and feature updates may re-enable it. Skip unless you want it; don't touch any other service.

---

## 6. Ads, "suggestions" and promotions

### Settings toggles ✅

| Where | What |
|---|---|
| **Personalization → Start** | Turn off: *Show recommendations for tips, shortcuts, new apps, and more*; *Show account-related notifications*; *Show mobile device in Start* (Phone Link pane); *Show websites from your browsing history*. Turn off *recently added / recently opened* items if you want an empty "Recent" area. (Since the 27 Aug 2026 preview / 8 Sep 2026 update, rolling out gradually, *Recommended* is renamed *Recent* with separate toggles — turn off everything you don't want.) |
| **Personalization → Lock screen** | *Personalize your lock screen* = **Picture** or **Slideshow** (not Windows Spotlight); *Get fun facts, tips, tricks and more on your lock screen* **off**; *Lock screen status* = **None**. |
| **Personalization → Background** | Don't use **Windows Spotlight** (adds a "Learn about this picture" icon and Bing content). |
| **Personalization → Taskbar** | *Widgets* off; *Search* = *Search icon only* or *Hide*; *Task view* as you like. |
| **Privacy & security → Recommendations & offers** (replaces *General* on current 25H2) | All five **off**: *Get personalized tips, ads, and recommendations based on my Windows activity* (formerly *Tailored experiences*); *Let apps show me personalized ads by using my advertising ID*; *Let websites show me locally relevant content by accessing my language list*; *Let Windows improve Start and search results by tracking app launches*; *Show me suggested content in the Settings app*. On older builds the same toggles are under **Privacy & security → General** (+ *Tailored experiences* under *Diagnostics & feedback*). |
| **Privacy & security → Search** | ✅ **New (Aug/Sep 2026 updates, gradual rollout): turn off web and Store suggestions/results in search**. *Show search highlights* **off**. |
| **System → Notifications → Additional settings** | Untick all three: *Show the Windows welcome experience…*, *Suggest ways to get the most out of Windows…*, *Get tips and suggestions when using Windows*. |
| **System → Notifications** | Turn off notifications from apps you don't want to hear from (e.g. *Suggested*, *Tips*, Microsoft Store promos). |
| **File Explorer → ⋯ → Options** | *General*: untick *Show recommended section* / *Show files from Office.com*. *View*: untick **Show sync provider notifications**. |
| **Microsoft Store → profile → Settings** | ⚙️ Turn off video autoplay. Keep **App updates on**. |
| **EEA:** Search → Bing | With the Ireland device region, Bing web results in Windows Search come from the removable *Web Search from Microsoft Bing* app (`Microsoft.BingSearch`). After §4 removes it — and/or with the Search toggle above off — search is local-only. |

### Pro-effective policies ✅

| Policy | Registry | Pro? |
|---|---|---|
| Admin Templates → Windows Components → Widgets → **Allow widgets = Disabled** | `HKLM\SOFTWARE\Policies\Microsoft\Dsh` `AllowNewsAndInterests = 0` | ✅ |
| Admin Templates → Windows Components → Search → **Allow search highlights = Disabled** | `HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search` `EnableDynamicContentInWSB = 0` | ✅ |
| Admin Templates → Start Menu and Taskbar → **Remove Recommended section from Start Menu = Enabled** | `HKLM\SOFTWARE\Policies\Microsoft\Windows\Explorer` `HideRecommendedSection = 1` | ⚠ Microsoft's CSP table now lists Pro, older ADMX text said Education only — verify with the audit; the Settings toggles above do the job anyway |
| Admin Templates → Start Menu and Taskbar → **Remove Personalized Website Recommendations from the Recommended section in the Start Menu = Enabled** | same key, `HideRecommendedPersonalizedSites = 1` | ✅ |
| *User Config* → Admin Templates → Windows Components → File Explorer → **Turn off display of recent search entries in the File Explorer search box = Enabled** — also removes Bing web results and web suggestions from Start search (the answer file's "Do not show Bing results" option sets this) | `HKCU\Software\Policies\Microsoft\Windows\Explorer` `DisableSearchBoxSuggestions = 1` | ✅ (side effect: no recent-search drop-down in File Explorer's search box; searching still works) |
| *User Config* → Admin Templates → Windows Components → Cloud Content → **Do not suggest third-party content in Windows spotlight = Enabled** | `HKCU\Software\Policies\Microsoft\Windows\CloudContent` `DisableThirdPartySuggestions = 1` | ✅ |
| Admin Templates → Control Panel → **Allow Online Tips = Disabled** (Settings tips/online content) | `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer` `AllowOnlineTips = 0` | ✅ |

```powershell
reg add "HKLM\SOFTWARE\Policies\Microsoft\Dsh" /v AllowNewsAndInterests /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Windows Search" /v EnableDynamicContentInWSB /t REG_DWORD /d 0 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Explorer" /v HideRecommendedSection /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\Explorer" /v HideRecommendedPersonalizedSites /t REG_DWORD /d 1 /f
reg add "HKCU\Software\Policies\Microsoft\Windows\Explorer" /v DisableSearchBoxSuggestions /t REG_DWORD /d 1 /f
reg add "HKCU\Software\Policies\Microsoft\Windows\CloudContent" /v DisableThirdPartySuggestions /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer" /v AllowOnlineTips /t REG_DWORD /d 0 /f
```

**Not available on Pro** (don't bother, they're ignored): *Turn off Microsoft consumer experiences*, *Do not show Windows tips*, *Turn off cloud optimized content*, all *Windows Spotlight* policies except *Do not suggest third-party content in Windows spotlight* (used above), *Don't search the web or display web results in Search*, *DisableShareAppPromotions*, *DisableGetStarted*. The Settings toggles above cover the same surfaces.

---

## 7. AI features

| Feature | What to do | Pro? |
|---|---|---|
| **Copilot app** | Removed in §4 (`Microsoft.Copilot`). **Confirm under *Settings → Apps → Installed apps*** — if Copilot is still listed (some Store builds of it may not be caught by the Appx cmdlets), uninstall it there. If it ever returns, uninstall it again. | ✅ |
| **Microsoft 365 Copilot app** (`Microsoft.MicrosoftOfficeHub`) | Removed in §4. Its automatic install only targets PCs with Microsoft 365 desktop apps, and EEA devices are exempt. | ✅ |
| ⚙️ **Block Copilot from reinstalling (advanced)** | Microsoft's documented durable block is an **AppLocker packaged-app Deny rule** for `MICROSOFT.COPILOT` (publisher `CN=MICROSOFT CORPORATION, O=MICROSOFT CORPORATION, L=REDMOND, S=WASHINGTON, C=US`). ⛔ **Warning:** as soon as any packaged-app rule exists, *every packaged app not explicitly allowed is blocked* — you must first create the **default rules** (*secpol.msc → Application Control Policies → AppLocker → Packaged app Rules → Create Default Rules*) and the *Application Identity* service must be running. Only do this if Copilot keeps coming back; follow *Manage Copilot* on Microsoft Learn step by step. | ✅ (AppLocker is enforceable on all Win 11 editions) |
| `TurnOffWindowsCopilot` policy | ⛔ Deprecated — does nothing for the current Copilot app. Skip. | — |
| `RemoveMicrosoftCopilotApp` policy (Apr 2026) | ⛔ Skip — it only fires if Microsoft 365 Copilot is also installed and the app hasn't been used for 28 days. Useless on your PC. | — |
| **Copilot key** (if your keyboard has one) | *Settings → Personalization → Text input → Customize Copilot key on keyboard* → map it to *Search* or another app. | ✅ |
| **"AI actions" in File Explorer's right-click menu** | *Settings → Apps → Actions* → turn off everything. When nothing is enabled, the *AI actions* menu disappears. ⚠ Undocumented registry alternative: `HKLM\SOFTWARE\Policies\Microsoft\Windows\Explorer` `HideAIActionsMenu = 1`. | ✅ |
| **Apps using Windows' built-in AI models** | *Settings → Privacy & security → Text and image generation* → off. Policy: Admin Templates → Windows Components → App Privacy → **Let Windows apps make use of Text and image generation features of Windows = Enabled, Default for all apps: Force Deny** → `HKLM\SOFTWARE\Policies\Microsoft\Windows\AppPrivacy` `LetAppsAccessSystemAIModels = 2`. | ✅ ⚠ (in the ADMX; not yet on Learn) |
| **Paint** (Cocreator, Generative fill/erase, Image Creator) | Admin Templates → Windows Components → **Paint** → disable each; or registry `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Paint` `DisableCocreator = 1`, `DisableGenerativeFill = 1`, `DisableImageCreator = 1`. | ✅ |
| **Notepad** (Rewrite / Summarize / Write) | Registry `HKLM\SOFTWARE\Policies\WindowsNotepad` `DisableAIFeatures = 1` (Notepad 11.2503.16+; the gpedit template is a separate download from Microsoft). Also Notepad → Settings → turn off AI features / sign out. | ✅ |
| **Photos, Snipping Tool** | Microsoft removed their Copilot buttons in 2026. No policy; turn off anything AI-related in each app's settings. | — |
| **Recall** | Only exists on **Copilot+ PCs** (NPU ≥ 40 TOPS). If you have one: `AllowRecallEnablement = 0` and `DisableAIDataAnalysis = 1` under `HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsAI`, or remove the feature: `Disable-WindowsOptionalFeature -Online -FeatureName Recall -Remove`. Reboot. | ✅ |
| **Click to Do** | Copilot+ only. *Settings → Privacy & security → Click to Do* off, or `HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsAI` `DisableClickToDo = 1`. | ✅ ⚠ |
| **AI components** | Copilot+ only: *Settings → System → AI components* → remove what's listed. | ✅ |
| **Edge** (if you kept it) | *Edge → Settings → Copilot and sidebar*: turn Copilot and the sidebar off — for a personal profile, **the Copilot toolbar button can only be turned off here by hand** (since Edge 141 its policy only applies to work/Entra profiles). *Settings → Privacy, search, and services*: turn off personalization/"AI innovations"-type features and shopping/"Follow creators". Policy `HKLM\SOFTWARE\Policies\Microsoft\Edge` `HubsSidebarEnabled = 0` hides the sidebar for local (not Microsoft-account-signed-in) profiles. | ✅ |

```powershell
$paint = 'HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Paint'
reg add $paint /v DisableCocreator /t REG_DWORD /d 1 /f
reg add $paint /v DisableGenerativeFill /t REG_DWORD /d 1 /f
reg add $paint /v DisableImageCreator /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Policies\WindowsNotepad" /v DisableAIFeatures /t REG_DWORD /d 1 /f
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\AppPrivacy" /v LetAppsAccessSystemAIModels /t REG_DWORD /d 2 /f
# Copilot+ PCs only:
# $ai = 'HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsAI'
# reg add $ai /v AllowRecallEnablement /t REG_DWORD /d 0 /f
# reg add $ai /v DisableAIDataAnalysis /t REG_DWORD /d 1 /f
# reg add $ai /v DisableClickToDo /t REG_DWORD /d 1 /f
```

---

## 8. Windows Update hygiene (keep it on, make it polite)

| | Setting |
|---|---|
| ✅ | Keep automatic updates **on**. Set **Active hours** (*Windows Update → Advanced options*). |
| ✅ | *Windows Update → Get the latest updates as soon as they're available* = **off** — this is how new features (often AI) are rolled out early. Security fixes are unaffected. |
| ✅ | **Delivery Optimization:** *Advanced options → Delivery Optimization → Allow downloads from other PCs* **off**, or policy Admin Templates → Windows Components → Delivery Optimization → **Download Mode = HTTP only (0)** → `HKLM\SOFTWARE\Policies\Microsoft\Windows\DeliveryOptimization` `DODownloadMode = 0`. ⛔ Never disable the DoSvc service and never use mode 100 ("Bypass") — Microsoft says it's deprecated and can make downloads fail. |
| ⚙️ | *Receive updates for other Microsoft products* — on if you'll use Visual Studio / .NET SDKs, else either way. |
| ⚙️ | **Keep your GPU driver from being replaced:** Admin Templates → Windows Components → Windows Update → Manage updates offered from Windows Update → **Do not include drivers with Windows Updates = Enabled** (`HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate` `ExcludeWUDriversInQualityUpdate = 1`). Trade-off: you then update *all* drivers (chipset, Wi-Fi, etc.) yourself. |
| ⚙️ | When **26H2** is offered (expected Oct 2026) as *"Feature update to Windows 11, version 26H2"*, install it — it's a small enablement package. Then re-run the §11 audit. |

```powershell
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\DeliveryOptimization" /v DODownloadMode /t REG_DWORD /d 0 /f
```

---

## 9. Programming setup

### System → Advanced (replaces "For developers" and Dev Home) ✅
*Settings → System → Advanced*:
- **Enable long paths** (if not set by the answer file).
- **sudo** → on, mode **In a new window** (`forceNewWindow`). ⛔ Avoid *Input closed* (`disableInput`) and *Inline* (`normal`) — Microsoft documents security risks for those two configurations. CLI: `sudo config --enable forceNewWindow`.
- **File Explorer:** show file extensions, hidden/system files, full path in title bar.
- **End task** in taskbar right-click → on.
- **PowerShell execution policy** → allow local scripts (RemoteSigned).
- **Terminal:** default terminal app = Windows Terminal.
- ⚙️ **Developer Mode** — only if you need to sideload apps or create symlinks without admin. It lowers security a little.

### Dev Drive ✅ (instead of Defender exclusions)
Dev Drive is a ReFS volume that Defender scans in **performance mode** (asynchronously) — Microsoft calls it "a secure alternative to folder exclusions".

1. *Settings → System → Storage → Advanced storage settings → Disks & volumes → Create Dev Drive* — new VHD(X) (≥ 50 GB, dynamic) or a spare partition/unallocated space.
2. Check: `fsutil devdrv query D:` → should say it's a *trusted developer volume* with antivirus *performance mode*.
3. Put **source repos, package caches and build output** on it. Keep tools/SDKs/IDEs on C:.
4. Point package caches at it (user environment variables — *Settings → System → About → Advanced system settings → Environment Variables*):

| Tool | Variable → example value |
|---|---|
| npm | `npm_config_cache` → `D:\packages\npm` |
| pip | `PIP_CACHE_DIR` → `D:\packages\pip` |
| Cargo | `CARGO_HOME` → `D:\packages\cargo` |
| NuGet | `NUGET_PACKAGES` → `D:\packages\nuget` |
| Gradle | `GRADLE_USER_HOME` → `D:\packages\gradle` |
| Maven | `MAVEN_OPTS` → `-Dmaven.repo.local=D:\packages\maven` |
| vcpkg | `VCPKG_DEFAULT_BINARY_CACHE` → `D:\packages\vcpkg` |

⛔ Don't run `fsutil devdrv enable /disallowAv` and ⛔ don't add broad Defender folder exclusions. ⛔ Don't keep WSL projects on the Dev Drive — keep them *inside* the WSL Linux filesystem.

### Tooling via winget ✅
```powershell
winget install --id Git.Git -e
winget install --id Microsoft.PowerShell -e          # PowerShell 7
winget install --id Microsoft.VisualStudioCode -e
winget install --id Microsoft.PowerToys -e
# add your stacks, e.g.: Python.Python.3.13, OpenJS.NodeJS.LTS, Microsoft.DotNet.SDK.9, Rustlang.Rustup
```
- **App execution aliases:** if you install Python from python.org/winget, turn off the `python.exe` / `python3.exe` Store aliases in *Settings → Apps → Advanced app settings → App execution aliases*.
- ⚙️ Reproducible setup: save your installs as a **WinGet Configuration** file and re-apply with `winget configure <file>` after future reinstalls.
- OpenSSH client and Windows Terminal are built in.

### WSL2 ✅
```powershell
wsl --install            # installs WSL + Ubuntu; reboot when asked
wsl --update
```
⚙️ Mirrored networking (IPv6, localhost both ways, better VPN behaviour) — create `%UserProfile%\.wslconfig`:
```ini
[wsl2]
networkingMode=mirrored
```
Then `wsl --shutdown`. systemd is on by default in current Ubuntu images (`/etc/wsl.conf` → `[boot] systemd=true`).

### ⚙️ Virtualization extras (Pro)
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All      # Hyper-V
Enable-WindowsOptionalFeature -Online -FeatureName Containers-DisposableClientVM # Windows Sandbox
```

---

## 10. Gaming setup

### ✅ Recommended
| Setting | Where |
|---|---|
| **Game Mode** on (default) | *Settings → Gaming → Game Mode* |
| **Hardware-accelerated GPU scheduling** on (required for NVIDIA DLSS Frame Generation) | *Settings → System → Display → Graphics → Advanced graphics settings* |
| **Optimizations for windowed games** on (lower latency for DX10/11 borderless games) | *Settings → System → Display → Graphics* (default graphics settings) |
| **Variable refresh rate** / **Auto HDR** on if your monitor supports them; set the correct **refresh rate** | *System → Display → Advanced display* and *Graphics* |
| **Power plan: Balanced** (with the vendor chipset driver installed) | *Control Panel → Power Options*. High Performance gains ~0–2 FPS on desktops; "Ultimate Performance" is no better. On laptops use *Best performance* power mode while plugged in. |
| **GPU driver from the vendor** (and ⚙️ block Windows Update drivers — §8) | NVIDIA / AMD / Intel |
| **Keep the security baseline**: TPM 2.0, Secure Boot, VBS + Memory Integrity, IOMMU, Defender | Needed by Riot Vanguard (incl. the June 2026 on-demand mode: 25H2+, Secure Boot, TPM, VBS, HVCI, IOMMU), **FACEIT (TPM + Secure Boot mandatory; Windows 11 required from 14 Oct 2026)**, EA Javelin (Battlefield 6), Call of Duty. Memory Integrity costs ~a few % FPS on modern CPUs. |
| Steam/Epic library on a normal NTFS drive (not the Dev Drive) | — |

### ⛔ Avoid (placebo or harmful)
- Disabling **HPET** / `bcdedit` timer tweaks — gains ~<1 %, inconsistent, can cause stutter.
- **"Ultimate Performance"** plan and core-parking hacks — no measurable benefit on modern CPUs (X3D chips rely on the chipset driver, not parking hacks).
- Disabling **Defender**, **VBS/Memory Integrity**, **services** via scripts — breaks anti-cheat and updates for no proven gain.
- **Hosts-file / firewall telemetry blocklists** — break Update/Store.
- **AtlasOS, ReviOS, Tiny11 (core), MicroWin/"Win11 Creator" ISOs** — see §12.

---

## 11. Audit — run after setup and after every feature update

Read-only (`Get-*` only). Open **Windows PowerShell as admin** and paste:

```powershell
Write-Host "`n== Core services (StartType must not be Disabled) ==" -ForegroundColor Cyan
Get-Service wuauserv, UsoSvc, BITS, CryptSvc, DoSvc, WinDefend, SecurityHealthService, InstallService, AppXSvc, ClipSVC |
  Select-Object Name, Status, StartType | Format-Table -AutoSize

Write-Host "== Defender ==" -ForegroundColor Cyan
Get-MpComputerStatus | Select-Object AMServiceEnabled, AntivirusEnabled, RealTimeProtectionEnabled, IsTamperProtected, AntivirusSignatureLastUpdated | Format-List

Write-Host "== Secure Boot / TPM / VBS ==" -ForegroundColor Cyan
"Secure Boot: " + (Confirm-SecureBootUEFI)
Get-Tpm | Select-Object TpmPresent, TpmReady | Format-List
$dg = Get-CimInstance -Namespace root\Microsoft\Windows\DeviceGuard -ClassName Win32_DeviceGuard
"VBS running: " + ($dg.VirtualizationBasedSecurityStatus -eq 2)
"Memory integrity (HVCI) running: " + ($dg.SecurityServicesRunning -contains 2)

Write-Host "== Must be present ==" -ForegroundColor Cyan
'Microsoft.WindowsStore','Microsoft.StorePurchaseApp','Microsoft.DesktopAppInstaller','Microsoft.SecHealthUI' | ForEach-Object {
  "{0,-35} {1}" -f $_, [bool](Get-AppxPackage -Name $_) }

Write-Host "== Must be absent (bloat) ==" -ForegroundColor Cyan
$bloat = 'Clipchamp.Clipchamp','Microsoft.BingNews','Microsoft.BingWeather','Microsoft.BingSearch',
  'Microsoft.MicrosoftOfficeHub','Microsoft.Copilot','Microsoft.OutlookForWindows','MSTeams','Microsoft.Todos',
  'Microsoft.PowerAutomateDesktop','Microsoft.MicrosoftSolitaireCollection','Microsoft.GetHelp','Microsoft.Getstarted',
  'Microsoft.WindowsFeedbackHub','Microsoft.WindowsMaps','Microsoft.Windows.DevHome','Microsoft.GamingApp','Microsoft.XboxGamingOverlay'
$found  = @(Get-AppxPackage -AllUsers | Where-Object { $bloat -contains $_.Name } | Select-Object -ExpandProperty Name)
$found += Get-AppxProvisionedPackage -Online | Where-Object { $bloat -contains $_.DisplayName } | Select-Object -ExpandProperty DisplayName
if ($found) { $found | Sort-Object -Unique | ForEach-Object { "RETURNED: $_" } } else { "none - OK" }

Write-Host "== Policies ==" -ForegroundColor Cyan
$checks = @(
  @('HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection','AllowTelemetry',1),
  @('HKLM:\SOFTWARE\Policies\Microsoft\Windows\DataCollection','DoNotShowFeedbackNotifications',1),
  @('HKCU:\Software\Policies\Microsoft\Windows\CloudContent','DisableTailoredExperiencesWithDiagnosticData',1),
  @('HKCU:\Software\Policies\Microsoft\Windows\CloudContent','DisableThirdPartySuggestions',1),
  @('HKCU:\Software\Policies\Microsoft\Windows\Explorer','DisableSearchBoxSuggestions',1),
  @('HKLM:\SOFTWARE\Policies\Microsoft\Windows\System','EnableActivityFeed',0),
  @('HKLM:\SOFTWARE\Policies\Microsoft\Windows\AdvertisingInfo','DisabledByGroupPolicy',1),
  @('HKLM:\SOFTWARE\Policies\Microsoft\Windows\Windows Search','AllowCloudSearch',0),
  @('HKLM:\SOFTWARE\Policies\Microsoft\Windows\Windows Search','EnableDynamicContentInWSB',0),
  @('HKLM:\SOFTWARE\Policies\Microsoft\Dsh','AllowNewsAndInterests',0),
  @('HKLM:\SOFTWARE\Policies\Microsoft\Windows\Explorer','HideRecommendedSection',1),
  @('HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Paint','DisableCocreator',1),
  @('HKLM:\SOFTWARE\Policies\WindowsNotepad','DisableAIFeatures',1),
  @('HKLM:\SOFTWARE\Policies\Microsoft\Windows\AppPrivacy','LetAppsAccessSystemAIModels',2),
  @('HKLM:\SOFTWARE\Policies\Microsoft\Windows\DeliveryOptimization','DODownloadMode',0)
)
foreach ($c in $checks) {
  $v = (Get-ItemProperty -Path $c[0] -Name $c[1] -ErrorAction SilentlyContinue).($c[1])
  $state = if ($null -eq $v) { 'MISSING' } elseif ($v -eq $c[2]) { 'OK' } else { "WRONG ($v)" }
  "{0,-8} {1}" -f $state, $c[1]
}

Write-Host "== Device region (EEA = Ireland is 68) ==" -ForegroundColor Cyan
(Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\Control Panel\DeviceRegion' -ErrorAction SilentlyContinue).DeviceRegion
```

**Reading it:** services may show *Stopped* (many are trigger-start) — only **Disabled** is a problem. Anything under *RETURNED* → re-run the §4 removal block. *MISSING/WRONG* policies → re-apply that section. Also open **Microsoft Store → Library → Get updates** and **Windows Update → Check for updates** once to prove both work.

---

## 12. Appendix

### Third-party tools (you chose native-only — for reference)
- **Reasonable if you ever want one:** *Raphire Win11Debloat* (maintained PowerShell script; skip its update/Delivery Optimization options) and *O&O ShutUp10++* (individual reversible toggles; stay in the "recommended" set).
- **⛔ Avoid for this setup:** ChrisTitusTech *MicroWin / Win11 Creator* ISOs (MicroWin builds broke cumulative updates on 25H2 — error 0x800f0991), **Tiny11** (core variant can't update; bypasses TPM/Secure Boot), **AtlasOS / ReviOS** (Atlas restricts Windows Update to Defender definitions by default; lowers mitigations; unsupported).

### ⚠ Items that couldn't be fully verified (check with the §11 audit / by eye)
- `HideRecommendedSection` taking effect on Pro.
- `HideAIActionsMenu` (undocumented); `DisableClickToDo` on non-Insider builds.
- `LetAppsAccessSystemAIModels` is in the AppPrivacy ADMX but not yet listed on Learn.
- Copilot packaging: Learn removes it as the Appx `Microsoft.Copilot`; some tools treat the current Store build as non-Appx (Store ID `XP9CXNGPPJ97XX`). If it still shows in *Installed apps* after §4, uninstall it there.
- `Microsoft.BingSearch` may need `-PackageTypeFilter Bundle` to remove for all users — if the audit shows it RETURNED, uninstall it from *Installed apps* (it's listed as "Web Search from Microsoft Bing").
- The Game Bar pop-up fix values (`GameDVR_Enabled`, `AppCaptureEnabled`) are community-documented, not on Learn.
- Whether a DiagTrack disable has side effects.
- Exact inbox app list of a fresh 25H2 Pro image (the removal list skips absent packages safely).
- Exact generator label wording on schneegans.de.
- 26H2 GA date (press reports say October 2026).

### Sources (accessed 27 Sep 2026)
- Windows 11 release information — https://learn.microsoft.com/en-us/windows/release-health/windows11-release-information (updated 2026-09-22)
- 26H2 Release Preview — https://blogs.windows.com/windows-insider/2026/08/27/releasing-windows-11-version-26h2-to-the-release-preview-channel/
- KB5124008 (8 Sep 2026) — https://support.microsoft.com/en-us/servicing/os/windows-11/2026/09/kb5124008-windows-11-24h2-25h2-security-update · BleepingComputer coverage 2026-09-08
- Local-account bypass status — Windows Latest 2026-09-13; Pureinfotech (updated 2026-09-14); XDA 2026-07-22; schneegans issue #381 https://github.com/cschneegans/unattend-generator/issues/381
- Unattend generator — https://schneegans.de/windows/unattend-generator/
- Device region / DMA — Rafael Rivera, https://withinrafael.com/2023/11/17/device-region-and-integrated-services-region-policy-in-windows/ ; Windows Central DMA compliance article
- Policy CSPs: System https://learn.microsoft.com/en-us/windows/client-management/mdm/policy-csp-system · Experience …/policy-csp-experience · Start …/policy-csp-start · Search …/policy-csp-search · NewsAndInterests …/policy-csp-newsandinterests · WindowsAI …/policy-csp-windowsai (updated 2026-09-23) · Settings …/policy-csp-settings
- Manage connections from Windows components — https://learn.microsoft.com/en-us/windows/privacy/manage-connections-from-windows-operating-system-components-to-microsoft-services (2026-02-12)
- Manage Copilot — https://learn.microsoft.com/en-us/windows/client-management/manage-windows-copilot (2026-08-18) · Manage Recall …/manage-recall (2025-12-10) · Manage Click to Do …/manage-click-to-do (2026-08-18) · Manage Notepad …/manage-notepad
- RemoveMicrosoftCopilotApp — Windows Latest 2026-05-24
- Remove default Store packages policy (Ent/Edu only) — BleepingComputer 2025-10-27
- Windows quality commitments — https://blogs.windows.com/windows-insider/2026/03/20/our-commitment-to-windows-quality/ and 2026-07-31 update
- Delivery Optimization reference — https://learn.microsoft.com/en-us/windows/deployment/do/waas-delivery-optimization-reference (2026-05-20)
- Advanced settings / sudo — https://learn.microsoft.com/en-us/windows/advanced-settings/ (2026-06-02), …/advanced-settings/sudo/ (2026-02-26)
- Dev Drive — https://learn.microsoft.com/en-us/windows/dev-drive/ (2026-06-01)
- WSL networking — https://learn.microsoft.com/en-us/windows/wsl/networking (2026-06-02)
- WinGet Configuration — https://learn.microsoft.com/en-us/windows/package-manager/configuration/
- VBS/HVCI gaming impact — Tom's Hardware benchmarks; Memory Integrity auto-enable from Oct 2026 — Help Net Security 2026-09-03
- Anti-cheat requirements — Riot Vanguard support + on-demand mode (2026-06-24); FACEIT announcements (2025-11-25, 2026-10-14 Win 11 requirement); EA Javelin (PCGamesN/Corsair)
- Xbox mode / Advanced Shader Delivery — Windows Experience Blog 2026-04-30; DirectX dev blog (2026-06-11)
- Third-party tools — github.com/Raphire/Win11Debloat; christitus.com/winutil-in-2026 (2026-02-16), WinUtil known issues; docs.atlasos.net
