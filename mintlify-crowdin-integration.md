# Integrate Mintlify docs with Crowdin

This guide assumes you already have a Mintlify documentation site in a GitHub repository and want to translate the MDX content with Crowdin.

The setup has three moving parts:

1. `crowdin.yml` tells Crowdin which Mintlify source files to upload and where translated files should be downloaded.
2. `docs.json` tells Mintlify which languages exist and which translated pages belong to each language.
3. A GitHub Actions workflow keeps GitHub and Crowdin synchronized.

After the integration is connected, you can decide how translation work is done in Crowdin: fully AI-generated translations, human translation/review, or a mixed workflow where only part of the content is routed through AI before review.

## Expected repository layout

A typical Mintlify repository starts with English source pages at the root:

```text
.
|-- docs.json
|-- index.mdx
|-- quickstart.mdx
|-- essentials/
|   |-- markdown.mdx
|   `-- settings.mdx
`-- api-reference/
    `-- introduction.mdx
```

Crowdin should download translations into language folders that mirror the source structure:

```text
.
|-- index.mdx                         # English source
|-- quickstart.mdx                    # English source
|-- de/
|   |-- index.mdx
|   `-- quickstart.mdx
|-- fr/
|   |-- index.mdx
|   `-- quickstart.mdx
`-- uk/
    |-- index.mdx
    `-- quickstart.mdx
```

Keep the same file names and folder structure for every language. That makes `docs.json` predictable and makes missing translations easy to find.

## 1. Choose language codes

Use language codes supported by Mintlify and Crowdin. Common examples:

```text
en      English
fr      French
de      German
pt      Portuguese
pt-BR   Brazilian Portuguese
uk      Ukrainian
es      Spanish
ja      Japanese
ko      Korean
zh-Hans Simplified Chinese
zh-Hant Traditional Chinese
```

Do not assume every ISO language code is accepted by Mintlify. For example, if Mintlify rejects a code during deployment, do not add it to `docs.json` until Mintlify supports it.

## 2. Add `crowdin.yml`

Create `crowdin.yml` in the repository root.

Use environment variables for credentials. Do not commit personal tokens.

```yaml
"project_id_env": "CROWDIN_PROJECT_ID"
"api_token_env": "CROWDIN_PERSONAL_TOKEN"
"base_path": "."
"base_url": "https://api.crowdin.com"

"preserve_hierarchy": true

files:
  - source: /**/*.mdx
    translation: /%two_letters_code%/**/%original_file_name%
    ignore:
      - /de/**
      - /fr/**
      - /pt/**
      - /pt-BR/**
      - /uk/**
      - /es/**
      - /ja/**
      - /ko/**
      - /zh/**
      - /zh-Hans/**
      - /zh-Hant/**
      - /node_modules/**
```

What this does:

- `source: /**/*.mdx` uploads MDX files from the repository.
- `translation: /%two_letters_code%/**/%original_file_name%` downloads translations into language folders such as `fr/index.mdx` and `uk/quickstart.mdx`.
- `preserve_hierarchy: true` keeps the folder structure from the source docs.
- `ignore` prevents translated folders from being uploaded back to Crowdin as source files.

If you use Crowdin Enterprise, set `base_url` to your organization API URL, for example:

```yaml
"base_url": "https://your-org.api.crowdin.com"
```

If you use Crowdin.com, keep:

```yaml
"base_url": "https://api.crowdin.com"
```

## 3. Configure source files in Crowdin

In Crowdin, create or open the project for the docs site.

Recommended project setup:

- Source language: English, or the language used in the root Mintlify files.
- Target languages: the languages you want to publish in Mintlify.
- File format: Markdown/MDX content should be handled as Markdown-style content. Keep MDX syntax, imports, code blocks, and component tags intact.
- Translation process: choose AI, human translation, review steps, or a combination.

Practical rules for translators and reviewers:

- Translate frontmatter values such as `title` and `description`.
- Do not translate frontmatter keys such as `title`, `description`, or `icon`.
- Do not translate file paths, imports, component names, props, or code blocks unless the text is clearly user-facing.
- Keep Mintlify component syntax valid. Examples: `<Card>`, `<Accordion>`, `<Tabs>`, `<CodeGroup>`.

## 4. Update `docs.json`

Mintlify does not enable languages only because translated folders exist. You must declare languages in `docs.json`.

Use `navigation.languages` and include the full navigation for each language. The source language should also be present as a language entry.

Example with English source plus French, German, and Ukrainian:

```json
{
  "$schema": "https://mintlify.com/docs.json",
  "theme": "mint",
  "name": "Example Docs",
  "navigation": {
    "languages": [
      {
        "language": "en",
        "default": true,
        "tabs": [
          {
            "tab": "Guides",
            "groups": [
              {
                "group": "Getting started",
                "pages": [
                  "index",
                  "quickstart"
                ]
              },
              {
                "group": "Writing content",
                "pages": [
                  "essentials/markdown",
                  "essentials/settings"
                ]
              }
            ]
          }
        ]
      },
      {
        "language": "fr",
        "tabs": [
          {
            "tab": "Guides",
            "groups": [
              {
                "group": "Getting started",
                "pages": [
                  "fr/index",
                  "fr/quickstart"
                ]
              },
              {
                "group": "Writing content",
                "pages": [
                  "fr/essentials/markdown",
                  "fr/essentials/settings"
                ]
              }
            ]
          }
        ]
      },
      {
        "language": "de",
        "tabs": [
          {
            "tab": "Guides",
            "groups": [
              {
                "group": "Getting started",
                "pages": [
                  "de/index",
                  "de/quickstart"
                ]
              },
              {
                "group": "Writing content",
                "pages": [
                  "de/essentials/markdown",
                  "de/essentials/settings"
                ]
              }
            ]
          }
        ]
      },
      {
        "language": "uk",
        "tabs": [
          {
            "tab": "Guides",
            "groups": [
              {
                "group": "Getting started",
                "pages": [
                  "uk/index",
                  "uk/quickstart"
                ]
              },
              {
                "group": "Writing content",
                "pages": [
                  "uk/essentials/markdown",
                  "uk/essentials/settings"
                ]
              }
            ]
          }
        ]
      }
    ]
  }
}
```

Important details:

- Keep English/source inside `navigation.languages`. This lets Mintlify treat it as the source language in the language switcher and admin views.
- Use root paths for English source files: `index`, `quickstart`, `essentials/markdown`.
- Use language-prefixed paths for translated files: `fr/index`, `de/index`, `uk/index`.
- Do not use the same page path in multiple language entries.
- Add a language to `docs.json` only when the translated files exist or when you are ready for Mintlify to expect those paths.

## 5. Add GitHub secrets

In GitHub, open the repository settings and add these secrets:

```text
CROWDIN_PROJECT_ID
CROWDIN_PERSONAL_TOKEN
```

Use a Crowdin token with access to upload source files and download translations for the project.

The workflow can use GitHub's built-in `GITHUB_TOKEN` for creating branches and pull requests. If your repository has stricter permissions, use a GitHub App token or a fine-grained token with contents and pull request permissions.

## 6. Add the GitHub Actions workflow

Create `.github/workflows/crowdin-sync.yml`.

This workflow uploads source MDX files to Crowdin and opens a pull request with downloaded translations.

```yaml
name: Sync Crowdin translations

on:
  push:
    branches:
      - main
    paths:
      - "**/*.mdx"
      - "docs.json"
      - "crowdin.yml"
      - ".github/workflows/crowdin-sync.yml"
  workflow_dispatch:

permissions:
  contents: write
  pull-requests: write

jobs:
  crowdin-sync:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Upload sources and download translations
        uses: crowdin/github-action@v2
        with:
          upload_sources: true
          upload_translations: false
          download_translations: true
          localization_branch_name: l10n_crowdin_translations
          create_pull_request: true
          pull_request_title: "Update Crowdin translations"
          pull_request_body: "Downloads the latest translations from Crowdin."
          pull_request_base_branch_name: "main"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
          CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}
```

This is the usual flow:

1. A writer changes English MDX files on `main`.
2. GitHub Actions uploads those source files to Crowdin.
3. Translators, reviewers, or AI workflows complete translation work in Crowdin.
4. The workflow downloads translations into folders such as `fr/`, `de/`, and `uk/`.
5. The action opens or updates a pull request with the translated files.
6. You review and merge the translation pull request.
7. Mintlify deploys the updated localized docs.

## 7. Optional: download only approved translations

If translations must be reviewed before publishing, add export options:

```yaml
with:
  upload_sources: true
  upload_translations: false
  download_translations: true
  export_only_approved: true
  skip_untranslated_strings: true
  localization_branch_name: l10n_crowdin_translations
  create_pull_request: true
```

Use this when you do not want unapproved or untranslated strings to reach the docs repository.

## 8. Optional: separate upload and download workflows

Some teams prefer two workflows:

- Upload source changes automatically on every merge to `main`.
- Download translations manually or on a schedule.

Upload only:

```yaml
name: Upload sources to Crowdin

on:
  push:
    branches:
      - main
    paths:
      - "**/*.mdx"
      - "crowdin.yml"
  workflow_dispatch:

jobs:
  upload-sources:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: crowdin/github-action@v2
        with:
          upload_sources: true
          upload_translations: false
          download_translations: false
        env:
          CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
          CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}
```

Download translations and create a PR:

```yaml
name: Download translations from Crowdin

on:
  workflow_dispatch:
  schedule:
    - cron: "0 6 * * 1-5"

permissions:
  contents: write
  pull-requests: write

jobs:
  download-translations:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: crowdin/github-action@v2
        with:
          upload_sources: false
          upload_translations: false
          download_translations: true
          localization_branch_name: l10n_crowdin_translations
          create_pull_request: true
          pull_request_title: "Update Crowdin translations"
          pull_request_body: "Downloads the latest translations from Crowdin."
          pull_request_base_branch_name: "main"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          CROWDIN_PROJECT_ID: ${{ secrets.CROWDIN_PROJECT_ID }}
          CROWDIN_PERSONAL_TOKEN: ${{ secrets.CROWDIN_PERSONAL_TOKEN }}
```

This setup gives localization managers more control over when translated content is proposed for publishing.

## 9. Validate before merging translation PRs

Before merging a translation PR, check these items:

- `docs.json` contains every language you want Mintlify to render.
- Every page path listed in `docs.json` exists in the repository.
- Translated folders are not uploaded back as source files in `crowdin.yml`.
- Frontmatter is valid YAML.
- MDX component syntax is valid.
- Internal links point to the correct localized paths when needed.
- Mintlify deployment succeeds.

Useful commands:

```bash
python3 -m json.tool docs.json >/dev/null
mint broken-links
mint dev
```

## 10. Common issues

### Translated files are uploaded back to Crowdin as source

Add language folders to the `ignore` list in `crowdin.yml`:

```yaml
ignore:
  - /fr/**
  - /de/**
  - /uk/**
```

### A language does not appear in Mintlify

Check that:

- The language code is supported by Mintlify.
- The language is listed in `navigation.languages`.
- The translated files exist at the paths listed in `docs.json`.
- The source language is also listed in `navigation.languages`.

### Mintlify deployment fails with an invalid language code

Remove that language from `docs.json` or map it to a Mintlify-supported code. Keep the files out of Mintlify indexing until the code is supported.

### The admin view shows duplicate or confusing entries

Check for these causes:

- Translated folders are being treated as source files by Crowdin because they are missing from `crowdin.yml` `ignore`.
- Source and translated pages have the same untranslated frontmatter titles.
- The same page path is listed in more than one language entry in `docs.json`.

### A translated page returns 404

The translated file is missing, or the path in `docs.json` does not match the file path. For example, this entry:

```json
"pages": ["uk/quickstart"]
```

requires this file:

```text
uk/quickstart.mdx
```

## Final checklist

Use this checklist when setting up a new Mintlify and Crowdin integration:

- Create a Crowdin project.
- Add target languages in Crowdin.
- Add `crowdin.yml` to the Mintlify repository.
- Ignore all translated language folders in `crowdin.yml`.
- Add each published language to `docs.json` under `navigation.languages`.
- Keep the source language, usually `en`, in `navigation.languages` and mark it as default.
- Add GitHub secrets for Crowdin credentials.
- Add a Crowdin GitHub Actions workflow.
- Run the workflow manually once.
- Confirm sources appear in Crowdin.
- Complete or pre-translate content in Crowdin.
- Download translations through the workflow.
- Review the translation PR.
- Run Mintlify checks.
- Merge and deploy.
