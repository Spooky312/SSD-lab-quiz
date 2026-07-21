# Secure Software Development Lab Reference

A self-contained browser reference for ICT2216 / ICT2516C secure software development labs.

## Included in `index.html`

- Full lab cheat sheet
- Copy-ready code snippets
- macOS and Apple Silicon companion guide
- Windows 11, PowerShell, and WSL 2 companion guide
- Topic navigation, filtering, dark mode, code-copy buttons, responsive layout, and print styling

The site uses no external JavaScript, CSS, fonts, images, analytics, cookies, or package dependencies.

## Publish with GitHub Pages

1. Create an empty GitHub repository.
2. Upload the complete contents of this folder, including `.github/workflows/deploy-pages.yml`.
3. Commit the files to the repository’s `main` branch.
4. Open the repository on GitHub and go to **Settings -> Pages**.
5. Under **Build and deployment**, select **GitHub Actions** as the source.
6. Open the **Actions** tab and allow the “Deploy static reference to GitHub Pages” workflow to finish.
7. Return to **Settings -> Pages** to find the published URL.

The workflow also has a manual **Run workflow** option.

## Important publication note

The supplied course PDFs are labelled “SIT Internal” and are intentionally excluded from this deployment package. Confirm that you have permission to publish the derived reference content itself before making the repository or Pages site public.

If the material should only be available to authorized people, use an access-controlled platform. A normal public GitHub Pages site is not access-controlled.

## Local preview

Double-click `index.html`, or open it in a browser. No build command or web server is required.

## Updating the page

The HTML is generated from the Markdown files in the parent `lab-guides` directory. After changing those files, run `build-html.mjs` in the original project and copy or commit the regenerated `github-pages-site/index.html`.
