# Windows Companion Guide for the Labs

This guide adapts the lab instructions for a typical Windows 11 development laptop using Docker Desktop, PowerShell, and WSL 2.

Use it beside the [cheat sheet](CHEAT_SHEET.md) and [code snippets](CODE_SNIPPETS.md).

## The six most important Windows differences

1. **Know which shell you are using.** Bash uses `\` for line continuation, PowerShell uses a backtick, and Command Prompt uses `^`. Do not mix their syntax.
2. **Docker Desktop normally uses the WSL 2 backend.** Install/update WSL, enable integration for your chosen Linux distribution, and run Linux containers for the lab images.
3. **Use `docker compose`, with a space.** Docker Desktop includes the Compose v2 plugin.
4. **Control line endings.** Windows CRLF can break shell scripts inside Linux containers. Use `.gitattributes` and repair affected files explicitly.
5. **Use the correct network name.** From a container, `localhost` refers to that container. Use `host.docker.internal` for the Windows host or a Compose service name for another container.
6. **Match filename capitalization.** Windows commonly uses a case-insensitive filesystem, while GitHub Actions/Linux is case-sensitive.

---

## Choose a working environment

You can perform most labs in either of these environments:

### Option A - PowerShell with Docker Desktop

Use Windows Terminal and open a PowerShell profile. This is convenient for Docker, Git, Node.js, browser tools, and Windows-native applications.

PowerShell prompt examples in this guide are labeled `powershell`.

### Option B - Ubuntu or another Linux distribution in WSL 2

Use this when lab instructions are written for Bash/Linux. Docker Desktop can expose its Docker engine to the WSL distribution through **Settings -> Resources -> WSL Integration**.

Bash examples in the main [code snippets](CODE_SNIPPETS.md) generally run most naturally inside WSL.

### Recommended approach

- Use **PowerShell** for Docker Desktop management and Windows-native tools.
- Use **WSL 2** for Bash scripts and Linux-oriented project workflows.
- Keep each repository primarily in one filesystem. Do not constantly alternate between `C:\...` and `/home/...` paths for the same working tree.
- For best WSL filesystem performance, keep Linux-heavy projects under the WSL home directory, such as `~/ssd-labs`.
- Use a Windows directory such as `C:\Users\YOUR_NAME\Documents\ssd-labs` when Windows applications must directly manipulate many project files.

---

## Install and verify WSL 2

Open **PowerShell as Administrator** for the installation step:

```powershell
wsl --install
```

Restart Windows if requested. Then use a normal PowerShell window:

```powershell
wsl --status
wsl --version
wsl --update
wsl --list --verbose
```

The chosen distribution should report version `2`. If an existing distribution uses version 1:

```powershell
wsl --set-version Ubuntu 2
```

Replace `Ubuntu` with the exact name shown by `wsl --list --verbose`.

### WSL path translation

| Windows | WSL |
|---|---|
| `C:\Users\Alice\project` | `/mnt/c/Users/Alice/project` |
| WSL home directory | `~` or `/home/USERNAME` |
| Open WSL files in Explorer | `explorer.exe .` from WSL |
| Access a WSL distribution in Explorer | `\\wsl$\DISTRIBUTION\home\USERNAME` |

Avoid manually modifying a distribution’s internal virtual-disk files from unsupported Windows locations.

---

## Install and verify Docker Desktop

Install Docker Desktop for Windows and enable the WSL 2 backend. During setup, use Linux containers for the images in these labs.

In PowerShell:

```powershell
docker version
docker compose version
docker info
```

In an integrated WSL distribution:

```bash
docker version
docker compose version
```

If the Docker client cannot contact the daemon:

1. Start Docker Desktop.
2. Wait until it reports that the engine is running.
3. Confirm **Use the WSL 2 based engine** is enabled.
4. Confirm integration is enabled for the selected WSL distribution.
5. Run `wsl --shutdown` in PowerShell and restart Docker Desktop if integration remains stale.

Do not install a second competing Docker engine inside WSL unless you intentionally know how it will be isolated and managed. For these labs, Docker Desktop integration is simpler.

---

## PowerShell, Command Prompt, and Bash translation

### Line continuation

PowerShell uses the backtick as the continuation character. It must be the final character on the line; trailing spaces break it.

```powershell
docker run --rm `
  --name example `
  nginx:alpine
```

Command Prompt uses `^`:

```bat
docker run --rm ^
  --name example ^
  nginx:alpine
```

Bash/WSL uses `\`:

```bash
docker run --rm \
  --name example \
  nginx:alpine
```

When copying from the main snippets, either run the Bash block in WSL or translate the whole command consistently to PowerShell.

### Common command translations

| Bash/Linux | PowerShell |
|---|---|
| `pwd` | `(Get-Location).Path` |
| `ls -la` | `Get-ChildItem -Force` |
| `mkdir -p reports` | `New-Item -ItemType Directory -Force reports` |
| `cp source dest` | `Copy-Item source dest` |
| `mv source dest` | `Move-Item source dest` |
| `rm file` | `Remove-Item file` |
| `export NAME=value` | `$env:NAME = "value"` |
| `unset NAME` | `Remove-Item Env:NAME` |
| `which docker` | `Get-Command docker` |
| `cat file` | `Get-Content file` |
| `curl URL` | `curl.exe URL` or `Invoke-WebRequest URL` |
| `open URL` | `Start-Process URL` |
| `ip addr` | `Get-NetIPAddress` |

Use `curl.exe` when you want familiar curl options such as `--fail` and `--show-error`. Windows PowerShell historically maps `curl` to `Invoke-WebRequest`, which has different parameters.

### Environment variables

PowerShell:

```powershell
$env:PROJECT_DIR = (Get-Location).Path
$env:SONAR_HOST_URL = "http://host.docker.internal:9000"
```

Remove a session variable:

```powershell
Remove-Item Env:SONAR_HOST_URL
```

Command Prompt:

```bat
set PROJECT_DIR=%CD%
```

WSL/Bash:

```bash
export PROJECT_DIR="$(pwd)"
```

---

## Windows line endings and executable scripts

### Why CRLF causes failures

Windows text files often use CRLF (`\r\n`). Linux shell scripts expect LF (`\n`). A script copied into a container may fail with errors such as:

```text
/bin/sh^M: bad interpreter
not found
syntax error: unexpected carriage return
```

### Add repository line-ending rules

Create `.gitattributes`:

```gitattributes
* text=auto
*.sh text eol=lf
*.bash text eol=lf
*.yml text eol=lf
*.yaml text eol=lf
Dockerfile text eol=lf
*.Dockerfile text eol=lf
*.bat text eol=crlf
*.cmd text eol=crlf
*.ps1 text eol=crlf
```

After agreeing with the team, normalize tracked files:

```powershell
git add --renormalize .
git status
```

Review the changes before committing.

### Repair the SecurityRAT script inside its Linux container

This avoids PowerShell encoding differences:

```powershell
docker exec -it securityrat-mariadb sh
```

Then inside the container:

```bash
cd /var
sed -i 's/\r$//' dumpRequirements.sh
chmod +x dumpRequirements.sh
./dumpRequirements.sh
```

Type `exit` to leave the container.

### Git executable-bit limitation

Windows filesystems do not represent the Unix executable bit in the same way. If a script must be executable in Linux:

```powershell
git update-index --chmod=+x path/to/script.sh
git diff --summary
```

---

## Docker Desktop networking on Windows

| Caller | Target | Address |
|---|---|---|
| Windows browser/PowerShell | Published container port | `localhost:HOST_PORT` |
| WSL shell | Published Docker Desktop port | Usually `localhost:HOST_PORT` |
| Container | Program running on Windows host | `host.docker.internal:PORT` |
| Compose service | Another service in same Compose project | `SERVICE_NAME:CONTAINER_PORT` |
| GitHub-hosted runner | Service on your laptop | Not directly reachable |

Examples:

```text
Windows browser -> Nginx:            http://localhost:8080
Selenium container -> Windows Node:  http://host.docker.internal:3000
App container -> MySQL service:      mysqldb:3306
Scanner container -> SonarQube:      http://host.docker.internal:9000
```

### Check a port in PowerShell

```powershell
Get-NetTCPConnection -LocalPort 80 -ErrorAction SilentlyContinue
Get-NetTCPConnection -LocalPort 3000 -ErrorAction SilentlyContinue
Get-NetTCPConnection -LocalPort 4444 -ErrorAction SilentlyContinue
Get-NetTCPConnection -LocalPort 9000 -ErrorAction SilentlyContinue
```

Identify the owning process:

```powershell
Get-Process -Id (Get-NetTCPConnection -LocalPort 3000).OwningProcess
```

Do not terminate a process until you know what it is and whether it is safe to stop. Changing a Compose host mapping from `80:80` to `8080:80` is often easiest.

### Windows Firewall

Publishing a port for `localhost` testing does not automatically mean it should be accessible from the network. If Windows asks whether to allow an application through the firewall, choose the narrowest network scope that satisfies the authorized lab.

Do not expose MySQL, Selenium, SonarQube, DVWA, ZAP, Burp, or an unauthenticated Git server to public networks.

---

## Windows volume paths

Compose files with relative paths are the least error-prone:

```yaml
services:
  app:
    volumes:
      - ./src:/usr/src/app
```

For a PowerShell one-off `docker run`, use the resolved current path:

```powershell
$ProjectDir = (Get-Location).Path
docker run --rm --volume "${ProjectDir}:/src" alpine ls -la /src
```

The braces prevent the colon after the variable from being parsed as part of the variable name.

If a mount is empty or denied:

1. Print `$ProjectDir` and confirm it is the intended folder.
2. Confirm the drive/path is accessible to Docker Desktop.
3. Avoid unusual system-protected directories.
4. Check whether antivirus or Controlled Folder Access blocked access.
5. Prefer a named volume for database data.

---

## Lab 1 on Windows

### Start the Compose stack

PowerShell:

```powershell
Set-Location C:\path\to\lab01
docker compose config
docker compose up -d
docker compose ps
docker compose logs --follow
```

Stop without removing persisted volumes:

```powershell
docker compose down
```

Do not add `--volumes` unless you intentionally want to remove named-volume data.

### Nginx

```powershell
curl.exe --fail --show-error http://localhost:8080/
Start-Process http://localhost:8080/
```

### MySQL

Named volumes avoid many host-permission and Linux-filesystem compatibility issues:

```yaml
services:
  mysqldb:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

Connect through the container:

```powershell
docker compose exec mysqldb mysql -u root -p
```

### Git server

The unauthenticated Git server from the lab is for isolated practice only. Clone it in PowerShell:

```powershell
git clone http://localhost:3000/repository.git
```

Do not allow the service through a public firewall profile.

### Selenium

Windows x64 commonly runs the lab Chrome image directly:

```powershell
docker run -d `
  --name selenium-server `
  --shm-size 2g `
  -p 4444:4444 `
  selenium/standalone-chrome

curl.exe --fail --show-error http://localhost:4444/wd/hub/status
```

Clean up:

```powershell
docker stop selenium-server
docker rm selenium-server
```

---

## Lab 2 on Windows

Clone over HTTPS unless GitHub SSH is already configured:

```powershell
git clone https://github.com/SecurityRAT/SecurityRAT-dockercompose.git
Set-Location SecurityRAT-dockercompose
docker compose up -d --remove-orphans
docker compose ps
Start-Process http://localhost:9002
```

If the requirements update script fails, use the CRLF repair procedure above.

Run the update:

```powershell
docker exec securityrat-mariadb sh -c '/var/dumpRequirements.sh'
```

Clean up:

```powershell
docker compose down
```

The lab’s default accounts are for local demonstration. Never expose them publicly.

---

## X03 and X04 on Windows

### Microsoft Threat Modeling Tool

Windows is the intended platform for X03. Install the Microsoft Threat Modeling Tool from the official source linked in the supplied lab PDF. Confirm the required .NET version and install updates before the exercise.

Suggested workflow:

1. Create a project-specific model file.
2. Draw external entities, processes, data stores, flows, and trust boundaries.
3. Label flows with protocol, data, and authentication.
4. Open the analysis view and review the generated STRIDE threats.
5. Record mitigation, justification, priority, owner, and status.
6. Export the full report and retain it with the corresponding application version.

Do not mark threats as mitigated merely because a control is planned.

### OWASP Threat Dragon

Threat Dragon is the cross-platform option for X04. Use the official Windows desktop release or authorized web deployment. If Windows SmartScreen warns about a download, confirm its official source and signature before allowing it. Do not bypass protection for an unverified package.

---

## X05 on Windows

### Local Nginx

```powershell
docker compose up -d nginx
docker compose exec nginx nginx -t
curl.exe --head http://localhost/
```

### Let’s Encrypt

Let’s Encrypt cannot issue an ordinary public certificate for `localhost`. HTTP-01 requires the public domain to resolve to a reachable server on port 80.

For the team project, issue and renew the certificate on the team VM or production-like server rather than a Windows laptop behind NAT. You can still validate the Nginx configuration locally.

Use the lab’s Certbot container or perform DNS-01 with narrowly scoped DNS credentials. Do not store DNS tokens, account keys, or TLS private keys in the repository.

### Scheduling

For an always-on Windows server, Task Scheduler can invoke a reviewed renewal script. A sleeping or powered-off laptop cannot perform renewal reliably. The preferred location remains the always-on server that terminates TLS.

### GitHub credentials

Use Git Credential Manager and GitHub’s secret store instead of embedding a token in a URL or workflow.

Check the configured credential helper:

```powershell
git config --show-origin --get credential.helper
```

---

## X06 on Windows

### Dependency-Check in PowerShell

```powershell
$ProjectDir = (Get-Location).Path
$ReportDir = Join-Path $ProjectDir "odc-reports"
$DataDir = Join-Path $ProjectDir "odc-data"

New-Item -ItemType Directory -Force $ReportDir | Out-Null
New-Item -ItemType Directory -Force $DataDir | Out-Null

docker run --rm `
  --volume "${ProjectDir}:/src:ro" `
  --volume "${ReportDir}:/report" `
  --volume "${DataDir}:/usr/share/dependency-check/data" `
  owasp/dependency-check `
  --project "CHANGE_ME_PROJECT" `
  --scan /src `
  --format HTML `
  --format JSON `
  --out /report
```

Keep `odc-data` so later scans can reuse downloaded vulnerability data. Add both directories to `.gitignore` unless reports are intentional submission evidence.

If the first scan takes a long time, inspect activity before stopping it; the initial data download can be substantial.

---

## X07 on Windows

### Node.js project

PowerShell:

```powershell
node --version
npm --version
npm ci
npm test
npm start
```

Open the application:

```powershell
Start-Process http://localhost:3000/
```

### Filename capitalization

Windows may resolve `Services.js` when code imports `services.js`, but Linux CI may fail. Match the exact case. For a case-only Git rename:

```powershell
git mv services.js temporary-name.js
git mv temporary-name.js Services.js
git status
```

### Selenium reaches a Node server on Windows

If Node runs directly in PowerShell and Selenium runs in Docker, use:

```text
http://host.docker.internal:3000/
```

If both run as Compose services, use the Node service name, such as `http://app:3000/`.

The Dockerized Selenium server avoids manually matching a locally installed ChromeDriver with the Windows Chrome version.

---

## X08 on Windows

ESLint runs the same through Node.js:

```powershell
npm ci
New-Item -ItemType Directory -Force reports | Out-Null
npx eslint .
npx eslint . --format=@microsoft/eslint-formatter-sarif --output-file=reports/eslint-results.sarif
```

Open the report in the default associated application:

```powershell
Invoke-Item reports\eslint-results.sarif
```

Use the project’s local ESLint through `npx`; do not rely on a different global version. Again, filename capitalization and CRLF behavior can differ from Linux GitHub Actions.

---

## X09 on Windows

### Start SonarQube

```powershell
docker compose -f sonarqube-compose.yml config
docker compose -f sonarqube-compose.yml up -d
docker compose -f sonarqube-compose.yml ps
docker compose -f sonarqube-compose.yml logs --follow sonarqube
```

Open the dashboard:

```powershell
Start-Process http://localhost:9000
```

Change the default administrator password and keep the service on a trusted local network.

If SonarQube exits because of memory pressure, increase Docker Desktop’s available resources and restart the stack. A container limit does not create more total memory.

### Scanner container reaching SonarQube

PowerShell:

```powershell
$ProjectDir = (Get-Location).Path
$SecureToken = Read-Host "SonarQube token" -AsSecureString
$TokenPointer = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($SecureToken)

try {
  $env:SONAR_TOKEN = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($TokenPointer)

  docker run --rm `
    --volume "${ProjectDir}:/usr/src:ro" `
    --workdir /usr/src `
    --env SONAR_HOST_URL="http://host.docker.internal:9000" `
    --env SONAR_TOKEN `
    sonarsource/sonar-scanner-cli
}
finally {
  [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($TokenPointer)
  Remove-Item Env:SONAR_TOKEN -ErrorAction SilentlyContinue
}
```

Avoid putting the token directly in the command because it can remain in shell history. A GitHub-hosted runner cannot reach SonarQube at `localhost:9000` on your laptop; use an appropriately secured reachable server or controlled self-hosted runner.

---

## X11a on Windows

Install the official Windows OWASP ZAP application. A desktop ZAP instance can scan an explicitly authorized local container target through a published port:

```text
http://localhost:8081/
```

If ZAP itself runs in Docker and the target runs directly on Windows, use:

```text
http://host.docker.internal:8081/
```

Windows Defender Firewall may prompt when ZAP opens a listener. Allow only the network scope required for the lab; loopback/local testing should not become public exposure.

ZAP sessions and reports can contain credentials, cookies, local paths, and response data. Do not commit them without sanitization.

---

## X11b on Windows

### Burp proxy

Burp commonly listens on:

```text
Host: 127.0.0.1
Port: 8080
```

Configure only the lab browser/profile to use that proxy. Using a separate Firefox profile avoids disrupting normal browsing.

If DVWA runs through a Docker published port, start with:

```text
http://localhost:PORT/
```

If browser proxy-bypass settings cause local traffic not to pass through Burp, use an appropriate local interface address and ensure the target remains authorized and non-public.

Find local IPv4 addresses:

```powershell
Get-NetIPAddress -AddressFamily IPv4 |
  Where-Object { $_.IPAddress -notlike '127.*' -and $_.AddressState -eq 'Preferred' }
```

After the exercise:

1. Turn Burp Intercept off.
2. Remove the browser’s manual proxy configuration.
3. Close or protect the Burp project.
4. Delete or sanitize captured credentials/session cookies.
5. Stop DVWA and other intentionally vulnerable services.

---

## Windows troubleshooting quick reference

### Docker Desktop is not ready

```powershell
docker info
wsl --status
wsl --list --verbose
```

Start Docker Desktop and wait. If WSL integration remains unresponsive:

```powershell
wsl --shutdown
```

Then restart Docker Desktop.

### Wrong container mode

The lab images are Linux containers. If Docker reports incompatible operating-system errors, switch Docker Desktop to Linux containers.

### Container cannot reach Windows program

Replace container-side `localhost` with:

```text
host.docker.internal
```

Also confirm the Windows program listens on an address that accepts the required connection and that the firewall policy permits only the authorized scope.

### Windows cannot reach a container

```powershell
docker compose ps
docker compose logs --tail 200 SERVICE_NAME
Get-NetTCPConnection -LocalPort HOST_PORT -ErrorAction SilentlyContinue
```

Confirm the service publishes `HOST_PORT:CONTAINER_PORT`.

### Script reports `^M` or bad interpreter

Check `.gitattributes`, convert the file to LF, and verify the executable bit. Prefer doing the repair inside WSL or the Linux container.

### Volume path is rejected

```powershell
$ProjectDir = (Get-Location).Path
$ProjectDir
docker compose config
```

Use `"${ProjectDir}:/container/path"` in PowerShell and confirm Docker Desktop can access the folder.

### Works locally but fails in GitHub Actions

Check:

- filename/import capitalization;
- CRLF versus LF;
- uncommitted files;
- Windows-only absolute paths;
- PowerShell commands copied into Bash steps or vice versa;
- missing Linux executable bit;
- `npm install` locally versus reproducible `npm ci` in CI;
- use of `host.docker.internal` where CI service networking needs `localhost` or a service name.

### Collect environment evidence

```powershell
Get-ComputerInfo | Select-Object WindowsProductName, WindowsVersion, OsArchitecture
wsl --version
wsl --list --verbose
docker version
docker compose version
git --version
node --version
npm --version
```

Do not include tokens, `.env` contents, private keys, cookies, or captured credentials in screenshots.
