# Secure Software Development Labs - Code Snippets

These are adaptable templates, not blind copy/paste answers. Replace every `CHANGE_ME` value, keep secrets out of Git, and compare each template with the lecturer-provided files required by your lab.

> Commands use `docker compose`. Substitute `docker-compose` only if your environment uses the older standalone client.

> On your Apple Silicon Mac, also check the [macOS companion guide](MACOS_GUIDE.md). In particular, do not use `sudo` with Docker Desktop, use `host.docker.internal` when a container must reach the Mac host, and prefer native ARM images.

> On Windows, use the [Windows companion guide](WINDOWS_GUIDE.md) to translate Bash commands into PowerShell, configure WSL 2, and avoid CRLF and volume-path problems.

## Quick command index

```bash
# Validate, start, inspect, stop
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs -f
docker compose down

# Git baseline
git init
git add .
git commit -m "Add initial files"

# Node baseline
npm ci
npm test

# Never print secrets to logs; set them in the platform's secret store.
```

---

## Lab 1 - Docker Compose, Nginx, MySQL, Git, and Selenium

### Minimal Nginx Compose file

`compose.yaml`:

```yaml
name: ssd-lab01

services:
  nginxwebsvr:
    image: nginx:alpine
    container_name: nginxwebsvr
    ports:
      - "8080:80"
    restart: unless-stopped
```

```bash
docker compose config
docker compose up -d
curl --fail --show-error http://localhost:8080/
docker compose ps
docker compose logs nginxwebsvr
docker compose down
```

### Nginx + MySQL with a health check

Create a local `.env` for lab use and do not commit it:

```dotenv
MYSQL_ROOT_PASSWORD=CHANGE_ME_ROOT_PASSWORD
MYSQL_DATABASE=testdb
MYSQL_USER=appuser
MYSQL_PASSWORD=CHANGE_ME_APP_PASSWORD
```

`compose.yaml`:

```yaml
name: ssd-lab01

services:
  nginxwebsvr:
    image: nginx:alpine
    ports:
      - "8080:80"
    restart: unless-stopped
    networks:
      - frontend

  mysqldb:
    image: mysql:8.0
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: "${MYSQL_ROOT_PASSWORD:?set in .env}"
      MYSQL_DATABASE: "${MYSQL_DATABASE:-testdb}"
      MYSQL_USER: "${MYSQL_USER:-appuser}"
      MYSQL_PASSWORD: "${MYSQL_PASSWORD:?set in .env}"
    volumes:
      - mysql_data:/var/lib/mysql
    networks:
      - backend
    healthcheck:
      test: ["CMD-SHELL", "mysqladmin ping -h localhost -u root -p$$MYSQL_ROOT_PASSWORD --silent"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s

networks:
  frontend:
  backend:

volumes:
  mysql_data:
```

```bash
docker compose up -d
docker compose ps
docker compose logs -f mysqldb
docker compose exec mysqldb mysql -u root -p
```

Inside the MySQL prompt:

```sql
SHOW DATABASES;
USE testdb;
SELECT CURRENT_USER(), VERSION();
EXIT;
```

### Lab-faithful bind mount option

The PDF uses a host directory instead of a named volume:

```bash
mkdir -p ./mysql_data
```

```yaml
services:
  mysqldb:
    image: mysql:8.0
    volumes:
      - ./mysql_data:/var/lib/mysql
```

Add `mysql_data/` to `.gitignore`.

### Unauthenticated lab Git server

> Use only in an isolated lab environment. It has no authentication or authorization.

`gitserver.Dockerfile`:

```dockerfile
FROM node:alpine

RUN apk add --no-cache tini git \
    && yarn global add git-http-server \
    && adduser -D -g git git

USER git
WORKDIR /home/git
RUN git init --bare repository.git

EXPOSE 3000
ENTRYPOINT ["tini", "--", "git-http-server", "-p", "3000", "/home/git"]
```

Compose service:

```yaml
services:
  git-server:
    build:
      context: .
      dockerfile: gitserver.Dockerfile
    restart: unless-stopped
    ports:
      - "3000:3000"
```

```bash
docker compose build git-server
docker compose up -d git-server
git clone http://localhost:3000/repository.git
```

The lab PDF also shows a `./repos:/var/www/git` mount, but the shown server serves `/home/git`; a mount must target the directory actually served if persistence is required.

### Selenium standalone service

`compose-selenium.yaml`:

```yaml
name: ssd-selenium

services:
  selenium:
    image: selenium/standalone-chrome
    shm_size: 2gb
    ports:
      - "4444:4444"
      - "7900:7900"
    restart: unless-stopped
    healthcheck:
      test: ["CMD-SHELL", "curl --fail http://localhost:4444/wd/hub/status || exit 1"]
      interval: 10s
      timeout: 5s
      retries: 12
      start_period: 20s
```

```bash
docker compose -f compose-selenium.yaml up -d
curl --fail --show-error http://localhost:4444/wd/hub/status
docker compose -f compose-selenium.yaml down
```

### Useful Docker diagnostics

```bash
docker compose config
docker compose ps
docker compose logs --tail=200 SERVICE_NAME
docker inspect CONTAINER_NAME
docker network ls
docker volume ls
docker stats --no-stream
```

---

## Lab 2 - SecurityRAT and requirement templates

### SecurityRAT lab commands

```bash
git clone https://github.com/SecurityRAT/SecurityRAT-dockercompose.git
cd SecurityRAT-dockercompose
docker compose up -d --remove-orphans
docker compose ps
docker compose logs --tail=200
```

Refresh the requirements database using the command expected by the supplied repository/lab:

```bash
docker exec securityrat-mariadb sh -c '/var/dumpRequirements.sh'
```

If a checked-out script has Windows CRLF line endings:

```bash
docker exec -it securityrat-mariadb sh
cd /var
sed -i 's/\r$//' dumpRequirements.sh
./dumpRequirements.sh
exit
```

Open `http://localhost:9002`, use the lab’s local demonstration account, then clean up:

```bash
docker compose down
```

### Security requirement record

```markdown
## SR-AUTH-001 - Privileged-user multi-factor authentication

- **Statement:** The system shall require multi-factor authentication before granting an administrator session.
- **Asset:** Administrative functions and configuration.
- **Risk/threat:** Credential theft leading to elevation of privilege.
- **Actors:** Administrator, identity provider.
- **Preconditions:** The account is active and assigned the administrator role.
- **Control:** Standards-based MFA through the approved identity provider.
- **Failure behavior:** Access is denied; no privileged session is created; a security event is recorded.
- **Acceptance tests:**
  1. Valid password without the second factor cannot create an administrator session.
  2. Valid factors create a session with the administrator role.
  3. Replayed/expired factors are rejected.
  4. The audit event identifies the account, time, outcome, and correlation ID without recording credentials.
- **Verification evidence:** Automated authentication tests and reviewed audit record.
- **Priority:** High
- **Owner:** CHANGE_ME
- **Status:** Proposed
```

### Authorization matrix

```markdown
| Resource/action | Anonymous | User | Resource owner | Administrator |
|---|---:|---:|---:|---:|
| Read public item | Allow | Allow | Allow | Allow |
| Read private item | Deny | Deny | Allow | Allow with audit |
| Update private item | Deny | Deny | Allow | Allow with audit |
| Delete user account | Deny | Deny | Own account only | Allow with audit |
```

### Requirement traceability matrix

```markdown
| Requirement ID | Asset | Threat/risk | Design control | Implementation | Test ID | Evidence | Status |
|---|---|---|---|---|---|---|---|
| SR-AUTH-001 | Admin functions | Credential theft | MFA | Auth service | AT-AUTH-001 | CI run/report | Open |
```

---

## X03 and X04 - Threat model records

### System inventory

```markdown
| ID | Type | Name | Trust zone | Technology | Identity/privilege | Sensitive data |
|---|---|---|---|---|---|---|
| E1 | External entity | Customer | Internet | Browser | Anonymous/user | Credentials, profile |
| P1 | Process | Web API | Application network | Node.js | App service account | Session, request data |
| D1 | Data store | Main database | Data network | PostgreSQL | DB application role | User and transaction data |
```

### Data-flow inventory

```markdown
| Flow ID | Source -> destination | Direction | Protocol/port | Authentication | Data | Crosses boundary? |
|---|---|---|---|---|---|---|
| F1 | Customer -> Web API | Inbound | HTTPS/443 | Session/OIDC | Login and app requests | Yes |
| F2 | Web API -> Database | Outbound | TLS/5432 | DB role | Parameterized queries/results | Yes |
```

### Threat record

```markdown
## T-001 - Attacker accesses another user's record

- **Element/flow:** P1 Web API / F2 database query
- **STRIDE:** Elevation of privilege; information disclosure
- **Attack scenario:** An authenticated user changes a record identifier and the server does not verify ownership.
- **Preconditions:** Valid low-privilege account and knowledge/guessability of another identifier.
- **Affected asset:** Private user records.
- **Impact:** Unauthorized disclosure or modification.
- **Likelihood:** CHANGE_ME
- **Priority:** High
- **Mitigation:** Server-side object-level authorization on every read/write; deny by default; indirect identifiers are defense-in-depth only.
- **Verification:** Negative integration tests for another user's ID and administrator audit behavior.
- **Owner:** CHANGE_ME
- **Status:** Not Started
- **Residual risk / notes:** CHANGE_ME
```

### STRIDE review checklist

```markdown
- [ ] Spoofing: user, service, webhook, and administrator identities
- [ ] Tampering: requests, messages, files, code, configuration, logs, and backups
- [ ] Repudiation: audit identity, time, source, outcome, integrity, and retention
- [ ] Information disclosure: transport, storage, logs, errors, caches, backups, metadata
- [ ] Denial of service: CPU, memory, storage, connection pools, queues, dependencies
- [ ] Elevation of privilege: roles, object ownership, service accounts, admin paths
```

---

## X05 - Nginx reverse proxy, TLS, and basic CI

### Reverse-proxy Compose file

Project layout:

```text
nginx-proxy/
├── compose.yaml
├── nginx.conf
├── html/
│   └── index.html
├── certbot-www/
└── letsencrypt/
```

`compose.yaml`:

```yaml
name: ssd-proxy

services:
  nginx:
    image: nginx:alpine
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf:ro
      - ./html:/usr/share/nginx/html:ro
      - ./certbot-www:/var/www/certbot:ro
      - ./letsencrypt:/etc/letsencrypt:ro

  certbot:
    image: certbot/certbot
    volumes:
      - ./certbot-www:/var/www/certbot
      - ./letsencrypt:/etc/letsencrypt
```

### HTTP configuration for initial ACME challenge

`nginx.conf` before the first certificate is issued:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name CHANGE_ME.example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }
}
```

```bash
docker compose up -d nginx
docker compose exec nginx nginx -t
curl --fail --show-error http://CHANGE_ME.example.com/
```

Request the certificate only after DNS and inbound port 80 work:

```bash
docker compose run --rm certbot certonly \
  --webroot \
  --webroot-path /var/www/certbot \
  --email CHANGE_ME@example.com \
  --agree-tos \
  --no-eff-email \
  -d CHANGE_ME.example.com
```

### HTTPS configuration after certificate issuance

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name CHANGE_ME.example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        return 301 https://$host$request_uri;
    }
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name CHANGE_ME.example.com;

    ssl_certificate /etc/letsencrypt/live/CHANGE_ME.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/CHANGE_ME.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;

    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    location / {
        proxy_pass http://CHANGE_ME_APP_SERVICE:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Validate before reloading:

```bash
docker compose exec nginx nginx -t
docker compose exec nginx nginx -s reload
curl --fail --show-error --head https://CHANGE_ME.example.com/
```

Renew and reload only on success:

```bash
docker compose run --rm certbot renew && \
docker compose exec nginx nginx -s reload
```

Example cron entry (confirm absolute paths and operational ownership first):

```cron
0 3 * * * cd /absolute/path/to/nginx-proxy && docker compose run --rm certbot renew --quiet && docker compose exec -T nginx nginx -s reload
```

### Minimal GitHub Actions Node CI

`.github/workflows/ci.yml`:

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

concurrency:
  group: ci-${{ github.ref }}
  cancel-in-progress: true

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 15
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test
```

Pin versions/commits according to the course or team policy.

---

## X06 - OWASP Dependency-Check and Dependabot

### Local Docker scan

```bash
docker pull owasp/dependency-check

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
  --format JSON \
  --out /report
```

The cached data directory accelerates later scans. Add generated directories to `.gitignore` unless reports are intentional evidence:

```gitignore
odc-data/
odc-reports/
```

### GitHub Actions Dependency-Check template

`.github/workflows/dependency-check.yml`:

```yaml
name: OWASP Dependency-Check

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  security-scan:
    runs-on: ubuntu-latest
    timeout-minutes: 30
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Run OWASP Dependency-Check
        uses: dependency-check/Dependency-Check_Action@main
        with:
          project: CHANGE_ME_PROJECT
          path: .
          format: HTML
          out: dependency-check-report

      - name: Upload report
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: dependency-check-report
          path: dependency-check-report
          if-no-files-found: error
```

The lab uses the action’s `main` branch. For a stable project, replace it with a reviewed release tag or full commit SHA supported by your policy.

### Dependabot

`.github/dependabot.yml`:

```yaml
version: 2
updates:
  - package-ecosystem: npm
    directory: /
    schedule:
      interval: weekly
    open-pull-requests-limit: 5
    labels:
      - dependencies
      - security
```

For other projects, change `package-ecosystem` to the relevant supported value and add one entry per manifest directory.

### Vulnerability exception record

```markdown
## SCA-EX-001

- **Component/version:** CHANGE_ME
- **Finding/CVE:** CHANGE_ME
- **Why it is present:** Direct / transitive dependency
- **Reachability/exposure evidence:** CHANGE_ME
- **Compensating controls:** CHANGE_ME
- **Upgrade/blocker:** CHANGE_ME
- **Owner:** CHANGE_ME
- **Approved by:** CHANGE_ME
- **Review/expiry date:** YYYY-MM-DD
- **Tracking issue:** CHANGE_ME
```

---

## X07 - Node unit tests, Selenium, and GitHub Actions

### Express server based on the lab screenshot

`src/server.js`:

```javascript
import { fileURLToPath } from 'node:url';
import express from 'express';

export function getCurrentTimestamp() {
  return new Date().toISOString();
}

export function createApp() {
  const app = express();

  app.get('/timestamp', (_req, res) => {
    res.json({ timestamp: getCurrentTimestamp() });
  });

  app.get('/', (_req, res) => {
    res.type('html').send(`<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>Browser Info</title>
  </head>
  <body>
    <h1>Browser and Timestamp Info</h1>
    <p id="browser-info">Loading browser details...</p>
    <p id="timestamp">Fetching server timestamp...</p>
    <script>
      document.getElementById('browser-info').textContent =
        'Your browser: ' + navigator.userAgent;

      fetch('/timestamp')
        .then((response) => response.json())
        .then((data) => {
          document.getElementById('timestamp').textContent =
            'Server timestamp: ' + data.timestamp;
        })
        .catch(() => {
          document.getElementById('timestamp').textContent =
            'Error fetching timestamp';
        });
    </script>
  </body>
</html>`);
  });

  return app;
}

export function startServer(port = Number(process.env.PORT ?? 3000)) {
  return createApp().listen(port, '0.0.0.0', () => {
    console.log(`Server is running on port ${port}`);
  });
}

if (process.argv[1] && fileURLToPath(import.meta.url) === process.argv[1]) {
  startServer();
}
```

The `createApp`/`startServer` separation lets tests import the application without always opening port 3000.

### Mocha unit tests

`tests/test.js`:

```javascript
import assert from 'node:assert/strict';
import { getCurrentTimestamp } from '../src/server.js';

describe('Timestamp Function', () => {
  it('should return a valid ISO timestamp', () => {
    const value = getCurrentTimestamp();
    assert.equal(new Date(value).toISOString(), value);
  });

  it('should return the current timestamp', () => {
    const before = Date.now();
    const value = Date.parse(getCurrentTimestamp());
    const after = Date.now();
    assert.ok(value >= before && value <= after);
  });
});
```

`package.json` essentials:

```json
{
  "type": "module",
  "scripts": {
    "start": "node src/server.js",
    "test": "mocha \"tests/**/*.js\""
  },
  "dependencies": {
    "express": "CHANGE_ME"
  },
  "devDependencies": {
    "mocha": "CHANGE_ME",
    "selenium-webdriver": "CHANGE_ME"
  }
}
```

Install versions appropriate to the supplied project, commit the lockfile, then:

```bash
npm ci
npm test
npm start
```

### Selenium UI test

`tests/SeleniumTest.mjs`:

```javascript
import assert from 'node:assert/strict';
import { Builder, By, until } from 'selenium-webdriver';

const seleniumUrl = process.env.SELENIUM_URL ?? 'http://localhost:4444/wd/hub';
const appUrl = process.env.APP_URL ?? 'http://host.docker.internal:3000/';

const driver = await new Builder()
  .usingServer(seleniumUrl)
  .forBrowser('chrome')
  .build();

try {
  await driver.get(appUrl);
  const heading = await driver.wait(until.elementLocated(By.css('h1')), 10_000);
  assert.equal(await heading.getText(), 'Browser and Timestamp Info');

  const timestamp = await driver.findElement(By.id('timestamp'));
  await driver.wait(async () => {
    return (await timestamp.getText()).startsWith('Server timestamp:');
  }, 10_000);
} finally {
  await driver.quit();
}
```

Host networking differs by platform. If both application and Selenium are Compose services, use the application service name in `APP_URL`.

### Start Selenium locally

```bash
docker run -d \
  --name selenium-server \
  --shm-size 2g \
  -p 4444:4444 \
  selenium/standalone-chrome

curl --fail --show-error http://localhost:4444/wd/hub/status
node tests/SeleniumTest.mjs
docker stop selenium-server
docker rm selenium-server
```

### GitHub Actions with a Selenium service

`.github/workflows/selenium-tests.yml`:

```yaml
name: Unit and UI tests

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 20
    services:
      selenium:
        image: selenium/standalone-chrome
        ports:
          - 4444:4444
        options: >-
          --shm-size=2g
          --health-cmd="curl --fail http://localhost:4444/wd/hub/status || exit 1"
          --health-interval=10s
          --health-timeout=5s
          --health-retries=12

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm test

      - name: Start application
        run: |
          npm start > app.log 2>&1 &
          for attempt in {1..30}; do
            if curl --fail --silent http://localhost:3000/timestamp >/dev/null; then
              exit 0
            fi
            sleep 1
          done
          exit 1

      - name: Run UI test
        env:
          SELENIUM_URL: http://localhost:4444/wd/hub
          APP_URL: http://localhost:3000/
        run: node tests/SeleniumTest.mjs

      - name: Upload application log on failure
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: app-log
          path: app.log
          if-no-files-found: ignore
```

---

## X08 - ESLint, SARIF, security plugins, and CodeQL

### Install packages

```bash
npm install --save-dev \
  eslint \
  @microsoft/eslint-formatter-sarif \
  @babel/eslint-parser \
  @babel/preset-react \
  eslint-plugin-react \
  eslint-plugin-jest \
  eslint-plugin-testing-library \
  eslint-plugin-security \
  eslint-plugin-security-node \
  eslint-plugin-no-unsanitized
```

### Flat ESLint configuration

`eslint.config.mjs`:

```javascript
import { defineConfig } from 'eslint/config';
import babelParser from '@babel/eslint-parser';
import reactPlugin from 'eslint-plugin-react';
import jestPlugin from 'eslint-plugin-jest';
import testingLibraryPlugin from 'eslint-plugin-testing-library';
import securityPlugin from 'eslint-plugin-security';
import securityNodePlugin from 'eslint-plugin-security-node';
import noUnsanitizedPlugin from 'eslint-plugin-no-unsanitized';

export default defineConfig([
  {
    ignores: [
      'node_modules/**',
      'build/**',
      'dist/**',
      'coverage/**',
      'reports/**',
    ],
  },
  {
    files: ['**/*.{js,jsx,mjs}'],
    languageOptions: {
      parser: babelParser,
      parserOptions: {
        requireConfigFile: false,
        babelOptions: { presets: ['@babel/preset-react'] },
      },
    },
    plugins: {
      react: reactPlugin,
      security: securityPlugin,
      'security-node': securityNodePlugin,
      'no-unsanitized': noUnsanitizedPlugin,
    },
    rules: {
      ...reactPlugin.configs.recommended.rules,
      ...securityPlugin.configs.recommended.rules,
      ...noUnsanitizedPlugin.configs.recommended.rules,
      'react/react-in-jsx-scope': 'off',
      'security/detect-eval-with-expression': 'error',
      'security-node/detect-crlf': 'error',
    },
    settings: {
      react: { version: 'detect' },
    },
  },
  {
    files: ['**/*.{test,spec}.{js,jsx}'],
    plugins: {
      jest: jestPlugin,
      'testing-library': testingLibraryPlugin,
    },
    rules: {
      ...jestPlugin.configs.recommended.rules,
      ...testingLibraryPlugin.configs.react.rules,
    },
    languageOptions: {
      globals: {
        afterEach: false,
        beforeEach: false,
        describe: false,
        expect: false,
        it: false,
        jest: false,
        test: false,
      },
    },
  },
]);
```

Plugin exports and rule compatibility can differ by installed major version. If a spread fails, inspect that plugin’s installed documentation and enable only supported rules.

### Local analysis and SARIF

```bash
mkdir -p reports
npx eslint .
npx eslint . \
  --format=@microsoft/eslint-formatter-sarif \
  --output-file=reports/eslint-results.sarif
```

Known-bad fixture for verifying the rule is active (do not ship this code):

```javascript
const expression = '1 + 1';
eval(`console.log(${expression})`);
```

```bash
npx eslint --debug test.js
```

### ESLint SARIF GitHub workflow

`.github/workflows/eslint.yml`:

```yaml
name: ESLint security scan

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read
  security-events: write

jobs:
  eslint:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Create report directory
        run: mkdir -p reports

      - name: Run ESLint and produce SARIF
        run: >-
          npx eslint .
          --format=@microsoft/eslint-formatter-sarif
          --output-file=reports/eslint-results.sarif

      - name: Upload SARIF
        if: always() && hashFiles('reports/eslint-results.sarif') != ''
        uses: github/codeql-action/upload-sarif@v3
        with:
          sarif_file: reports/eslint-results.sarif
```

Some pull-request contexts restrict `security-events: write`; align triggers with repository policy.

### CodeQL workflow starter

`.github/workflows/codeql.yml`:

```yaml
name: CodeQL

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: "23 3 * * 1"

permissions:
  contents: read
  security-events: write

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript-typescript

      - name: Analyze
        uses: github/codeql-action/analyze@v3
```

Adapt languages and build mode to the repository.

---

## X09 - SonarQube and SonarScanner

### SonarQube + PostgreSQL Compose starter

Create a local `.env` that is excluded from Git:

```dotenv
SONAR_DB_USER=sonar
SONAR_DB_PASSWORD=CHANGE_ME_DATABASE_PASSWORD
SONAR_DB_NAME=sonar
```

`sonarqube-compose.yml`:

```yaml
name: ssd-sonarqube

services:
  sonarqube:
    image: sonarqube:community
    depends_on:
      db:
        condition: service_healthy
    environment:
      SONAR_JDBC_URL: jdbc:postgresql://db:5432/${SONAR_DB_NAME:-sonar}
      SONAR_JDBC_USERNAME: ${SONAR_DB_USER:-sonar}
      SONAR_JDBC_PASSWORD: ${SONAR_DB_PASSWORD:?set in .env}
    ports:
      - "9000:9000"
    volumes:
      - sonar_data:/opt/sonarqube/data
      - sonar_extensions:/opt/sonarqube/extensions
      - sonar_logs:/opt/sonarqube/logs
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: ${SONAR_DB_USER:-sonar}
      POSTGRES_PASSWORD: ${SONAR_DB_PASSWORD:?set in .env}
      POSTGRES_DB: ${SONAR_DB_NAME:-sonar}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${SONAR_DB_USER:-sonar} -d ${SONAR_DB_NAME:-sonar}"]
      interval: 10s
      timeout: 5s
      retries: 10
    restart: unless-stopped

volumes:
  sonar_data:
  sonar_extensions:
  sonar_logs:
  postgres_data:
```

```bash
docker compose -f sonarqube-compose.yml config
docker compose -f sonarqube-compose.yml up -d
docker compose -f sonarqube-compose.yml ps
docker compose -f sonarqube-compose.yml logs -f sonarqube
```

Open `http://localhost:9000`. The lab PDF states the initial credentials are `admin` / `admin`; change them immediately and do not expose the lab server publicly.

### SonarScanner configuration

`sonar-project.properties`:

```properties
sonar.projectKey=CHANGE_ME_PROJECT_KEY
sonar.projectName=CHANGE_ME_PROJECT_NAME
sonar.sources=src
sonar.tests=tests
sonar.sourceEncoding=UTF-8
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

Run a scanner container on macOS/Windows Docker Desktop when SonarQube is published on the host:

```bash
PROJECT_DIR="$(pwd)"

docker run --rm \
  --volume "$PROJECT_DIR:/usr/src:ro" \
  --workdir /usr/src \
  --env SONAR_HOST_URL="http://host.docker.internal:9000" \
  --env SONAR_TOKEN="CHANGE_ME_OR_INJECT_SECURELY" \
  sonarsource/sonar-scanner-cli
```

On Linux, either add an appropriate host-gateway mapping or put the scanner and server on the same user-defined network and address the server by service/container name. Avoid putting a real token in shell history.

Lab-style explicit properties:

```bash
docker run --rm \
  --network host \
  --volume "$(pwd):/usr/src:ro" \
  sonarsource/sonar-scanner-cli \
  -Dsonar.projectKey=CHANGE_ME_PROJECT_KEY \
  -Dsonar.sources=/usr/src \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=CHANGE_ME_TOKEN
```

The host-network form is primarily applicable to Linux; Docker Desktop networking differs.

### GitHub Actions scanner starter

Store `SONAR_TOKEN` and `SONAR_HOST_URL` as GitHub secrets. The server URL must actually be reachable from the selected runner.

```yaml
name: SonarQube analysis

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read

jobs:
  sonar:
    runs-on: self-hosted
    steps:
      - name: Check out repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Scan
        uses: SonarSource/sonarqube-scan-action@v5
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}
```

Choose the runner and action version according to the team’s supported setup.

### Diagnostics

```bash
curl --fail --show-error http://localhost:9000/api/system/status
docker compose -f sonarqube-compose.yml logs --tail=300 sonarqube
docker compose -f sonarqube-compose.yml logs --tail=200 db
docker stats --no-stream
```

---

## X11a - OWASP ZAP baseline scan

The supplied lab teaches the ZAP desktop Automated Scan. The following command is useful for a passive-oriented CI baseline against an explicitly authorized test target.

```bash
mkdir -p zap-reports

docker run --rm \
  --volume "$(pwd)/zap-reports:/zap/wrk/:rw" \
  ghcr.io/zaproxy/zaproxy:stable \
  zap-baseline.py \
  -t "http://CHANGE_ME_AUTHORIZED_TARGET" \
  -r zap-baseline.html \
  -J zap-baseline.json
```

Do not substitute an Internet target unless written authorization explicitly permits it. An active full scan has more impact and must be separately approved.

### Finding record

```markdown
## ZAP-001 - CHANGE_ME alert name

- **Authorized target:** CHANGE_ME
- **URL/parameter:** CHANGE_ME
- **Risk/confidence:** CHANGE_ME
- **Request evidence:** Sanitized reference or attachment
- **Response evidence:** Sanitized reference or attachment
- **Reproduction:** CHANGE_ME
- **Impact and prerequisites:** CHANGE_ME
- **Root cause:** CHANGE_ME
- **Remediation:** CHANGE_ME
- **Owner:** CHANGE_ME
- **Retest date/result:** CHANGE_ME
- **Status:** Open / False positive / Accepted / Fixed
```

### Scope record

```markdown
- **Written authorization:** CHANGE_ME
- **In-scope hosts/URLs:** CHANGE_ME
- **Out-of-scope paths/functions:** logout, destructive operations, CHANGE_ME
- **Test accounts:** CHANGE_ME (credentials stored separately)
- **Permitted scan types:** passive / spider / AJAX spider / active
- **Rate/time restrictions:** CHANGE_ME
- **Emergency contact and stop condition:** CHANGE_ME
```

---

## X11b - Burp Intruder test record

Burp Intruder is configured through the UI in this lab, so the most reusable “snippet” is a precise experiment record.

```markdown
## FUZZ-001 - Login input variation

- **Authorization/scope reference:** CHANGE_ME
- **Target:** Local DVWA / CHANGE_ME
- **Captured request:** Sanitized attachment
- **Payload positions:** `username`, `password`
- **Attack type:** Cluster bomb
- **Payload set 1:** Name, source, count, hash
- **Payload set 2:** Name, source, count, hash
- **Expected request count:** set1_count x set2_count
- **Rate/resource settings:** CHANGE_ME
- **Success/failure discriminator:** `Location` response header
- **Stop conditions:** Availability impact, unexpected data change, lockout, CHANGE_ME
- **Candidate result:** CHANGE_ME
- **Manual verification:** CHANGE_ME
- **Finding/remediation link:** CHANGE_ME
```

### HTTP login request shape for understanding positions

```http
POST /login.php HTTP/1.1
Host: CHANGE_ME_LOCAL_TARGET
Content-Type: application/x-www-form-urlencoded
Cookie: CHANGE_ME_REDACTED_SESSION

username=FUZZ_POSITION_1&password=FUZZ_POSITION_2&Login=Login
```

Do not paste real session cookies into notes or version control.

### Response comparison table

```markdown
| Request # | Username case | Password case | Status | Length | Location | Time | Candidate? |
|---:|---|---|---:|---:|---|---:|---|
| 1 | baseline-invalid | baseline-invalid | 302 | CHANGE_ME | login.php | CHANGE_ME | No |
| 2 | candidate | candidate | 302 | CHANGE_ME | index.php | CHANGE_ME | Verify safely |
```

---

## Repository hygiene snippets

### `.gitignore` starter

```gitignore
# Secrets and local configuration
.env
.env.*
!.env.example
*.pem
*.key

# Dependencies and builds
node_modules/
dist/
build/
coverage/

# Lab data and generated reports
mysql_data/
odc-data/
odc-reports/
reports/
zap-reports/

# Scanner/proxy sessions
*.session
*.burp
```

### Safe `.env.example`

```dotenv
MYSQL_ROOT_PASSWORD=CHANGE_ME
MYSQL_DATABASE=testdb
MYSQL_USER=appuser
MYSQL_PASSWORD=CHANGE_ME
SONAR_TOKEN=CHANGE_ME
SONAR_HOST_URL=http://localhost:9000
```

### Evidence README template

```markdown
# Lab evidence - CHANGE_ME

- **Date/time/timezone:** CHANGE_ME
- **Team/operator:** CHANGE_ME
- **Commit SHA:** CHANGE_ME
- **Environment:** OS, Docker/tool versions
- **Scope/authorization:** CHANGE_ME
- **Configuration:** Links to sanitized files
- **Command/workflow:** CHANGE_ME
- **Expected result:** CHANGE_ME
- **Actual result:** CHANGE_ME
- **Raw report:** CHANGE_ME
- **Findings and triage:** CHANGE_ME
- **Remediation:** CHANGE_ME
- **Rerun result:** CHANGE_ME
- **Known limitations/residual risk:** CHANGE_ME
```
