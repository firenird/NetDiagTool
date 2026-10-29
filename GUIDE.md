# NetDiag Tool – Guide to Running It on Your PC

## Step 1 – Build the EXE (Only Once)

You have the source code. You need to build it once on Windows. After that, you can run the EXE on any compatible Windows machine.

1. Install the .NET 8 SDK (x64): [Download .NET 8](https://dotnet.microsoft.com/download/dotnet/8.0?utm_source=chatgpt.com)

   You only need the SDK on the machine used to build the application. You do not need to install it on the machine where you will run the published EXE.
2. Extract the ZIP file, for example, to `C:\Tools\NetDiagTool-src\`.
3. Open PowerShell in that folder (Shift + right-click inside the folder and select the PowerShell option), then run:
 

   powershell -ExecutionPolicy Bypass -File .\build.ps1
   

The script automatically performs the following steps:

- Restore dependencies
- Build the application
- Run tests
- Publish the application
- Create a ZIP package

At the end, the output paths will appear in green text:

```
publish\NetDiagTool\NetDiagTool.exe       <- GUI application
publish\NetDiagTool\NetDiagCli.exe        <- Command-line application
NetDiagTool-v1.2.0-win-x64.zip
```

The first build may take some time because the Windows runtime is downloaded through NuGet. An internet connection is required.

To skip the tests, run:

```
.\build.ps1 -SkipTests
```

Don't want to install the SDK?

Push the source code to your GitHub repository. The `.github/workflows/build.yml` workflow will build the application on a Windows runner and provide the EXE as an artifact.

Navigate to:

GitHub → Actions → build → Artifacts → NetDiagTool-win-x64

## Step 2 – Run the Application

1. Copy the `publish\NetDiagTool\` folder to any location you prefer, such as your Desktop, `C:\Tools\`, or a USB drive.
2. Double-click `NetDiagTool.exe`. Administrator privileges are not required for normal use.
3. On the first run, the application automatically creates `config.json`, `Logs\`, `Sample Files\`, and other required files. Everything stays in the same folder, making the application portable.
4. If Windows Firewall prompts you while starting Port Listen, click Allow if you need to test connections from another machine.

## Step 3 – How to Test Every Feature

| Feature        | How to test it                                                                                                                                                                                                                                                                                                                                          |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ping           | Open the Ping page, enter `8.8.8.8`, and click Ping. You will see per-packet results and packet loss, minimum, average, and maximum response times. Use Stop to cancel.                                                                                                                                                                                 |
| Multi Ping     | Open Multi Ping, click Load Sample, then Start. Export the results to CSV or save a report.                                                                                                                                                                                                                                                             |
| TCP / Telnet   | On the TCP page, enter `google.com` and port `443`, then click Test. The result should be OPEN if reachable. Port `81` may return TIMEOUT or CLOSED.                                                                                                                                                                                                    |
| Custom Message | First, open Port Listen, enter port `2323`, enable Response Enabled, and click Start. Then open the TCP page, enter `127.0.0.1` and port `2323`, enable Enable Custom TCP/Telnet Message, enter `HELLO`, and click Test. The Received field should show `HELLO FROM NETDIAG`. The listener page should display the client IP and received data.         |
| Multi TCP      | Open Multi TCP, select Sample (ports) or Sample (messages), and click Start.                                                                                                                                                                                                                                                                            |
| Trace Route    | Open Trace Route, enter `8.8.8.8`, and click Trace. Live output will appear on the right.                                                                                                                                                                                                                                                               |
| Multi Trace    | Open Multi Trace, click Load Sample, and click Start. Use a concurrency value of `3`.                                                                                                                                                                                                                                                                   |
| DNS            | Open DNS, enter `google.com`, and select Lookup or Compare to compare the system DNS with `8.8.8.8` and `1.1.1.1`. Enter an IP address and select Reverse (PTR) for a reverse lookup.                                                                                                                                                                   |
| Port Monitor   | Use the Listening quick filter or enter `443` in the Port field. Select a row to view process details below.                                                                                                                                                                                                                                            |
| Multi Listen   | Add ports `8080` and `9000`, then click Start All. If a port is already being used by another application, it will display Port already in use, while the remaining ports continue running.                                                                                                                                                             |
| SMTP           | Go to Settings and configure the SMTP server, port, security, username, password, sender address, and default recipient. Click Test SMTP Connection, then Send Test Email, and save. Gmail and Microsoft 365 may require an app password or another supported authentication method. Port `587` typically uses STARTTLS, while port `465` uses SSL/TLS. |
| Monitoring     | Open Monitoring, select the sample Production profile, edit the targets, save, and start monitoring. If a target goes offline, the application should generate an incident and send an email containing `[NetDiag ALERT] ... is DOWN` after the configured threshold. When the target recovers, it should send `[NetDiag RECOVERY] ... is UP`.          |
| Reports        | Open Reports & History, select a date range, and review the charts and availability data. Save the report as HTML, CSV, or JSON, or email it.                                                                                                                                                                                                           |
| Jobs           | Open Jobs and select the sample `ProductionCheck` job. Click Run Now. You can configure a schedule and select Create Windows Task to register it with Windows Task Scheduler. The scheduled job can run even when NetDiag is closed, provided its task configuration and executable paths are valid.                                                    |
| Network Info   | Review network adapters, IP addresses, MAC addresses, gateways, DNS, DHCP, and link speed. Use Copy or Export to save the information.                                                                                                                                                                                                                  |
| Tools          | Generate a diagnostics ZIP bundle, use Backup/Restore, and run Run Self-Diagnostics. All checks should PASS. A WARN result for SMTP is normal if SMTP has not been configured.                                                                                                                                                                          |
| System Tray    | Go to Settings and enable Minimize to System Tray. Minimize the application; monitoring and listeners should continue running in the background.                                                                                                                                                                                                        |

## Step 4 – Command-Line Usage (Scripts / Task Scheduler)

Open CMD or PowerShell in the `NetDiagTool` folder.

Run the following commands:

```
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

### Exit Codes

| Exit code | Meaning                           |
| --------- | --------------------------------- |
| `0`       | Everything completed successfully |
| `1`       | One or more tests failed          |
| `2`       | Invalid arguments                 |
| `3`       | Configuration error               |
| `4`       | Runtime error                     |
| `5`       | Operation cancelled               |

Important: Use `NetDiagCli.exe` for scripts and scheduled tasks. The GUI executable is not intended for command-line automation.

## Troubleshooting

| Problem                                              | Solution                                                                                                                                                                         |
| ---------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `build.ps1` reports that running scripts is disabled | Run `powershell -ExecutionPolicy Bypass -File .\build.ps1`.                                                                                                                      |
| `.NET 8 SDK not found`                               | Install the SDK, close PowerShell, and open it again.                                                                                                                            |
| NuGet error during restore or publish                | Check your internet connection and proxy settings. The first publish downloads the Windows x64 runtime.                                                                          |
| A closed port shows TIMEOUT instead of CLOSED        | Windows may take approximately two seconds to report a refused connection. Set the TCP timeout to 3000 ms or higher. Actual results depend on the network and firewall behavior. |
| All Ping tests show TIMEOUT                          | The destination or an intermediate firewall may be blocking ICMP traffic. Try a TCP test instead.                                                                                |
| Port Listen reports Access Denied                    | Windows may have reserved the port. Run `netsh int ipv4 show excludedportrange protocol=tcp` and try another port.                                                               |
| Another machine cannot connect to the listener       | Allow NetDiagTool through Windows Firewall and configure the listener to bind to `0.0.0.0` if connections from other interfaces are required.                                    |
| SMTP returns error 535                               | Check your username and password. Your provider may require an app password or a different supported authentication method.                                                      |
| SMTP password does not work on another PC            | DPAPI encryption is specific to the Windows user and machine. Re-enter the password in Settings on the new PC.                                                                   |
| Where is the application data stored?                | Open Tools → Folders, or check the application directory. Logs are stored under `Logs\YYYY-MM-DD\netdiag.log`.                                                                   |

Recommended first test: Build the application, launch `NetDiagTool.exe`, run the self-diagnostics, and test Ping, TCP on port `443`, DNS lookup, and Port Monitor before configuring SMTP and automated monitoring.