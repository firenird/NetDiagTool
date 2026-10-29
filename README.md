# NetDiag Tool v1.2

Portable Windows network diagnostic & monitoring toolkit (freeware / internal use, no telemetry).
C# / .NET 8 / WPF (MVVM), single self-contained EXE, no installer, no .NET runtime needed on the target PC.

| Version | Features |
|---|---|
| **V1.0** | Dashboard, Single/Multi Ping (CSV), Single/Multi TCP "Telnet" test (CSV), Custom TCP/Telnet Message, Single/Multi Trace Route (CSV), Port Monitor (TCP/UDP + PID/process), Single & Multi Port Listen with optional response, logging, CSV import/export, settings (`config.json`) |
| **V1.1** | SMTP (None/STARTTLS/SSL-TLS, DPAPI-encrypted password), test email, HTML email reports with CSV attachments, continuous Ping/TCP monitoring, profiles, failure/recovery alerts with thresholds + cooldown, incidents (`INC-yyyyMMdd-nnnnn`), history, availability, background email queue, system tray |
| **V1.2** | CLI (`NetDiagCli.exe` + `NetDiagTool.exe <cmd>`), multi-stage jobs, in-app scheduler + Windows Task Scheduler integration, DNS diagnostics (forward/reverse/custom server/compare), network adapter info, charts, HTML/CSV/JSON reports, slow-target detection, latency alerts, notification center, diagnostics bundle, backup/restore, self-diagnostics |

---

## 1. Build the EXE (one time, on any Windows 10/11 x64 PC)

1. Install the **.NET 8 SDK (x64)**: https://dotnet.microsoft.com/download/dotnet/8.0
2. Extract this source folder, open **PowerShell** in it and run:

```powershell
powershell -ExecutionPolicy Bypass -File .\build.ps1
```

This restores, builds, runs the test suite, publishes both EXEs and creates:

```
publish\NetDiagTool\NetDiagTool.exe      <- GUI (double-click)
publish\NetDiagTool\NetDiagCli.exe       <- console CLI (scripts / Task Scheduler)
publish\NetDiagTool-v1.2.0-win-x64.zip   <- copy this anywhere / USB drive
```

Manual equivalent of the spec command:

```powershell
dotnet publish src/NetDiagTool -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:PublishTrimmed=false
dotnet publish src/NetDiagCli  -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:PublishTrimmed=false
```

No SDK on your PC? Push the folder to a GitHub repo - `.github/workflows/build.yml` builds it on a Windows runner and gives you the EXEs as a downloadable artifact.

Tests only: `dotnet run -c Release --project tests/NetDiagTool.Tests` (exit code 0 = all passed; loopback only, no internet needed).

## 2. Run

Copy `publish\NetDiagTool\` anywhere and double-click **NetDiagTool.exe**. No admin rights needed.
All data stays in that folder (portable). If the folder is read-only (e.g. Program Files) data goes to `%LOCALAPPDATA%\NetDiagTool`.

```
NetDiagTool\
  NetDiagTool.exe  NetDiagCli.exe  config.json
  Profiles\  Jobs\  Logs\YYYY-MM-DD\  Reports\  Exports\  Backups\  Data\  Sample Files\
```

Windows Firewall may ask once when you start a **Port Listen** on `0.0.0.0` - allow it on the networks where other machines must reach your listener.

## 3. Pages (GUI)

| Page | What it does |
|---|---|
| Dashboard | Tests today (total/success/failed/timeout), open TCP ports, active listeners, running monitors, open incidents, recent activity, 24h availability |
| Ping / Multi Ping | ICMP via .NET APIs. Per-packet results, loss, min/avg/max. CSV `Name,Target`, concurrency (default 10), export, report, email |
| TCP / Telnet | TcpClient connect test: DNS resolution, OPEN/CLOSED/TIMEOUT/ERROR, response time, retries. **Custom TCP/Telnet Message**: UTF-8/ASCII, CR/LF/CRLF, `\r \n \t \xHH` escapes, wait-for-reply + reply timeout. Generic payload test - no protocol is emulated, replies are never invented |
| Multi TCP | CSV `Name,Target,Port[,Message,Encoding,WaitForReply]`, every retry exported as its own row |
| Trace Route / Multi Trace | `tracert.exe` hidden, live output, cancellable, raw output saved; hop table + analysis (unreached target, latency jumps). Built-in ICMP engine as fallback |
| DNS | System resolver + raw queries to chosen DNS servers (A/AAAA/CNAME/MX/TXT/NS/PTR), compare servers, reverse lookup, measured times |
| Port Monitor | GetExtendedTcpTable/UdpTable (IPv4+IPv6) with PID, process, path, start time. Filters: port, PID, app, protocol, state, local/remote; quick filters; auto refresh. Never kills processes |
| Port Listen / Multi Listen | Diagnostic TCP listeners on any address/port, response message (OnConnect or AfterData), receive timeout, keep-open, client IP/port, received/sent data, "Port already in use" detection, duplicates blocked, all stopped on exit |
| Monitoring | Profiles (new/save/duplicate/delete/import/export), Ping/TCP targets, interval/timeout/retries, failure & recovery thresholds, cooldown, latency alert, live status, incidents |
| Reports & History | Range (hour/24h/7d/30d/custom), charts (response, availability, failures), previous-period comparison, availability/slow targets/incidents, save HTML/CSV/JSON, email |
| Jobs & Scheduling | Multi-stage jobs (Ping, Tcp, Trace, TraceFailed, Dns, Report, Email), stop conditions, schedule (once / minutes / hours / daily / weekly), in-app scheduler, **Create Windows task** |
| Network Info | Computer, Windows version, adapters, IPv4/IPv6, MAC, gateway, DNS, DHCP, link speed - copy/export |
| Tools | Diagnostics bundle ZIP (secrets removed), backup/restore, self-diagnostics, folder shortcuts |
| Notifications | Alerts, recoveries, job results, email failures (also tray balloons) |
| Settings | All defaults, logging, retention, UI/tray, SMTP + Test SMTP Connection + Send Test Email |

## 4. CLI

```powershell
NetDiagCli.exe ping 8.8.8.8
NetDiagCli.exe ping --csv Servers.csv --concurrency 20 --out results.csv
NetDiagCli.exe tcp server01 443
NetDiagCli.exe tcp 10.10.10.20 23 --message "HELLO" --append crlf --reply-timeout 3000
NetDiagCli.exe tcp --csv Ports.csv --format json
NetDiagCli.exe trace 8.8.8.8
NetDiagCli.exe dns internal.company.local --server 192.168.1.1,8.8.8.8,1.1.1.1
NetDiagCli.exe ports --port 443
NetDiagCli.exe ports --listening --app sqlservr
NetDiagCli.exe listen --port 9000,8080 --response "ACK FROM NETDIAG" --duration 300
NetDiagCli.exe netinfo
NetDiagCli.exe job list
NetDiagCli.exe job run ProductionCheck
NetDiagCli.exe job schedule ProductionCheck        # creates the Windows Scheduled Task
NetDiagCli.exe monitor Production --duration 8h
NetDiagCli.exe report --profile Production --last 24h --out report.html --email
NetDiagCli.exe bundle | backup | restore <zip> | selftest | help
```

Output: `--format human|csv|json`, `--out file`. Exit codes: `0` success, `1` tests completed with failures, `2` invalid arguments, `3` configuration error, `4` runtime error, `5` cancelled.
Secrets are never accepted as arguments (`--password` is rejected); SMTP uses the stored, encrypted settings.
`NetDiagTool.exe` accepts the same commands, but use `NetDiagCli.exe` in scripts: a GUI EXE does not make cmd.exe wait.

## 5. CSV formats

```csv
Name,Target                                   (Multi Ping / Multi Trace)
Name,Target,Port                              (Multi TCP)
Name,Target,Port,Message,Encoding,WaitForReply  (Multi TCP with custom messages)
Address,Port,ResponseEnabled,ResponseMessage,Encoding,Append  (Multi Listen)
```

UTF-8 (BOM ok), quoted fields, empty and `#` comment rows skipped, invalid rows reported with line numbers and skipped (the rest still runs), max 100,000 rows / 50 MB. Samples are in `Sample Files\`.

## 6. Security

- SMTP password: Windows DPAPI (current user) - never plaintext, never in logs, profiles, exports, bundles or CLI args. Moving to another PC/user means re-entering it.
- Logs mask `password=`, `PASS`, `AUTH`, bearer tokens, etc. Payload logging can be switched off in Settings.
- No telemetry, no cloud dependency. No packet capture, exploitation, credential harvesting or process termination.

## 7. Architecture

```
src/NetDiagTool.Core        net8.0, no packages: services (Ping, Tcp, TcpMessage, Trace, PortMonitor, PortListener,
                            Csv, Logging, Reporting, Email/SMTP, Monitoring engine, History, Incidents, DNS,
                            NetworkInfo, Jobs, Scheduler, Bundle, Backup, SelfDiagnostics, CLI runner)
src/NetDiagTool.ViewModels  net8.0 MVVM view models (no WPF types - testable)
src/NetDiagTool             WPF GUI (views, tray, charts) + CLI entry
src/NetDiagCli              console front-end
tests/NetDiagTool.Tests     dependency-free test runner (CSV, config, ports, listeners, custom message,
                            SMTP via fake server, alert state machine, monitoring end-to-end, reports, CLI, jobs,
                            backup/restore, DNS parser, view-model smoke test)
```

Core has no UI dependency, so a future Windows Service / remote agent can reuse `AppServices` directly.
Monitoring history is stored as daily JSON-lines files in `Data\History` (embedded, no database server, no extra DLLs).
