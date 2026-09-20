Network Device Manager - installing as a Windows Service
==========================================================

The .msi installs the program files and a Start Menu shortcut, but does
NOT automatically register or start the Windows Service - that needs one
elevated command after installing, since registering a service is a
system-level change worth doing deliberately rather than silently as part
of running the installer.

1. Run the .msi as normal (double-click, click through).

   Optional but recommended first: before installing the service for
   real, you can test that it actually runs correctly with NO admin
   rights and nothing persistent - open a normal Command Prompt, cd to
   the install folder, and run:

       NetworkDeviceManagerService.exe debug

   This runs the exact same server logic in the foreground, right there
   in that window - no registry entries, nothing left behind. Press
   Ctrl+C to stop it. If https://localhost:8812 works while this is
   running, the real service (step 4 below) will too - this just proves
   it before you commit to installing it as one.

2. Open Command Prompt or PowerShell AS ADMINISTRATOR (right-click ->
   "Run as administrator"). This step needs admin rights; the rest don't.

3. Go to the install folder (default shown - adjust if you changed it):

       cd "C:\Program Files\Network Device Manager"

4. Install the service. To have it start automatically every time Windows
   boots (recommended for a server):

       NetworkDeviceManagerService.exe --startup auto install

   Or, to install it but start it manually yourself each time instead:

       NetworkDeviceManagerService.exe install

5. Start it now (skip this if you used --startup auto and are about to
   reboot anyway - it'll start on its own):

       NetworkDeviceManagerService.exe start

6. Confirm it's running:

       sc query NetworkDeviceManager

   Look for STATE: 4 RUNNING. It also shows up in services.msc as
   "Network Device Manager".

   Then check the port is actually listening:

       netstat -an | findstr 8812

7. Open https://localhost:8812 (or https://<this-server's-IP>:8812 from
   another machine) in a browser. Your browser will warn the certificate
   isn't trusted - that's expected for a self-signed certificate; proceed
   past the warning. You'll land on first-time setup (portal password,
   then a separate, stronger database secret).

   If connecting from another machine, you'll also need to allow port
   8812 through Windows Firewall on this server - that's a deliberate
   choice left to you rather than opened automatically.

Troubleshooting
----------------
- Log file (both the service and the console-mode .exe write to the same
  one): C:\ProgramData\NSS MikroTIK Manager\app.log
  (Note: that folder is named "NSS MikroTIK Manager", not "Network Device
  Manager" - it's a leftover from an earlier product name and hasn't been
  renamed, but it's the real one.)
- If the service won't start, that log almost always has the reason.
  Windows' own Event Viewer (Windows Logs > Application, source
  "NetworkDeviceManager") will also show a start/stop/crash entry.
- Service not listed at all: step 4 didn't run, or didn't run elevated.
- "Access is denied" on install/start: the Command Prompt wasn't actually
  running as Administrator.

Uninstalling the service (the .msi's own uninstall does NOT do this -
do it first, separately, before uninstalling via Control Panel):

       NetworkDeviceManagerService.exe stop
       NetworkDeviceManagerService.exe remove

Running it without a service instead
--------------------------------------
NetworkDeviceManager.exe (Start Menu shortcut, or run directly) runs the
exact same app in a console window you keep open - no admin rights or
service registration needed, useful for a quick look without committing
to the service.
