# NetDiag Tool V1.0
A portable WPF/.NET 8 network diagnostic utility for authorized personal and in-house IT use.

## Build
Requires Windows 10/11 x64 and the .NET 8 SDK or Visual Studio 2022 Desktop Development workload.

```cmd
dotnet restore NetDiagTool\NetDiagTool.csproj
dotnet build NetDiagTool\NetDiagTool.csproj -c Release
dotnet publish NetDiagTool\NetDiagTool.csproj -c Release -r win-x64 --self-contained true -p:PublishSingleFile=true -p:PublishTrimmed=false
```
Output is under `NetDiagTool\bin\Release\net8.0-windows\win-x64\publish\NetDiagTool.exe`.

## CSV
Ping/Trace headers: `Name,Target`. TCP headers: `Name,Target,Port`. UTF-8 and quoted commas are supported. Duplicate rows are suppressed.

## Notes
TCP/Telnet performs TCP connection establishment only and is not an interactive Telnet client. Trace uses Windows `tracert.exe`. Port Monitor uses Windows IP Helper API; protected process paths show `Access Denied`. Run elevated only when authorized. Logs are newline-delimited JSON under `Logs`. V1.0 intentionally excludes scanning, packet capture, credentials, exploits, SMTP, scheduling, alerts, charts, and database history.

## Known limitations
This source package is intended for Windows build/validation. Port Monitor currently enumerates IPv4 TCP rows; UDP and IPv6 are extension points. The concise V1.0 UI does not parse localized `tracert` output into hop columns. Always test the published EXE on a clean Windows 10/11 x64 machine before internal release.