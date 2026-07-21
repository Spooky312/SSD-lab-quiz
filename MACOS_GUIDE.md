# macOS Companion Guide for the Labs

This guide adapts the lab instructions for your current environment:

```text
Operating system: macOS
Processor: Apple Silicon / arm64
Shell: zsh
Package manager: Homebrew under /opt/homebrew
Docker: Docker Desktop with Docker Compose v2
```

Use this beside the [cheat sheet](CHEAT_SHEET.md) and [code snippets](CODE_SNIPPETS.md).

## The five most important Mac differences

1. Use `docker compose`, with a space. The older `docker-compose` form may work, but Compose v2 is already installed.
2. Do **not** add `sudo` to Docker commands when using Docker Desktop. The Linux/team-VM instructions are different.
3. From a container, use `host.docker.internal` to reach a service running directly on your Mac. Inside Compose, use the other service’s name.
4. Prefer native `arm64` or multi-architecture images. Only force `linux/amd64` when an image has no ARM version; emulation is slower.
5. The usual macOS filesystem is case-insensitive, while GitHub Actions/Linux is case-sensitive. Match import filenames exactly.

---

## Initial Mac setup

### Confirm the machine and tools

```bash
uname -m
sw_vers
docker version
docker compose version
git --version
node --version
npm --version
```

Expected processor output on this Mac:

```text
arm64
```

If `docker version` cannot contact the daemon, open Docker Desktop from Applications and wait until it reports that the engine is running.

### Optional Homebrew installations

Only install what is missing:

```bash
brew update
brew install git node
brew install --cask docker
```

After installing Docker Desktop, launch the application once. Installing the command-line executable alone is not enough; the Docker engine runs through Docker Desktop’s background services/virtual machine.

### Recommended project location

Keep the labs somewhere under your user directory, for example:

```bash
mkdir -p "$HOME/Documents/ssd-labs"
cd "$HOME/Documents/ssd-labs"
```

Docker Desktop normally supports bind mounts under `/Users`. If Docker reports that a path is not shared or allowed, check Docker Desktop’s file-sharing settings and avoid protected system folders.

### zsh path setup

Homebrew on Apple Silicon normally uses `/opt/homebrew`. If a newly installed Homebrew command is not found, add Homebrew’s environment setup to `~/.zprofile` using the instruction printed by the Homebrew installer, then open a new Terminal window.

Do not overwrite `PATH` with a single directory. Preserve the existing value if you modify it.

---

## macOS command translation table

| Lab wording or Linux command | Use on this Mac | Reason |
|---|---|---|
| `sudo docker-compose up` | `docker compose up` | Docker Desktop does not normally need root. |
| `docker-compose down` | `docker compose down` | Compose v2 syntax. |
| `wsl --install` | Not applicable | WSL is Windows-only. |
| `apt install ...` | `brew install ...`, if required | Homebrew is the Mac package manager. |
| Container reaches host via `localhost` | `host.docker.internal` | Container `localhost` is the container itself. |
| Linux host networking | Prefer published ports or Compose DNS | Docker Desktop networking runs through a VM. |
| GNU `sed -i '...' file` on host | `sed -i '' '...' file` | BSD `sed` requires a backup suffix argument. |
| `ip addr` / `hostname -I` | `ipconfig getifaddr en0` | Common way to obtain the Wi-Fi address. |
| Open a URL from terminal | `open http://localhost:PORT` | macOS opens it in the default browser. |
| Copy to clipboard | `pbcopy` | Built-in macOS clipboard utility. |

### CRLF correction

The SecurityRAT lab contains a line-ending repair step. The character before `i` in the PDF may visually resemble a dash; it must be the ASCII hyphen-minus.

On the Mac host:

```bash
sed -i '' $'s/\r$//' dumpRequirements.sh
```

Inside a Linux container:

```bash
sed -i 's/\r$//' dumpRequirements.sh
```

---

## Apple Silicon container compatibility

### Check image architecture

Most official images used by the labs are multi-architecture. To inspect an image manifest:

```bash
docker buildx imagetools inspect IMAGE_NAME:TAG
```

After pulling an image:

```bash
docker image inspect IMAGE_NAME:TAG --format '{{.Architecture}}/{{.Os}}'
```

### When no ARM image exists

Only if Docker reports that no matching `linux/arm64` manifest exists, you can request x86 emulation:

```yaml
services:
  legacy-service:
    image: CHANGE_ME_IMAGE
    platform: linux/amd64
```

Or for a one-off command:

```bash
docker run --platform linux/amd64 CHANGE_ME_IMAGE
```

This is a compatibility fallback. It can start slowly and use more CPU/memory. Do not add `platform: linux/amd64` to every service.

### Selenium browser image on Apple Silicon

Google Chrome for Linux has historically created architecture limitations in ARM containers. If the lab’s image fails on Apple Silicon, use Selenium’s Chromium image:

```bash
docker run -d \
  --name selenium-server \
  --shm-size 2g \
  -p 4444:4444 \
  selenium/standalone-chromium
```

If the lecturer requires the exact `selenium/standalone-chrome` image, try it normally first. Use `--platform linux/amd64` only if the exact image lacks an ARM variant and note the use of emulation in your evidence.

---

## Docker Desktop networking on Mac

Choose the address based on where the caller and target are running:

| Caller | Target | Address to use |
|---|---|---|
| Mac browser/terminal | Published container port | `localhost:HOST_PORT` |
| Container | Program running directly on Mac | `host.docker.internal:PORT` |
| Compose service | Another service in same Compose network | `SERVICE_NAME:CONTAINER_PORT` |
| GitHub-hosted Actions runner | Service on your Mac | Not directly reachable |

Example:

```text
Mac browser -> Nginx container:       http://localhost:8080
Selenium container -> Mac Node app:   http://host.docker.internal:3000
App container -> MySQL service:       mysqldb:3306
Scanner container -> Mac SonarQube:   http://host.docker.internal:9000
```

### Find the Mac’s LAN address

For Wi-Fi, usually:

```bash
ipconfig getifaddr en0
```

For other interfaces:

```bash
networksetup -listallhardwareports
```

Do not assume the address is permanent. It may change when you reconnect to the network.

### Port conflicts

Check what is listening on a port:

```bash
lsof -nP -iTCP:80 -sTCP:LISTEN
lsof -nP -iTCP:3000 -sTCP:LISTEN
lsof -nP -iTCP:4444 -sTCP:LISTEN
lsof -nP -iTCP:9000 -sTCP:LISTEN
```

If port 80 is busy, use a different host-side mapping such as `8080:80`. The container still listens on 80.

---

## Lab 1 on macOS

### Docker Desktop

Docker Desktop already includes the Compose plugin. You do not need to install Docker Engine through Ubuntu instructions.

```bash
open -a Docker
docker version
docker compose version
```

Run the lab without `sudo`:

```bash
cd /path/to/lab01
docker compose config
docker compose up -d
docker compose ps
docker compose logs -f
```

Press `Control-C` only for a foreground `docker compose up`. For background services:

```bash
docker compose stop
docker compose down
```

Do not add `-v` to `down` unless you intentionally want to remove named-volume data.

### Nginx

```bash
curl --fail --show-error http://localhost:8080/
open http://localhost:8080/
```

You can publish port 80 directly through Docker Desktop, but `8080:80` is less likely to conflict with AirPlay receivers, local development servers, or other software.

### MySQL bind mounts

Named volumes are generally less troublesome on Docker Desktop:

```yaml
services:
  mysqldb:
    image: mysql:8.0
    volumes:
      - mysql_data:/var/lib/mysql

volumes:
  mysql_data:
```

If the lab requires `./mysql_data:/var/lib/mysql`, keep the directory beneath your user folder and exclude it from Git.

### Selenium

On Apple Silicon, use the ARM-compatible guidance above. Verify the status from the Mac:

```bash
curl --fail --show-error http://localhost:4444/wd/hub/status
```

---

## Lab 2 on macOS

Clone over HTTPS unless your GitHub SSH key is already configured:

```bash
git clone https://github.com/SecurityRAT/SecurityRAT-dockercompose.git
cd SecurityRAT-dockercompose
docker compose up -d --remove-orphans
open http://localhost:9002
```

If a container image has no ARM manifest, first look for a maintained multi-architecture tag. Use x86 emulation only as a fallback.

Update the database as shown in the snippets guide:

```bash
docker exec securityrat-mariadb sh -c '/var/dumpRequirements.sh'
```

Clean up without deleting persisted data:

```bash
docker compose down
```

---

## X03 and X04 on macOS

### Microsoft Threat Modeling Tool

The Microsoft Threat Modeling Tool lab requires Windows and .NET. It does not run natively on macOS.

Practical options:

1. Use the Windows environment provided by the school, if available.
2. Use a Windows virtual machine that meets licensing and processor requirements.
3. Use a teammate’s Windows machine for the Microsoft-tool-specific exercise.
4. Use OWASP Threat Dragon natively for your team model when the assignment permits it.

Do not assume an Intel Windows VM image will work unchanged on Apple Silicon. Apple Silicon virtual machines generally require an ARM-compatible guest and compatible application stack.

### OWASP Threat Dragon

Threat Dragon is the more Mac-friendly lab option. Download the correct macOS architecture release when offered. If macOS blocks an application obtained outside the App Store, confirm it came from the official release source before using **System Settings -> Privacy & Security -> Open Anyway**. Do not bypass Gatekeeper for unverified downloads.

---

## X05 on macOS

### Local Nginx testing

Docker Desktop can publish ports 80 and 443 without prefixing every Docker command with `sudo`:

```bash
docker compose up -d nginx
docker compose exec nginx nginx -t
curl --head http://localhost/
```

### Let’s Encrypt limitation

Let’s Encrypt cannot issue a normal public certificate for `localhost`. HTTP-01 also requires the public domain to resolve to a publicly reachable server on port 80.

For the team project, certificate issuance will normally belong on the team VM/server rather than a laptop behind home/campus NAT. You can still test Nginx configuration locally before deploying it.

If using DNS-01 from the Mac, protect DNS API credentials and prefer a narrowly scoped token. The lab’s manual DNS flow avoids placing a credential in automation but requires updating the TXT record interactively.

### Scheduled renewal

`cron` does not run reliably while a Mac is asleep. Production certificate renewal should run on the always-on team server. If you genuinely need a recurring Mac task, `launchd` is the native scheduler, but it still cannot run while the machine is off.

### GitHub credentials

Use GitHub secrets for CI values. On the Mac, Git can use the Keychain credential helper:

```bash
git config --global credential.helper osxkeychain
```

Never place a GitHub token directly in the remote URL, workflow YAML, shell history, or screenshots.

---

## X06 on macOS

The local Dependency-Check snippet uses `$(pwd)` and quoted volume paths, so it works even when the project is under a directory whose name contains spaces:

```bash
PROJECT_DIR="$(pwd)"
mkdir -p "$PROJECT_DIR/odc-reports" "$PROJECT_DIR/odc-data"

docker run --rm \
  --volume "$PROJECT_DIR:/src:ro" \
  --volume "$PROJECT_DIR/odc-reports:/report" \
  --volume "$PROJECT_DIR/odc-data:/usr/share/dependency-check/data" \
  owasp/dependency-check \
  --project "CHANGE_ME_PROJECT" \
  --scan /src \
  --format HTML \
  --out /report
```

Do not use the PDF’s `export pwd=...` example. `pwd` is already the familiar shell command name, and a task-specific uppercase variable such as `PROJECT_DIR` is clearer.

If the first scan appears stuck, inspect the logs; downloading vulnerability data can take significant time. Keep `odc-data` between runs.

---

## X07 on macOS

### Node and filename case

```bash
node --version
npm --version
npm ci
npm test
```

macOS may allow an import such as `../utils/Services` to resolve a file named `services.js`. GitHub Actions on Linux may reject it. Match every character’s case exactly and use Git to record case-only renames:

```bash
git mv services.js services.tmp.js
git mv services.tmp.js Services.js
```

### Selenium reaching a Node server on the Mac

If Node runs directly on macOS and Selenium runs in Docker:

```bash
npm start
```

Use this application address in the Selenium test:

```text
http://host.docker.internal:3000/
```

If Node and Selenium are both Compose services, use the Node service name instead, such as `http://app:3000/`.

### ChromeDriver alternative

The Dockerized Selenium server avoids manually matching the Mac Chrome version with a local ChromeDriver. If you install a driver locally, its architecture and version must match the browser and Node Selenium setup.

---

## X08 on macOS

ESLint itself is platform-independent. The important Mac-specific issue is preventing local case-insensitive resolution from hiding import errors that fail in Linux CI.

```bash
npm ci
mkdir -p reports
npx eslint .
npx eslint . \
  --format=@microsoft/eslint-formatter-sarif \
  --output-file=reports/eslint-results.sarif
```

Open the SARIF as text when debugging generation:

```bash
open -a TextEdit reports/eslint-results.sarif
```

Do not depend on a globally installed ESLint. `npx eslint` uses the project’s pinned development dependency.

---

## X09 on macOS

### Start SonarQube

Docker Desktop must have enough memory available for SonarQube and PostgreSQL. Start with the Compose file, then inspect actual behavior before increasing resources:

```bash
docker compose -f sonarqube-compose.yml up -d
docker compose -f sonarqube-compose.yml ps
docker compose -f sonarqube-compose.yml logs -f sonarqube
open http://localhost:9000
```

If SonarQube exits because of memory pressure, increase Docker Desktop’s resource allocation in its settings and restart the stack. A container memory limit does not manufacture additional memory for Docker Desktop.

### Scanner container reaching SonarQube

Do not copy the Linux `--network host` command blindly. On the Mac, point the scanner at the host-published SonarQube port:

```bash
PROJECT_DIR="$(pwd)"

docker run --rm \
  --volume "$PROJECT_DIR:/usr/src:ro" \
  --workdir /usr/src \
  --env SONAR_HOST_URL="http://host.docker.internal:9000" \
  --env SONAR_TOKEN \
  sonarsource/sonar-scanner-cli
```

Set `SONAR_TOKEN` in the current shell without placing it in the command itself:

```bash
read -s "SONAR_TOKEN?SonarQube token: "
echo
export SONAR_TOKEN
```

When finished:

```bash
unset SONAR_TOKEN
```

A GitHub-hosted runner cannot reach `localhost:9000` on your Mac. Use an appropriately secured and reachable server or a controlled self-hosted runner for the team integration.

---

## X11a on macOS

Install the official macOS ZAP desktop application and run the lab target locally. If the target is in Docker and published on port 8081, the desktop ZAP application can reach it at:

```text
http://localhost:8081/
```

If ZAP itself runs in Docker, use:

```text
http://host.docker.internal:8081/
```

Do not active-scan a real website merely because it is reachable from your Mac. Keep the target and authorization evidence explicit.

ZAP sessions and reports may contain cookies, credentials, local paths, and response data. Store them outside the Git repository or sanitize them before submission.

---

## X11b on macOS

### Burp proxy

Burp normally listens on loopback port 8080. Configure Firefox or the lab browser to use:

```text
HTTP proxy: 127.0.0.1
Port: 8080
```

If DVWA runs directly through a local Mac web stack, use the Mac’s LAN address when the lab requires traffic not to bypass the proxy:

```bash
ipconfig getifaddr en0
```

If DVWA runs in Docker with a published port, try `http://localhost:PORT` through the configured browser proxy. If routing or proxy-bypass rules interfere, use the Mac LAN address and published port.

After the exercise:

1. Turn Intercept off.
2. Remove the browser’s manual proxy settings.
3. Close or protect the Burp project/session.
4. Delete or sanitize captured credentials and cookies.
5. Stop the vulnerable local application when it is no longer needed.

---

## Mac troubleshooting quick reference

### Docker command exists but engine is unavailable

```bash
open -a Docker
docker info
```

Wait for Docker Desktop to finish starting.

### Wrong CPU architecture

Typical message:

```text
no matching manifest for linux/arm64
```

Look for an ARM/multi-architecture image first. If the lab requires the exact x86 image:

```bash
docker run --platform linux/amd64 IMAGE_NAME
```

### Container cannot reach Mac service

Replace container-side `localhost` with:

```text
host.docker.internal
```

Also confirm the Mac application listens on an address accessible beyond only a restricted loopback binding when required.

### Mac cannot reach container

```bash
docker compose ps
docker compose logs --tail=200 SERVICE_NAME
lsof -nP -iTCP:HOST_PORT -sTCP:LISTEN
```

Confirm the Compose service publishes `HOST_PORT:CONTAINER_PORT`.

### Bind mount is empty or denied

```bash
pwd
ls -la
docker compose config
```

Use an absolute path for diagnosis, confirm Docker Desktop file-sharing permission, and keep the project under `/Users/...`.

### A workflow passes locally but fails on GitHub Actions

Check:

- filename/import capitalization;
- uncommitted files;
- platform-specific paths;
- `npm install` versus reproducible `npm ci`;
- missing executable bit on scripts;
- reliance on Mac-only applications or Keychain;
- a service addressed through `host.docker.internal` even though CI networking is different.

### Show useful environment evidence

```bash
uname -m
sw_vers
docker version
docker compose version
git --version
node --version
npm --version
```

Do not include tokens, `.env` contents, private keys, or captured cookies in screenshots.
