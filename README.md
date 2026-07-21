# SSD Lab Quiz

A comprehensive, searchable reference for the ICT2216 / ICT2516C Secure Software Development labs. SSD Lab Quiz consolidates lab procedures, security concepts, reusable commands, configuration templates, troubleshooting guidance, and platform-specific instructions into one self-contained website.

**Website:** [https://spooky312.github.io/SSD-lab-quiz/](https://spooky312.github.io/SSD-lab-quiz/)

The generated page works locally as a single HTML file and can be deployed automatically with GitHub Pages.

> **Publication notice:** The original course PDFs are labelled “SIT Internal” and are intentionally excluded from this repository by `.gitignore`. Confirm that you have permission to publish the derived reference content before making the repository or Pages site public.

## What is included

The reference covers the complete supplied lab sequence:

| Lab | Main topics |
|---|---|
| Lab 1 | Docker, Docker Compose, Nginx, MySQL, a lab Git server, and Selenium |
| Lab 2 | Secure software requirements, CIA, AAA, traceability, and SecurityRAT |
| X03 | Microsoft Threat Modeling Tool, data-flow diagrams, and STRIDE |
| X04 | OWASP Threat Dragon and threat-model reporting |
| X05 | Nginx reverse proxying, TLS, Certbot, and GitHub Actions |
| X06 | OWASP Dependency-Check, Dependabot, SCA, and vulnerability triage |
| X07 | Node.js unit tests, Mocha, Selenium, integration tests, and CI |
| X08 | ESLint, security plugins, SARIF, GitHub Code Scanning, and CodeQL |
| X09 | SonarQube, PostgreSQL, SonarScanner, networking, and quality gates |
| X11a | Authorized vulnerability assessment with OWASP ZAP |
| X11b | Authorized fuzz testing with Burp Suite Intruder |

It also includes dedicated platform guides for:

- macOS and Apple Silicon
- Windows 11 and Docker Desktop
- PowerShell, Command Prompt, and WSL 2
- Cross-platform Docker networking and filesystem differences

## Website features

- One self-contained `index.html`
- Overview, Cheat Sheet, Code Snippets, macOS Guide, and Windows Guide tabs
- Searchable topic navigation
- Responsive desktop and mobile layouts
- Light and dark themes
- Copy buttons for every code block
- Accessible headings and keyboard-friendly navigation
- Print-friendly styling
- No external JavaScript, CSS, fonts, images, analytics, cookies, or runtime APIs
- No web framework or server required after generation

## Repository structure

```text
.
├── .github/
│   └── workflows/
│       └── deploy-pages.yml       # Builds and publishes the site
├── lab-guides/
│   ├── README.md                  # Source-document index
│   ├── CHEAT_SHEET.md             # Concepts, procedures, and checklists
│   ├── CODE_SNIPPETS.md           # Commands and configuration templates
│   ├── MACOS_GUIDE.md             # macOS and Apple Silicon guidance
│   ├── WINDOWS_GUIDE.md           # Windows 11, PowerShell, and WSL guidance
│   ├── build-html.mjs             # Converts all Markdown into the website
│   ├── SSD_LAB_REFERENCE.html     # Local generated version with source links
│   └── github-pages-site/
│       ├── .nojekyll
│       └── index.html             # Public deployment artifact
├── sources/                       # Local internal PDFs; ignored by Git
├── .gitignore
├── package.json
├── package-lock.json
└── README.md
```

The `sources/` directory is local reference material. It must remain read-only and is deliberately excluded from version control and deployment.

## Quick start

### Open the existing page

No installation is needed merely to read the guide. Open either generated file in a browser:

- `lab-guides/github-pages-site/index.html` - public-safe GitHub Pages version
- `lab-guides/SSD_LAB_REFERENCE.html` - local version that can link to supplied PDFs

### Regenerate the page

Requirements:

- Node.js 20 or newer
- npm

Install the pinned build dependency:

```bash
npm ci
```

Build both HTML variants:

```bash
npm run build
```

The build produces:

```text
lab-guides/SSD_LAB_REFERENCE.html
lab-guides/github-pages-site/index.html
```

The public version contains every guide section but converts links to excluded internal PDFs into non-clickable source labels, preventing broken or accidentally published course-material links.

## Editing content

The Markdown files are the source of truth:

| File | Purpose |
|---|---|
| `lab-guides/README.md` | Reference index and navigation |
| `lab-guides/CHEAT_SHEET.md` | Concepts, workflows, completion lists, and troubleshooting |
| `lab-guides/CODE_SNIPPETS.md` | Copy-ready commands, YAML, code, and record templates |
| `lab-guides/MACOS_GUIDE.md` | Mac and Apple Silicon-specific instructions |
| `lab-guides/WINDOWS_GUIDE.md` | Windows, PowerShell, and WSL 2 instructions |

After changing any source Markdown:

```bash
npm run build
```

Commit both the Markdown source changes and the regenerated HTML. This keeps the repository readable on GitHub while ensuring the deployed site matches its source.

## Deploying with GitHub Pages

The repository includes `.github/workflows/deploy-pages.yml`. It performs the following steps:

1. Checks out the repository.
2. Installs the pinned Markdown build dependency.
3. Generates the HTML website.
4. Packages `lab-guides/github-pages-site/` as the Pages artifact.
5. Deploys the artifact to the `github-pages` environment.

### First deployment

1. Create a GitHub repository.
2. Push these repository files to the `main` branch.
3. Open **Settings -> Pages** in the GitHub repository.
4. Under **Build and deployment**, choose **GitHub Actions** as the source.
5. Open the **Actions** tab and monitor the “Build and deploy lab reference” workflow.
6. When it completes, the deployment URL appears in the workflow and under **Settings -> Pages**.

The workflow also supports manual deployment through **Actions -> Build and deploy lab reference -> Run workflow**.

### Repository and site address

The repository belongs under the `spooky312` GitHub account with the name `SSD-lab-quiz`:

```text
https://github.com/spooky312/SSD-lab-quiz
```

Its GitHub Pages project address is:

```text
https://spooky312.github.io/SSD-lab-quiz/
```

This is GitHub’s standard project-site URL, not a separately registered custom domain. It requires no DNS records and no `CNAME` file.

### Updating the hosted site

Edit the Markdown, regenerate locally if desired, and push to `main`. The workflow rebuilds and deploys the page automatically.

## Git setup example

If this directory is not yet a Git repository:

```bash
git init
git branch -M main
git add .
git commit -m "Add secure software development lab reference"
git remote add origin https://github.com/spooky312/SSD-lab-quiz.git
git push -u origin main
```

Before committing, always check what will be included:

```bash
git status
git diff --cached
```

Confirm that `sources/`, secrets, generated scanner data, database directories, and captured proxy sessions are not staged.

## Security and privacy

This repository contains educational security-testing guidance. Use it responsibly:

- Only scan, fuzz, intercept, or attack systems when you have explicit authorization.
- Keep DVWA and other intentionally vulnerable applications isolated from public networks.
- Never commit passwords, GitHub tokens, SonarQube tokens, private keys, session cookies, `.env` files, or captured credentials.
- Sanitize ZAP, Burp, SonarQube, SCA, and CI reports before sharing them.
- Treat generated findings as leads requiring validation, not automatic proof of a vulnerability.
- Review exceptions and risk acceptances periodically instead of suppressing findings permanently.
- Do not publish internal course PDFs or other copyrighted material without permission.

The included `.gitignore` excludes common secrets, reports, dependency data, database files, temporary content, and the local `sources/` directory. It is defense-in-depth, not a substitute for reviewing every commit.

## Design and implementation

The site is generated by `lab-guides/build-html.mjs` using the pinned `marked` Markdown parser. CSS and client-side JavaScript are embedded directly into the generated HTML.

The browser-side JavaScript provides only local interface behavior:

- document tabs;
- internal navigation;
- topic filtering;
- code-copy controls;
- theme preference through `localStorage`; and
- a back-to-top control.

It does not send data to a server or third party.

## Validation

Before publishing an update, verify:

- `npm ci` completes successfully;
- `npm run build` completes successfully;
- the generated site contains all five document panels;
- topic links reach valid heading anchors;
- code blocks and tables render correctly;
- no links reference the excluded `sources/` directory in the public artifact;
- the page works at desktop and mobile widths;
- light/dark mode and code-copy buttons work; and
- no sensitive data or internal PDFs are staged for commit.

## Contributing

When proposing changes:

1. Update the relevant Markdown source rather than editing only the generated HTML.
2. Keep commands clearly labelled by shell and operating system.
3. Use placeholders such as `CHANGE_ME` instead of real credentials or domains.
4. Explain when a snippet intentionally differs from a supplied lab example.
5. Keep destructive or high-impact security testing explicitly scoped and authorized.
6. Run `npm run build` and review the generated output.
7. Include both source and generated changes in the pull request.

## Limitations

- Tool interfaces, container images, action versions, and security recommendations change over time.
- Templates must be adapted to the actual application, language, architecture, and course requirements.
- Static analysis and scanners do not replace threat modeling, manual review, or security testing.
- Microsoft Threat Modeling Tool requires Windows; OWASP Threat Dragon is the cross-platform alternative covered by the labs.
- GitHub Pages is static hosting and does not provide application-level access control for a normal public site.

## License and attribution

No license has been assigned to the original course materials. Do not assume that the absence of a license grants permission to redistribute them.

Before adding an open-source license to this repository, confirm which original content you own and which content is derived from institutional materials. Retain appropriate attribution and follow institutional policy.
