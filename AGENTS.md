# Agent Guide: Freelens Landing Page

## Overview

This guide helps AI agents understand the Freelens landing page codebase, common
development tasks, and project conventions. Use this as a reference when working
on the project.

This repository contains the static landing page for Freelens, served via
**GitHub Pages** at <https://freelens.app>. It is a single HTML page with
accompanying CSS and images — there is no build system, no JavaScript framework,
and no server-side code.

### Project Structure

```text
.
├── index.html          # Main landing page (download links, badges, etc.)
├── style.css           # All page styles
├── README.md           # Project overview with badges
├── biome.jsonc         # Biome formatter/linter configuration
├── .renovaterc.json5   # Renovate configuration for version bumps
├── .editorconfig       # Editor configuration
├── .gitignore          # Git ignore rules
├── .well-known/        # Domain verification files (e.g. Flathub)
├── images/             # Screenshots and logos
├── .github/workflows/  # CI workflows (pages deploy, trunk-check, etc.)
└── .trunk/             # Trunk linter orchestrator configuration
```

## Security

Never read, display, reference, or include the contents of the following files
in any response or context, even if they are open in the editor:

- `.env`
- `.env.*`
- `.npmrc`
- `*.jks`
- `*.keystore`
- `*.p12`
- `*.pfx`
- `*.pem`
- `*.key`

## No Build System

This is a **static site** — there is no build step, no bundler, and no package
manager needed for the site itself. To preview changes, open `index.html`
directly in a browser or use any static file server:

```bash
python3 -m http.server 8080
# Then open http://localhost:8080
```

The tools used for linting and formatting (Biome, Trunk) are installed via
Trunk's managed runtime system. There is no `package.json` in this repository.

## Version Management with Renovate

Renovate is configured to automatically bump version numbers across the project.
The configuration is in `.renovaterc.json5`.

### Freelens App Version in HTML

The Freelens application version embedded in `index.html` (download links like
`v1.10.0`) is managed by a **custom regex manager** in Renovate. This manager:

- Scans `.html` files for version patterns like `\d+\.\d+\.\d+`
- Uses the `github-releases` datasource to check `freelensapp/freelens` releases
- Opens PRs automatically when a new release is published
- Auto-merges if CI passes (`automerge: true`)

### Tool Versions with Inline Hints

Versions in YAML files (`.trunk/trunk.yaml`, `.github/workflows/*.yaml`) and
`.tool-versions` files use inline Renovate hints in comments:

```yaml
# datasource=github-releases depName=trunk-io/trunk versioning=semver
```

These hints tell Renovate's custom regex manager how to find updates. When
adding or updating a tool version in such files, include the appropriate hint
comment.

### Renovate PRs

- All Renovate PRs get the `renovate` label
- PRs for the `freelens` dependency (app version in HTML) are set to automerge
- `platformAutomerge: true` is enabled globally
- Renovate is disabled in `.semanticCommitsDisabled` mode (no conventional
  commit prefixes)

## Formatting and Linting

### Biome

Biome handles HTML and CSS formatting. Configuration is in `biome.jsonc`.

To check formatting:

```bash
trunk check          # Runs all enabled linters including Biome
```

To auto-format:

```bash
trunk fmt            # Runs all enabled formatters including Biome
```

If `trunk` CLI is not available locally, use `npx trunk` instead.

### Trunk

Trunk orchestrates multiple linters and formatters. The enabled tools are:

- **biome** — HTML/CSS formatting
- **markdownlint** — Markdown linting
- **actionlint** — GitHub Actions workflow validation
- **git-diff-check** — Detects unresolved merge conflicts
- **oxipng** — PNG optimization
- **trufflehog** — Secret scanning
- **yamlfmt** — YAML formatting
- **yamllint** — YAML linting

Trunk is configured to auto-format on pre-commit and run checks on pre-push
(see `.trunk/trunk.yaml`).

## Common Development Tasks

### Updating the Freelens Version

The Freelens app version appears in `index.html` in download links:

```html
<a href=".../download/v1.10.0/Freelens-1.10.0-macos-arm64.pkg">...</a>
```

**Do not manually update these.** Renovate handles this automatically when a new
Freelens release is published. If you need to force an update, you can trigger
Renovate from the Renovate dashboard.

### Adding a New Download Variant

1. Add the new `<a>` tag inside the appropriate `<div class="download-links">`
   in `index.html`
2. Follow the existing URL pattern and version number format
3. Run `trunk check` to validate

### Changing Styles

1. Edit `style.css`
2. Preview changes by opening `index.html` in a browser
3. Run `trunk check` to validate before committing

### Adding a New Badge

1. Add the badge `<img>` or `<a>` tag in the appropriate section of `index.html`
2. GitHub release badges (`shields.io`) do not need version management
3. If the badge references a versioned tool, add a Renovate hint comment

### Updating Screenshots

1. Replace the image file in `images/`
2. Keep the same filename to avoid HTML changes, or update `index.html`
   accordingly
3. Run `trunk check` — oxipng will optimize PNG files automatically

## Troubleshooting

### Changes Not Appearing on the Live Site

1. GitHub Pages deploys from the `main` branch — wait a few minutes after push
2. Check the [pages-build-deployment
   workflow](https://github.com/freelensapp/freelensapp.github.io/actions/workflows/pages/pages-build-deployment)
   for errors
3. Verify the file is not in `.gitignore`

### Renovate Not Opening Version Bump PRs

1. Check the Renovate dashboard for this repository
2. Verify `.renovaterc.json5` regex patterns still match the current HTML
   structure
3. Check if there are open Renovate PRs that need merging first

### Trunk Check Failures

1. Run `trunk check --verbose` to see detailed output
2. Check `.trunk/trunk.yaml` for linter version constraints
3. Some linter issues can be auto-fixed with `trunk fmt`

## Best Practices

1. **Run `trunk check` before committing** — catches formatting and lint issues
2. **Let Renovate manage versions** — do not manually bump version numbers
3. **Preview locally** — open `index.html` in a browser before pushing
4. **Keep the HTML self-contained** — inline styles where practical, use
   `style.css` for shared rules
5. **Follow existing patterns** — match the structure of existing download
   links and badges when adding new ones
6. **Do not use Anthropic Fable for coding tasks** — Fable may be used only for
   planning, analysis, and thinking through problems. When writing or editing
   code, use standard editing tools instead.
7. **Check both desktop and mobile** — the page uses responsive CSS, verify
   changes at multiple viewport widths

## GitHub Actions (Claude Code Action) Rules

When operating via the `claude.yaml` workflow (i.e., invoked from a PR comment,
issue, or review), follow these rules:

### Code Review

When reviewing code and proposing fixes:

1. **Show the diff first** — present every proposed change as a unified diff
   block using the `diff` language tag:

   ```diff
   --- a/index.html
   +++ b/index.html
   @@ -10,7 +10,7 @@
    <a href=".../download/v1.10.0/Freelens-1.10.0-macos-arm64.pkg">...</a>
   -<a href=".../download/v1.10.0/Freelens-1.10.0-macos-amd64.pkg">...</a>
   +<a href=".../download/v1.11.0/Freelens-1.11.0-macos-amd64.pkg">...</a>
   ```

   You can generate this from the terminal with:
   ```bash
   git diff -u -- path/to/file
   ```

   If the change spans multiple files, group them under a single commit
   subject and show each file's diff sequentially.

2. **Propose a commit subject first** — before any code change, output a
   single line with the proposed commit subject:

   ```text
   **Proposed commit:** <short description>
   ```

   Do **not** use Conventional Commits prefixes (e.g. `fix:`, `feat:`,
   `chore:`, `refactor:`, `docs:`, `test:`, `ci:`). This project prefers
   plain, descriptive commit messages and PR titles without any prefix.

   Wait for the user to confirm (or adjust) the subject before applying the
   change.

3. **Comment style:**
   - Keep review comments concise and actionable
   - Reference specific lines (file + line number) when pointing out issues
   - Offer a concrete fix suggestion rather than just flagging a problem
   - Do **not** use emoji in any Markdown, comments, commit messages, or
     PR descriptions. The only exception is emoji that already appears
     inside code strings (e.g. application logs, user-facing messages).
   - Use GitHub's `suggestion` block for small targeted fixes so the PR
     author can accept the change with a single click:

     ````suggestion
     <same unified-diff format as shown above>
     ````

   - For larger multi-file changes, use `diff -u` blocks in a regular
     comment instead, with the proposed commit subject shown first

### Making Changes to a PR

When asked to implement a change on a PR:

1. Propose the commit subject (as above)
2. Describe what will change and why
3. After confirmation, apply the changes with commits on the PR branch
4. **One commit per fix** — when a review surfaces more than one issue or
   the plan includes more than one fix, apply and commit each fix
   separately. Do not batch multiple independent fixes into a single
   commit. This keeps the history bisectable and makes each change easy
   to revert individually.

### Modifying GitHub Actions Workflows

Claude cannot push changes to files under `.github/workflows/` directly,
because the GitHub token used by the action lacks the `workflows` permission.
Any patch to a workflow file MUST therefore be delivered as a new, complete
file under the `github-workflow-fix/` directory instead of editing the file in
place:

1. Write the full, final contents of the workflow to
   `github-workflow-fix/<workflow-file-name>` (e.g.
   `github-workflow-fix/claude.yaml`). Do **not** edit the original file under
   `.github/workflows/`.
2. Make it a **complete** file — the entire workflow as it should look after
   the change, not just a diff or fragment — so it can be copied verbatim.
3. Commit and open the PR as usual. In the PR description, clearly note that
   the file is a proposed workflow change and that a maintainer must move it
   from `github-workflow-fix/` to `.github/workflows/` manually.

This lets the PR be created successfully while leaving the actual workflow
change for a human to apply.

### Branch Naming Conventions

When creating a branch from an issue, use a human-readable name that includes
the issue number and a short slug derived from the issue title:

```text
claude/issue-<number>-<short-slug>
```

- `<number>` is the GitHub issue number
- `<short-slug>` is a kebab-case summary of the issue title, kept short
  (3–6 words maximum, omit articles and filler words)

Examples:

- Issue #1957 "Add PR title convention rule for agent-related changes"
  → `claude/issue-1957-add-pr-title-rules`
- Issue #42 "Fix crash when opening preferences dialog"
  → `claude/issue-42-fix-preferences-crash`

Do **not** use auto-generated timestamp suffixes (e.g.
`claude/issue-1957-20260612-2108`) — these are not human-readable and make
branch lists hard to scan.

### PR Title Conventions

When creating a PR, use the following title conventions:

- **Agent-related changes** — PRs whose changes are strictly related to coding
  agent configuration (e.g. `AGENTS.md`, `.github/workflows/claude.yaml`, or
  other files that govern how Claude operates in this repository) MUST use
  the prefix `Claude:` (followed by a space) in the title.

  Examples:
  - `Claude: Add rule for PR title conventions in AGENTS.md`
  - `Claude: Update claude.yaml workflow permissions`

- **All other PRs** — do **not** use any prefix (no `fix:`, `feat:`, `chore:`,
  etc.). Use plain, descriptive titles.

### Pushing Changes from Fork PRs

When you have commits ready to push but the PR originates from a fork
(different owner than `freelensapp`), you cannot push to the fork's
repository. Instead:

1. Create a new branch on `freelensapp/freelensapp.github.io` with the prefix
   `claude/` followed by the original branch name.
   Push to the `upstream` remote (not `origin`, which points to the fork):
   ```bash
   git checkout -b claude/<original-branch-name>
   git push --force-with-lease upstream claude/<original-branch-name>
   ```

2. Open a new PR from that branch. The new PR MUST use the **exact same
   title** as the original PR — copy it verbatim, do not rewrite, improve,
   or add any prefix. The description MUST reference the original PR
   (e.g. "Fixes #NNN, supersedes #NNN").

3. Post a comment on the original PR:
   - Explain that the fix has been implemented in a new PR
   - Include a link to the new PR
   - Mention that the original PR can be closed

4. Close the original PR.

### Closing PRs

Claude may only close a PR when ALL of the following are true:

1. The PR was created by Claude from a `claude/` branch, OR the PR is the
   original fork PR that Claude's `claude/` branch supersedes (see
   "Pushing Changes from Fork PRs" above).
2. The close reason is explicitly explained in a comment on the PR.

Claude MUST NOT close any PR that does not meet these conditions — even if
asked. Instead, explain to the requester why the PR cannot be closed
automatically and ask a human maintainer to close it manually.

### Model Information in Comments

When operating via the GitHub Actions workflow, always include the model you are
running on in the footer of your GitHub comment and in the PR description when
creating a pull request, alongside the job run link.
Your system environment context states the model name explicitly (e.g.
"You are powered by the model named Sonnet 4.6. The exact model ID is
claude-sonnet-4-6."). Use the exact model ID from that statement.

Format the footer line as:

```text
[View job run](...) | Model: `claude-sonnet-4-6`
```

In a PR description, append the model information at the end of the body:

```text
| Model: `claude-sonnet-4-6`
```

If the system context does not provide a model ID, omit the model field rather
than guessing.

### Development Environment

The GitHub Actions runner has `trunk` CLI and `git` available. For fork PRs,
the `origin` remote points to the contributor's fork. An `upstream` remote is
configured pointing to `freelensapp/freelensapp.github.io`. Push new branches
to `upstream` (never to `origin`) when the PR originates from a fork — this
ensures the resulting PR is internal and CI workflows run automatically.

The following CLI tools are explicitly allowed in the workflow:

- `trunk` (all subcommands) — for formatting and linting
- `git` (all subcommands) — for viewing changes, creating branches,
  committing, and pushing
- `gh` (all subcommands) — for managing pull requests
- `npx`, `node` — for running Node.js tools and scripts inline
- `yq`, `jq` — for YAML and JSON processing
- `grep`, `rg` (ripgrep), `find`, `xargs` — for searching and iterating
- `sed`, `awk`, `cut`, `tr` — for text transformation
- `sort`, `uniq` — for list processing
- `cat`, `head`, `tail`, `wc` — for viewing and measuring files
- `ls`, `tree` — for listing directory contents
- `mkdir`, `touch`, `cp`, `mv`, `rm` — for file and directory operations
- `tee`, `echo` — for pipeline debugging and scripting

Before committing any changes, apply the same validation rules as human
developers:

- Run `trunk check` to validate all files.
- Run `trunk fmt` to auto-format files that have formatting issues.

## Getting Help

- Check existing features for patterns
- Search codebase for similar implementations
- Review PR history for related changes
- Consult AGENTS.md for project conventions
