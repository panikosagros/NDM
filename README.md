Network Device Manager - installing
=====================================

Starting with version 1.2.0, the .msi installs the program files AND
automatically registers + starts the Windows Service - no separate
elevated command needed afterward. This also applies when installing a
new version over an existing older one (an upgrade): the installer stops
the currently-running service first (so it can safely replace its files),
then reinstalls and restarts it with the new version.

1. Run the .msi as an administrator (right-click -> "Run as
   administrator" is the safest way to make sure it's actually elevated -
   a plain double-click will usually still prompt via UAC, but running it
   as administrator up front avoids any doubt). Click through the
   installer as normal.

2. Give it a few seconds after the installer finishes - the service
   registration/start happens as part of the install itself, right at
   the end, so it may still be finishing for a moment after the "Setup
   complete" screen appears.

3. Confirm it's running:

       sc query NetworkDeviceManager

   Look for STATE: 4 RUNNING. It also shows up in services.msc as
   "Network Device Manager", set to start automatically on boot.

   Then check the port is actually listening:

       netstat -an | findstr 8812

4. Open https://localhost:8812 (or https://<this-server's-IP>:8812 from
   another machine) in a browser. Your browser will warn the certificate
   isn't trusted - that's expected for a self-signed certificate; proceed
   past the warning. You'll land on first-time setup (portal password,
   then a separate, stronger database secret) - unless this was an
   upgrade over an existing install, in which case your existing setup,
   resources, and backups are untouched (they live in a separate
   ProgramData folder the installer never touches).

   If connecting from another machine, you'll also need to allow port
   8812 through Windows Firewall on this server - that's a deliberate
   choice left to you rather than opened automatically.

Troubleshooting - if something doesn't work
----------------------------------------------
- Log file (the service, the automatic install/upgrade step, AND the
  console-mode .exe all write to the same one):
      C:\ProgramData\NSS MikroTIK Manager\app.log
  (Note: that folder is named "NSS MikroTIK Manager", not "Network Device
  Manager" - it's a leftover from an earlier product name and hasn't been
  renamed, but it's the real one.)

  Right after installing, this log will have a block of lines starting
  with "Installer: ensuring the Windows Service is installed and
  started." - followed by exactly what each step (stop the old one if
  running / install or update / start) did and whether it succeeded.
  This is the first place to look if the service isn't running after
  install - it will say specifically which step failed and why, not just
  "something went wrong."

- If that "Installer: ..." block is missing from the log ENTIRELY, the
  install's automatic step never ran at all (as opposed to running and
  hitting a problem) - re-run the installer AS ADMINISTRATOR (see step 1
  above), since without real elevation Windows may silently skip the
  parts of the install that need it. For a full trace of the installer
  itself (not just the service step), re-run it with verbose logging:

      msiexec /i "Network Device Manager-1.2.0-win64.msi" /l*v install.log

  and check install.log for "A_SVC_STOP" / "A_SVC_AUTOINSTALL" (the
  installer's internal names for the stop/install/start step) to confirm
  they were scheduled and ran.

- Windows' own Event Viewer (Windows Logs > Application, source
  "NetworkDeviceManager") also shows a start/stop/crash entry for the
  service itself, separate from the log file above.

- "Access is denied" anywhere in the above: the installer wasn't actually
  elevated - see step 1.

Manual fallback (only needed if the automatic install/start above didn't
work and you want to do it by hand while sorting out why): from an
elevated Command Prompt, cd to the install folder (default shown - adjust
if you changed it) and run any of:

       cd "C:\Program Files\Network Device Manager"
       NetworkDeviceManagerService.exe --startup auto install
       NetworkDeviceManagerService.exe start
       NetworkDeviceManagerService.exe stop
       NetworkDeviceManagerService.exe remove

   Before committing to a real install, you can also test that the
   service logic runs correctly with NO admin rights and nothing left
   behind - open a normal (non-elevated) Command Prompt, cd to the
   install folder, and run:

       NetworkDeviceManagerService.exe debug

   This runs the exact same server logic in the foreground, right there
   in that window - no registry entries, nothing persistent. Press Ctrl+C
   to stop it.

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
