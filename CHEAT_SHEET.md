# Secure Software Development Labs - Cheat Sheet

Use the [code snippets](CODE_SNIPPETS.md) for copy-ready templates, the [macOS companion guide](MACOS_GUIDE.md) for Apple Silicon-specific guidance, and the [Windows companion guide](WINDOWS_GUIDE.md) for Windows 11, PowerShell, and WSL 2.

## Universal lab workflow

1. **Understand scope** - define the application, data, users, trust boundaries, and systems you are authorized to test.
2. **Establish a baseline** - confirm the app starts and its normal behavior is known before adding a security tool.
3. **Run one control at a time** - build, unit test, dependency scan, static scan, dynamic scan, then fuzz as appropriate.
4. **Preserve evidence** - retain the tool version, configuration, timestamp, report, and workflow URL or screenshot.
5. **Triage findings** - decide whether each finding is a true positive, false positive, accepted risk, or needs investigation.
6. **Remediate and rerun** - a finding is not closed merely because code changed; the relevant test or scan must pass again.
7. **Prevent regression** - automate useful checks in CI and fail the pipeline according to an agreed quality gate.

### Evidence checklist

- Tool and version
- Target commit SHA
- Configuration used
- Exact authorized target and scope
- Start/end time
- Raw and human-readable report
- Finding severity, affected component, evidence, and remediation
- Rerun result after remediation
- Exceptions with owner, justification, and review/expiry date

### Secrets and repository hygiene

Never commit:

- `.env` files containing real values
- GitHub tokens or SonarQube tokens
- TLS private keys (`privkey.pem`)
- database data directories
- Burp/ZAP sessions containing credentials or captured traffic
- generated reports containing sensitive paths, tokens, or application data

Prefer GitHub repository/environment secrets, Docker secrets, or a proper secrets manager. Use placeholders in examples and rotate a credential immediately if it is exposed.

---

## Lab 1 - Docker and Docker Compose

### What you need to know

- An **image** is an immutable template; a **container** is a running instance.
- A Compose file defines **services**, **networks**, **volumes**, **configs**, and **secrets**.
- Compose normally creates a project-scoped default bridge network. Services on it reach each other using the **service name**, not `localhost`.
- `ports: ["HOST:CONTAINER"]` publishes a container port to the host.
- A bind mount such as `./mysql_data:/var/lib/mysql` maps a host path into a container. A named volume is managed by Docker.
- `docker compose stop` stops containers without removing them. `docker compose down` stops and removes the Compose containers and default network. Adding `-v` also removes named volumes and may destroy data.
- `depends_on` controls startup order, not application readiness. Use health checks when one service must wait for another to become usable.

### Core commands

| Goal | Command |
|---|---|
| Validate resolved Compose configuration | `docker compose config` |
| Build images | `docker compose build` |
| Start in foreground | `docker compose up` |
| Start in background | `docker compose up -d` |
| Show service state | `docker compose ps` |
| Follow logs | `docker compose logs -f` |
| Run a shell in a service | `docker compose exec SERVICE sh` |
| Stop | `docker compose stop` |
| Remove containers/network | `docker compose down` |
| List all containers | `docker ps -a` |

### Service-specific checks

**Nginx**

- Publish container port 80, then visit `http://localhost/`.
- If port 80 is already used, map `8080:80` and visit `http://localhost:8080/`.
- Validate configuration inside a container with `nginx -t`.

**MySQL**

- Persist `/var/lib/mysql`.
- Connect from another Compose service with host `mysqldb` and port `3306`.
- Connect inside the container with `mysql -u root -p`.
- Do not use the lab password in a real project. Put credentials in environment/secrets and restrict database exposure.

**Git server**

- A bare repository has no working tree and can be used as a remote.
- The unauthenticated HTTP server in the lab is for isolated practice only. Do not expose it publicly.
- A real deployment should use authenticated HTTPS or SSH keys, authorization, logging, backup, and a maintained SCM platform.

**Selenium**

- The standalone Chrome image exposes WebDriver on port 4444.
- Check readiness via `/wd/hub/status` (or `/status`, depending on server/version).
- Tests inside another container must use the Selenium service hostname; `localhost` would refer to the test container itself.

### Common failures

| Symptom | Likely cause | Fix |
|---|---|---|
| `port is already allocated` | Host port is occupied | Stop the conflicting service or change the host-side port. |
| Service name does not resolve | Services are not on the same network | Attach both to the same Compose network and use the service name. |
| MySQL continually restarts | Bad variables, permissions, incompatible existing data | Inspect logs; test with a fresh lab-only data directory if safe. |
| File permissions become awkward | Container UID/GID differs from host | Use a named volume or align user IDs/ownership. |
| Selenium connection refused | Server not ready or wrong hostname | Wait for health/readiness; use the Compose service name in containers. |

### Completion checklist

- [ ] `docker compose config` succeeds.
- [ ] Nginx responds over the expected host port.
- [ ] MySQL data survives a normal container recreation.
- [ ] The lab Git repository can be cloned in the isolated environment.
- [ ] Selenium status reports ready.
- [ ] No real secrets or generated database data are tracked by Git.

Source: [Lab 1 PDF](../sources/Lab01-DockerCompose-2026.pdf)

---

## Lab 2 - Secure Software Requirements

### Requirement hierarchy

- **Functional requirement** - what the system must do.
- **Non-functional requirement** - a measurable quality or constraint, such as performance, availability, privacy, or auditability.
- **Security requirement** - a testable statement that reduces an identified security risk.
- **Control** - the mechanism used to meet a requirement.
- **Verification criterion** - the evidence/test that proves the requirement is satisfied.

Avoid vague statements such as “the application must be secure.” Use measurable language:

> The system shall lock an account for 15 minutes after 5 failed authentication attempts within 10 minutes, record the event without logging the password, and alert the security operator.

### A useful requirement formula

> **The system shall** `[security behavior]` **for** `[asset/actor/action]` **under** `[condition]` **to achieve** `[security objective]`. **Verification:** `[test/evidence]`.

Add an ID, source/risk, priority, owner, status, and acceptance criteria so the requirement is traceable.

### CIA + AAA prompts

| Area | Identify | Typical risks | Possible requirements |
|---|---|---|---|
| Confidentiality | Sensitive data in transit, use, storage, logs, backups | Disclosure, weak access control, leaked secrets | TLS, encryption at rest, minimization, masking, role-limited reads |
| Integrity | Records, configuration, code, logs, transactions | Unauthorized modification, replay, injection | Server-side validation, authorization, signatures/hashes, versioning |
| Availability | Critical functions and dependencies | DoS, resource exhaustion, outages, data loss | Rate limits, timeouts, redundancy, backup/restore targets, monitoring |
| Authentication | Human users, admins, services | Credential stuffing, weak recovery, session theft | MFA for privileged users, secure reset, strong session handling |
| Authorization | Roles, resources, actions, ownership | IDOR/BOLA, privilege escalation, default allow | Deny by default, server-side checks, least privilege, role matrix |
| Accountability | Events needed to reconstruct actions | Log tampering, missing identity/time, repudiation | Central audit log, clock sync, restricted access, retention, alerting |

### Quality test for every requirement

Use **SMART-ish** requirements:

- Specific: names the subject, behavior, and protected asset.
- Measurable: has a threshold, time, algorithm class, role, or observable outcome.
- Achievable and relevant: maps to a real risk and system context.
- Testable: includes positive, negative, and abuse-case acceptance tests.
- Traceable: links requirement -> risk/threat -> design control -> test -> evidence.

### SecurityRAT workflow

1. Select the kind of software artifact being developed.
2. Let SecurityRAT propose applicable requirements.
3. Review applicability; do not accept requirements blindly.
4. Record the chosen handling and create issue-tracker tasks when action is needed.
5. Update compliance as the artifact and risks change.

Lab setup uses the SecurityRAT Docker Compose repository, starts it locally, refreshes its requirement database, and opens `http://localhost:9002`. The supplied lab lists default demonstration accounts; change or isolate them and never expose this lab configuration publicly.

### Common mistakes

- Confusing a control (“use JWT”) with a requirement (“only an authenticated user with role X may perform Y”).
- Writing untestable adjectives: secure, fast, strong, regularly, appropriate.
- Specifying an algorithm without key lifecycle, rotation, access, and failure behavior.
- Only covering happy paths; add misuse and negative authorization tests.
- Logging secrets or personal data in the name of accountability.
- No link between a threat and the requirement that mitigates it.

### Completion checklist

- [ ] 2-3 major functional and non-functional requirements are clear.
- [ ] CIA and AAA assets/risks/requirements are documented.
- [ ] Requirements are uniquely identified and testable.
- [ ] Each security requirement maps to a risk and verification method.
- [ ] SecurityRAT applicability decisions are saved or exported.
- [ ] Follow-up work exists as owned tickets.

Source: [Lab 2 PDF](../sources/Lab02%20-%20Secure%20Software%20Requirement%202026.pdf)

---

## X03 - Microsoft Threat Modeling Tool

### Data Flow Diagram vocabulary

- **External entity** - user or external system outside the application’s control.
- **Process** - transforms data or performs an operation.
- **Data store** - stores data beyond a single flow.
- **Data flow** - labeled movement of specific data between elements.
- **Trust boundary** - where identity, privilege, ownership, network zone, or policy changes.

Draw the architecture that actually exists. Label protocols, ports, data types, identities, and authentication. A boundary is not merely a decorative box; it must show a meaningful change in trust.

### STRIDE

| Threat | Security property violated | Ask | Common mitigations |
|---|---|---|---|
| Spoofing | Authentication | Can an attacker pretend to be this user/service? | MFA, strong service identity, certificate validation, secure sessions |
| Tampering | Integrity | Can data/code/configuration be changed undetected? | Authorization, validation, signatures/MACs, protected deployment |
| Repudiation | Accountability / non-repudiation | Can an actor deny an action? | Tamper-resistant audit trails, identity, timestamps, correlation IDs |
| Information disclosure | Confidentiality | Can data be read by an unauthorized party? | Encryption, minimization, access control, secret handling, safe errors |
| Denial of service | Availability | Can resources be exhausted or a dependency blocked? | Limits, quotas, timeouts, isolation, redundancy, graceful degradation |
| Elevation of privilege | Authorization | Can an actor gain capabilities they should not have? | Least privilege, deny by default, sandboxing, server-side checks |

### Analysis workflow

1. Define scope, assets, entry points, dependencies, assumptions, and excluded components.
2. Draw processes, entities, stores, flows, and trust boundaries.
3. Use the analysis view to generate candidate STRIDE threats.
4. Validate each candidate against the actual design.
5. Record attack scenario, affected asset, impact, priority, mitigation, owner, and evidence.
6. Set status deliberately: `Not Started`, `Needs Investigation`, `Mitigated`, or `Not Applicable` with justification.
7. Create the full report and review it with the team.

“Not Applicable” needs an architectural reason or existing guarantee, not merely “unlikely.” “Mitigated” needs a named implemented control and verification evidence.

### Completion checklist

- [ ] Every flow is directional and labeled with data/protocol.
- [ ] Every trust transition is explicit.
- [ ] Sensitive data stores and privileged processes are identified.
- [ ] Generated threats were reviewed, not accepted mechanically.
- [ ] Every threat has a status and rationale.
- [ ] Open threats have an owner and planned mitigation.
- [ ] Full report is exported and versioned with the design.

Source: [X03 PDF](../sources/X03%20-%20Microsoft%20Threat%20Modeling%20Tool_2026.pdf)

---

## X04 - OWASP Threat Dragon

Threat Dragon is the cross-platform/open-source option in the lab. It supports diagrams, automated threat generation, mitigation tracking, and reports.

### Fast workflow

1. Load a demo model if you need to learn the notation.
2. Create the model and add actors, processes, data stores, flows, and trust boundaries.
3. Select elements to edit properties and threats.
4. Connect a flow by selecting the source’s link tool and then the destination.
5. Review generated threats and add context-specific threats the rule engine cannot infer.
6. Add mitigations and mark genuinely resolved threats as mitigated.
7. Mark an element out of scope only with a written reason; threat generation is disabled for it.
8. Use red-highlighted elements to find open threats.
9. Generate a report and choose whether to include out-of-scope elements, mitigated threats, and diagrams.

### Threat-model quality checks

- Decompose components enough to expose different trust levels and technologies.
- Do not draw one generic “backend” if authentication, API, worker, and database have distinct risks.
- Include management interfaces, CI/CD, backups, telemetry, third parties, and administrative flows.
- Treat passwordless authentication as an architectural choice, not proof that phishing, recovery, session, enrollment, and device threats disappear.

Source: [X04 PDF](../sources/X04%20-%20OWASP%20Threat%20Dragon_2026.pdf)

---

## X05 - Web Proxy, TLS, and GitHub Actions

### Reverse proxy model

```text
Client --HTTPS:443--> Nginx --HTTP/internal network--> Application
                         |
                         +-- ACME challenge files / certificates
```

Nginx can terminate TLS, redirect HTTP to HTTPS, forward requests, add security headers, limit request size/rate, and centralize access logs. The upstream should not be unnecessarily published to the public host.

### TLS certificate flow

- **HTTP-01**: Certbot places a token beneath `/.well-known/acme-challenge/`; the CA retrieves it over public port 80.
- **DNS-01**: you publish a specific DNS TXT value. This works without exposing an HTTP challenge endpoint and supports wildcard certificates, but automation needs tightly scoped DNS credentials.
- Let’s Encrypt certificates are short-lived. Automate renewal and reload/restart Nginx only after successful renewal.
- Do not use `--force-renewal` in a daily scheduled job; unnecessary issuance can hit rate limits.

### GitHub Actions anatomy

```text
event trigger -> workflow -> job(s) -> ordered steps -> logs/artifacts/status
```

- Workflows live in `.github/workflows/*.yml`.
- Pin permissions to the minimum required.
- Treat pull-request code as untrusted. Never expose powerful secrets to untrusted fork workflows.
- Pin third-party actions to a trusted version or commit according to team policy.
- Use repository/environment secrets; redact and rotate anything accidentally logged.
- Separate build/test artifacts from deployment credentials and require environment approval for sensitive production deployments.

### Validation checklist

- [ ] DNS resolves to the intended host.
- [ ] Ports 80/443 and firewall/NAT are correctly configured.
- [ ] `nginx -t` passes before reload.
- [ ] HTTP redirects to HTTPS except the required ACME challenge path.
- [ ] Certificate hostname, chain, and expiry are correct.
- [ ] CI triggers on intended branches/events only.
- [ ] CI permissions and secrets are minimal.
- [ ] A deliberately failing test makes the workflow fail.

Source: [X05 PDF](../sources/X05-Github-Actions.pdf)

---

## X06 - Software Composition Analysis

### What SCA answers

- Which third-party components and versions are present?
- Which known vulnerabilities may affect them?
- What evidence, severity, and fix version are available?
- Are licenses or unsupported components a concern?

OWASP Dependency-Check uses dependency evidence and vulnerability data to identify likely known-vulnerable components. Results require triage: a version match may be a false positive, and absence of a finding does not prove a dependency is safe.

### Local scan flow

1. Pull the scanner image.
2. Mount source read-only where practical.
3. Persist the vulnerability-data cache so future scans are faster.
4. Write reports to a dedicated directory.
5. Review report evidence and affected dependencies.
6. Upgrade/remove/replace vulnerable dependencies, or document a time-bounded exception.
7. Rebuild, retest, and rescan.

### Dependency-Check vs Dependabot

| Tool | Primary job |
|---|---|
| Dependency-Check | Scans project dependencies for known vulnerabilities and emits reports. |
| Dependabot | Checks package ecosystems and proposes version-update pull requests. |

They complement each other. An update bot does not replace vulnerability analysis, and a scanner does not manage upgrades for you.

### Triage order

1. Confirm the component and version are actually used/shipped.
2. Read the vulnerability conditions, not just the score.
3. Check exploitability in your deployment and exposure.
4. Identify a safe fixed version and compatibility impact.
5. Upgrade and run regression/security tests.
6. Suppress only with precise evidence and an expiry/review date.

### CI checklist

- Cache vulnerability data appropriately.
- Upload human-readable and machine-readable reports even when policy fails the build.
- Agree on failure thresholds and exception handling.
- Protect any vulnerability-data API key as a secret.
- Pin the scanning action/image; avoid floating production dependencies such as `@main` where reproducibility matters.

Source: [X06 PDF](../sources/X06-Integrating%20OWASP%20Dependency%20Check.pdf)

---

## X07 - Automated Testing with GitHub Actions

### Testing pyramid

- **Unit tests**: many, fast, isolated tests of functions/classes.
- **Integration tests**: fewer tests of component boundaries, databases, services, or APIs.
- **UI/end-to-end tests**: few high-value journeys through a real browser; slower and more fragile.

Security needs negative tests as well as expected behavior: unauthorized access, invalid input, expired sessions, duplicate/replayed requests, boundary values, and dependency failure.

### Lab flow

1. Initialize the supplied Node project as a Git repository and commit the baseline.
2. Run `node src/server.js`; verify `http://localhost:3000/`.
3. Stop it and run Mocha through `npx mocha tests/test.js` or `npm test`.
4. Start standalone Chrome/Selenium and confirm it is ready on port 4444.
5. Run the supplied Selenium test locally.
6. Adapt the workflow in `.github/workflows/` for the project.
7. Push and inspect both build and test job details.

### CI service networking

- A job running directly on the GitHub runner usually reaches a service container through `localhost:<mapped-port>`.
- A job itself running in a container usually reaches a service by its service label and container port.
- Encode a health check or readiness loop; startup order alone is not readiness.

### Reliable-test rules

- Make the server importable without unintentionally starting multiple listeners.
- Close servers, browsers, database clients, and files after tests.
- Use explicit waits for observable UI states, not arbitrary sleeps.
- Isolate test data and make tests order-independent.
- Capture screenshots/logs on UI failure and upload them as artifacts.
- A flaky test is a defect: diagnose it rather than simply rerunning indefinitely.

### Completion checklist

- [ ] Local server and timestamp endpoint behave correctly.
- [ ] Unit tests fail when behavior is deliberately broken.
- [ ] Selenium reports ready before UI tests start.
- [ ] UI test targets the correct host for its execution environment.
- [ ] CI has deterministic dependency installation and test commands.
- [ ] Build/test jobs expose enough logs and failure evidence.

Source: [X07 PDF](../sources/X07%20-%20Github%20Actions%20with%20Automated%20Testing.pdf)

---

## X08 - Static Code Analysis

### Key terms

- **SAST** analyzes source/bytecode without attacking a running target.
- **Linting** focuses on correctness/style and can be extended with security rules.
- **SARIF** is a standard interchange format that code-scanning systems can ingest.
- **CodeQL** models code as data and queries it for vulnerability patterns.

### ESLint flat configuration

Modern `eslint.config.mjs` exports an array of configuration objects. Each can target file globs, choose a parser, define globals, load plugin objects, and enable rules.

The lab combines:

- React rules
- Jest and Testing Library rules for test files
- `eslint-plugin-security` for patterns such as dynamic `eval` and risky regular expressions
- `eslint-plugin-security-node` for Node-focused patterns
- `eslint-plugin-no-unsanitized` for dangerous DOM injection patterns
- Microsoft’s SARIF formatter for GitHub code scanning

### Recommended workflow

1. Install ESLint and the SARIF formatter.
2. Initialize and adapt the flat config to the actual language/framework/files.
3. Run locally with the default formatter.
4. Resolve configuration/parser failures before discussing security findings.
5. Add security plugins incrementally and test each with a small known-bad fixture.
6. Produce SARIF into a report directory.
7. Upload SARIF from CI with the required repository permission.
8. Triage results in the Security/Code scanning interface.

### Interpretation traps

- Static tools find patterns, not proof of exploitability.
- A clean scan does not cover business logic, runtime configuration, authentication flows, or all data flows.
- Security lint rules do not replace a security-focused analyzer such as CodeQL, Semgrep, or SonarQube.
- Do not disable a noisy rule globally before checking whether scoping or precise suppression is possible.
- A suppression should include reason, reviewer, and ideally an issue/reference.

### CI checklist

- Use deterministic installation (`npm ci` with a lockfile).
- Give the SARIF upload step `security-events: write` and otherwise minimal permissions.
- Ensure the report path exists even when findings cause a nonzero exit, or structure the workflow so upload still runs.
- Keep generated reports out of Git unless they are intentional evidence.
- Scan pull requests and the protected default branch according to repository policy.

Source: [X08 PDF](../sources/X08%20-%20Static%20Code%20Analysis.pdf)

---

## X09 - SonarQube

### Architecture

```text
Source + scanner configuration
          |
          v
     SonarScanner ----> SonarQube server ----> PostgreSQL
                              |
                              v
                   dashboard / quality gate
```

The scanner analyzes a checked-out project and sends results to the server. PostgreSQL persists server data. The project key identifies the analysis history.

### Lab flow

1. Start SonarQube and PostgreSQL with the supplied Compose file.
2. Open `http://localhost:9000` and change the default administrator password when prompted.
3. Create a local project, define its key/name, and use global settings as instructed.
4. Generate a project analysis token and store it securely.
5. Run SonarScanner locally or in Docker.
6. View issues, hotspots, measures, duplication, coverage, and quality-gate result.
7. For GitHub Actions, store token and reachable server URL as secrets.

### Networking rule that causes most failures

`localhost` means the current network namespace:

- From your host, `http://localhost:9000` can reach a published container port.
- From a scanner container, `localhost` points back to that scanner container.
- On Docker Desktop, a container may reach the host through `host.docker.internal`.
- On a shared Compose network, use the SonarQube service name.
- A GitHub-hosted runner cannot reach a SonarQube server on your laptop/private LAN unless you deliberately provide secure connectivity. A self-hosted runner is often the appropriate controlled design.

### Findings

- **Bug**: likely reliability/correctness problem.
- **Vulnerability**: security problem requiring review/remediation.
- **Security hotspot**: security-sensitive code requiring human review; it is not automatically a vulnerability.
- **Code smell**: maintainability concern.
- **Quality gate**: pass/fail policy over measures, ideally focused on new code.

### Operational cautions

- Do not expose the server with default credentials.
- Back up the database and configuration for a team deployment.
- Restrict tokens to the needed project/permissions and rotate them.
- If Docker runs out of memory, increase Docker’s available memory or configure service limits deliberately; blindly adding a container limit does not create host memory.

Source: [X09 PDF](../sources/X09%20-%20SonarQube.pdf)

---

## X11a - Vulnerability Assessment with OWASP ZAP

> Only scan targets you own or have explicit permission to test. Define the exact URL, environment, time window, accounts, excluded endpoints, and rate limits.

### Scan types

- **Traditional spider**: parses links from responses; fast, but may miss JavaScript-generated navigation.
- **AJAX spider**: drives a browser and can discover dynamic routes; slower and needs browser/headless setup.
- **Passive scan**: analyzes proxied/crawled traffic without sending attack payloads.
- **Active scan**: sends attack requests and can alter data or affect availability; use only in an authorized test environment.

### Lab flow

1. Install ZAP and an intentionally vulnerable local target such as the lab’s DVWA setup.
2. Verify the target normally before scanning.
3. Choose whether to persist the ZAP session; protect it if it contains sensitive traffic.
4. Select Automated Scan and enter only the authorized target URL.
5. Crawl with the traditional or AJAX spider as appropriate.
6. Allow passive scanning to complete, then run active scanning only if authorized.
7. Review Alerts and inspect request, response, evidence, confidence, and remediation.
8. Generate an HTML or other suitable report.
9. Validate high-risk findings manually, fix, and rescan.

### Triage checklist

- Is the alert within the defined scope?
- Can the exact request be reproduced safely?
- Is the evidence in the response conclusive?
- What prerequisite and user privilege are needed?
- What data or function is affected?
- Is it a duplicate/root-cause variant?
- What is the least disruptive corrective control?
- Does the rescan show closure without introducing regression?

Source: [X11a PDF](../sources/X11a%20-%20Vulnerability%20Assessment%20with%20OWASP%20ZAP.pdf)

---

## X11b - Fuzzing with Burp Suite

> Only fuzz a target when explicit authorization includes automated high-volume requests. Use a local training application for this lab.

### Proxy and Intruder flow

1. Start the local target and Burp Community Edition (default proxy commonly uses `127.0.0.1:8080`).
2. Configure the browser proxy. The lab notes that the target URL may need the machine’s IP instead of `localhost` so interception routes correctly.
3. Submit a harmless login request and intercept it in Burp.
4. Send the request to Intruder.
5. In **Positions**, clear automatic markers and mark only the authorized parameters.
6. Choose an attack type; the lab uses **Cluster bomb** for username/password combinations.
7. Load the approved payload set for each position.
8. Start with a small list and conservative resource settings.
9. Compare status code, length, redirect location, timing, and selected response text.
10. Use Grep Extract to expose a useful discriminator such as the `Location` header, then sort results.
11. Manually verify a candidate result and turn the browser proxy off when finished.

### Intruder attack types

| Type | Behavior | Typical use |
|---|---|---|
| Sniper | One payload set, changes one position at a time | Isolate which parameter reacts to malformed input |
| Battering ram | Same payload placed into all positions together | Repeated token/value across fields |
| Pitchfork | Multiple lists advance in parallel | Paired values such as corresponding usernames/passwords |
| Cluster bomb | Cartesian product of payload lists | All combinations; request count grows rapidly |

For list sizes `m` and `n`, Cluster bomb makes roughly `m x n` requests, while Pitchfork makes roughly `min(m, n)`. Estimate the load before starting.

### Ethical and analytical cautions

- Never use leaked credential lists or production accounts.
- Avoid destructive endpoints and account lockout unless explicitly part of the test.
- A different response length or redirect is a lead, not proof; verify safely.
- Record request count and rate, because fuzzing can become a denial-of-service test.
- Remove captured credentials/session tokens from screenshots and reports.
- Restore browser proxy settings after the lab.

Source: [X11b PDF](../sources/X11b%20-%20Fuzzing%20with%20Burp%20Suite.pdf)

---

## Cross-lab secure delivery pipeline

| Stage | Lab technique | Useful gate/evidence |
|---|---|---|
| Requirements | CIA/AAA requirements, SecurityRAT | Approved, testable requirements with traceability |
| Design | DFD + STRIDE in Microsoft TMT/Threat Dragon | Threat report; open risks owned |
| Build | Docker Compose and GitHub Actions | Reproducible build; minimal permissions |
| Dependencies | OWASP Dependency-Check, Dependabot | SCA report; reviewed upgrades/exceptions |
| Code | ESLint security plugins, CodeQL, SonarQube | SARIF/dashboard; quality gate |
| Test | Unit + integration + Selenium | Passing tests and retained failure evidence |
| Verification | ZAP and Burp | Authorized test scope, report, remediation, rescan |

The core traceability chain is:

```text
Asset -> Risk/Threat -> Security Requirement -> Design Control
      -> Implementation -> Automated Test/Scan -> Evidence -> Residual Risk
```
