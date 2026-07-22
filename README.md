# Github Page Link:
https://spooky312.github.io/SSD-lab-quiz/

# ICT2216 / ICT2516C Lab Reference

Looking for lecture theory and its relationship to the labs? Open the [Lecture Cheat Sheet](lectures.html).

This folder turns the supplied lab PDFs into searchable references:

- [Lecture Cheat Sheet](lectures.html) - Lectures 1-P1 through 9, quick-recall notes, and direct links from theory to the relevant labs.
- [CHEAT_SHEET.md](CHEAT_SHEET.md) - concepts, procedures, checklists, deliverables, and troubleshooting by lab.
- [CODE_SNIPPETS.md](CODE_SNIPPETS.md) - copy-ready commands and configuration templates by lab.
- [MACOS_GUIDE.md](MACOS_GUIDE.md) - Apple Silicon/macOS setup, command differences, networking, and lab-by-lab notes.
- [WINDOWS_GUIDE.md](WINDOWS_GUIDE.md) - Windows 11, PowerShell, WSL 2, Docker Desktop, and lab-by-lab notes.

> **Your environment:** Apple Silicon (`arm64`) Mac with Docker Desktop and Compose v2. Use the macOS guide alongside the other two references.

## Fast navigation

| Lab | Topic | Cheat sheet | Snippets |
|---|---|---|---|
| Lab 1 | Docker Compose, Nginx, MySQL, Git, Selenium | [Open](CHEAT_SHEET.md#lab-1---docker-and-docker-compose) | [Open](CODE_SNIPPETS.md#lab-1---docker-compose-nginx-mysql-git-and-selenium) |
| Lab 2 | Secure software requirements and SecurityRAT | [Open](CHEAT_SHEET.md#lab-2---secure-software-requirements) | [Open](CODE_SNIPPETS.md#lab-2---securityrat-and-requirement-templates) |
| X03 | Microsoft Threat Modeling Tool and STRIDE | [Open](CHEAT_SHEET.md#x03---microsoft-threat-modeling-tool) | [Open](CODE_SNIPPETS.md#x03-and-x04---threat-model-records) |
| X04 | OWASP Threat Dragon | [Open](CHEAT_SHEET.md#x04---owasp-threat-dragon) | [Open](CODE_SNIPPETS.md#x03-and-x04---threat-model-records) |
| X05 | Nginx reverse proxy, TLS, GitHub Actions | [Open](CHEAT_SHEET.md#x05---web-proxy-tls-and-github-actions) | [Open](CODE_SNIPPETS.md#x05---nginx-reverse-proxy-tls-and-basic-ci) |
| X06 | OWASP Dependency-Check and Dependabot | [Open](CHEAT_SHEET.md#x06---software-composition-analysis) | [Open](CODE_SNIPPETS.md#x06---owasp-dependency-check-and-dependabot) |
| X07 | Unit, integration, and UI testing | [Open](CHEAT_SHEET.md#x07---automated-testing-with-github-actions) | [Open](CODE_SNIPPETS.md#x07---node-unit-tests-selenium-and-github-actions) |
| X08 | ESLint, SARIF, security plugins, CodeQL | [Open](CHEAT_SHEET.md#x08---static-code-analysis) | [Open](CODE_SNIPPETS.md#x08---eslint-sarif-security-plugins-and-codeql) |
| X09 | SonarQube and SonarScanner | [Open](CHEAT_SHEET.md#x09---sonarqube) | [Open](CODE_SNIPPETS.md#x09---sonarqube-and-sonarscanner) |
| X11a | OWASP ZAP vulnerability assessment | [Open](CHEAT_SHEET.md#x11a---vulnerability-assessment-with-owasp-zap) | [Open](CODE_SNIPPETS.md#x11a---owasp-zap-baseline-scan) |
| X11b | Burp Suite Intruder fuzzing | [Open](CHEAT_SHEET.md#x11b---fuzzing-with-burp-suite) | [Open](CODE_SNIPPETS.md#x11b---burp-intruder-test-record) |

## How to use these files

1. Read the relevant cheat-sheet section before a lab.
2. Check the matching section in the [macOS guide](MACOS_GUIDE.md) or [Windows guide](WINDOWS_GUIDE.md).
3. Copy only the snippet you need and replace every `CHANGE_ME` value.
4. Compare configuration with the files supplied by the lecturer; those files remain authoritative for grading.
5. Record evidence as you work: commands, workflow runs, screenshots, reports, findings, fixes, and rerun results.
6. Never scan, fuzz, or attack a system unless you have explicit authorization.

## Scope and conventions

- The content is derived from the [local course PDFs](../sources/).
- Commands use the modern `docker compose` form. If your installation only supports the older standalone client, substitute `docker-compose`.
- Templates improve a few formatting or safety issues in the PDFs, such as YAML indentation, ASCII hyphens, portable paths, and avoiding hard-coded secrets.
- Tool interfaces and third-party actions change. If a lab-provided template conflicts with a snippet, adapt the snippet to the required lab version.
- Generated reports, tokens, passwords, database files, and TLS private keys should not be committed.

## Suggested evidence folder

```text
evidence/
├── lab01/
├── lab02/
├── x03-threat-model/
├── x04-threat-dragon/
├── x05-ci-tls/
├── x06-sca/
├── x07-tests/
├── x08-sast/
├── x09-sonarqube/
├── x11a-zap/
└── x11b-burp/
```

For each lab, keep a short `README.md` containing the date, environment, exact command, expected result, actual result, evidence link, and any remediation performed.
