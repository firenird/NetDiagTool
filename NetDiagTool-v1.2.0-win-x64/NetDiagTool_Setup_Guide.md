# NetDiag Tool - Setup & Usage Guide

## Step 1 - Build the EXE (one time only)

You have the source code. Build it once on a Windows machine; after that the EXE runs anywhere without installation.

1. Install the **.NET 8 SDK (x64)**: https://dotnet.microsoft.com/download/dotnet/8.0
   (Needed only on the build machine - nothing has to be installed on the PCs where the EXE will run.)
2. Extract the ZIP, for example to `C:\Tools\NetDiagTool-src\`
3. Open **PowerShell** in that folder (Shift + Right-click inside the folder > "Open PowerShell window here") and run:

```powershell
powershell -ExecutionPolicy Bypass -File .\build.ps1
```

The script automatically runs: restore > build > tests > publish > zip. When it finishes, the output paths are shown in green:

```
publish\NetDiagTool\NetDiagTool.exe     <- GUI
publish\NetDiagTool\NetDiagCli.exe      <- command line
publish\NetDiagTool-v1.2.0-win-x64.zip
```

The first build takes a little longer because the Windows runtime is downloaded from NuGet (internet access required).
To skip the tests: `.\build.ps1 -SkipTests`

**Don't want to install the SDK?** Push the source to your own GitHub repository. The workflow in `.github/workflows/build.yml` builds it on a Windows runner and publishes the EXEs as an artifact (Actions tab > build > NetDiagTool-win-x64).

## Step 2 - Run the application

1. Copy the `publish\NetDiagTool\` folder anywhere you like (Desktop, `C:\Tools`, a USB drive).
2. Double-click **NetDiagTool.exe**. Administrator rights are not required.
3. On first run, `config.json`, `Logs\`, `Sample Files\` and the other folders are created automatically - everything stays inside this folder (fully portable).
4. When starting a Port Listener, if Windows Firewall prompts, click **Allow** (required if other machines need to connect to it).

## Step 3 - How to test each feature

| Feature | How to verify |
|---|---|
| Ping | Ping page > enter `8.8.8.8` > Ping. Shows per-packet results plus loss / min / avg / max. Use Stop to cancel |
| Multi Ping | Multi Ping > "Load sample" > Start > Export CSV / Save report |
| TCP / Telnet | TCP page > `google.com`, port `443` > Test = OPEN. Port `81` returns TIMEOUT / CLOSED |
| Custom message | First, on the Port Listen page set port `2323`, tick "Response enabled", click Start. Then on the TCP page: `127.0.0.1`, port `2323`, tick "Enable Custom TCP/Telnet Message", message `HELLO` > Test. The Received box shows `HELLO FROM NETDIAG`, and the listener page shows the client IP and received data |
| Multi TCP | Multi TCP > "Sample (ports)" or "Sample (messages)" > Start |
| Trace | Trace Route > `8.8.8.8` > Trace (live output on the right) |
| Multi Trace | Multi Trace > Load sample > Start (concurrency 3) |
| DNS | DNS page > `google.com` > Lookup / Compare (system resolver + 8.8.8.8 + 1.1.1.1). Enter an IP and click "Reverse (PTR)" |
| Port Monitor | Use the "Listening" quick filter, or type `443` in the Port box. Select a row to see process details below |
| Multi Listen | Port 8080 > Add Port, port 9000 > Add Port > Start All. A port already owned by another application shows "Port already in use"; the others keep running |
| SMTP | Settings > enter SMTP server / port / security / username / password / from / default recipient > **Test SMTP Connection** > **Send Test Email** > Save. Gmail / Office 365 require an app password (587 = STARTTLS, 465 = SSL/TLS) |
| Monitoring | Monitoring > select the sample "Production" profile > edit targets > Save > Start monitoring. Stop one of the targets: after the failure threshold you get a `[NetDiag ALERT] ... is DOWN` email plus an incident; when it comes back you get `[NetDiag RECOVERY] ... is UP` |
| Reports | Reports & History > choose a range > charts + availability > Save report (HTML / CSV / JSON) or Email report |
| Jobs | Jobs > sample "ProductionCheck" > Run now. Set a schedule and click "Create Windows task" to register it in Windows Task Scheduler (it runs even when NetDiag is closed) |
| Network Info | Adapters, IP, MAC, gateway, DNS, DHCP, link speed > Copy / Export |
| Tools | Diagnostics bundle ZIP, Backup / Restore, **Run self-diagnostics** (everything should be PASS; a WARN for SMTP is normal if it is not configured) |
| Tray | Settings > enable "Minimize to system tray" > minimize the window; monitoring and listeners keep running |

## Step 4 - Command line (scripts / Task Scheduler)

Open CMD or PowerShell in the `NetDiagTool` folder:

```powershell
.\NetDiagCli.exe help
.\NetDiagCli.exe ping 8.8.8.8
.\NetDiagCli.exe tcp server01 443
.\NetDiagCli.exe ping --csv "Sample Files\Sample_Ping.csv" --out result.csv
.\NetDiagCli.exe ports --port 443
.\NetDiagCli.exe dns google.com --server 8.8.8.8,1.1.1.1
.\NetDiagCli.exe job run ProductionCheck
.\NetDiagCli.exe report --profile Production --last 24h --out report.html
.\NetDiagCli.exe selftest
```

Exit codes: `0` = success, `1` = some tests failed, `2` = invalid arguments, `3` = configuration error, `4` = runtime error, `5` = cancelled.
Use `NetDiagCli.exe` in scripts (CMD does not wait for the GUI EXE to finish).

## Troubleshooting

| Problem | Solution |
|---|---|
| `build.ps1`: "running scripts is disabled" | Run it exactly as shown: `powershell -ExecutionPolicy Bypass -File .\build.ps1` |
| ".NET 8 SDK not found" | Install the SDK, then close and reopen PowerShell |
| NuGet error during restore / publish | Check internet / proxy access - the win-x64 runtime is downloaded on the first publish |
| A closed port shows TIMEOUT instead of CLOSED | Windows reports a refused connection after ~2 seconds - keep the TCP timeout at 3000 ms or higher |
| Every ping times out | The target or a firewall in between blocks ICMP - use the TCP test instead |
| Port Listen: "Access denied" | Windows has reserved that port (`netsh int ipv4 show excludedportrange protocol=tcp`) - choose another port |
| Another machine cannot reach the listener | Allow NetDiagTool.exe in Windows Firewall and use the listen address `0.0.0.0` |
| SMTP error 535 | Wrong username / password, or the provider requires an app password |
| SMTP password does not work on another PC | DPAPI encryption is specific to the Windows user and PC - re-enter the password in Settings |
| Where is my data? | Tools > Folders, or the EXE folder (e.g. `Logs\YYYY-MM-DD\netdiag.log`) |
