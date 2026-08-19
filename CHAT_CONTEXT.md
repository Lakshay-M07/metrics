# GitHub Metrics & Profile README — Chat Context

This document records the full working context, decisions, configuration, troubleshooting history, and current state from the GitHub profile/metrics setup conversation.

## 1. Goal

The project is to redesign the GitHub profile page for `Lakshay-M07` using a modern, clean, card-based layout. The user wants multiple generated SVG cards from `lowlighter/metrics`, plus a custom profile README and a custom Tech Stack section. The metrics repository is a fork of `lowlighter/metrics` and is public:

- Repository: `https://github.com/Lakshay-M07/metrics`
- Default branch: `master`
- It is a fork of `lowlighter/metrics`.
- The profile repository is `https://github.com/Lakshay-M07/Lakshay-M07` on `main`.

The username profile repository must remain public because it is the special repository whose `README.md` is rendered on the GitHub profile page. The metrics repository also remains public because the README embeds its generated SVGs through public `raw.githubusercontent.com` URLs.

## 2. Important conceptual clarification

A repository-template example from `lowlighter/metrics` was initially considered, but it was determined to be the wrong plugin for the profile use case. The repository template is intended to render repository-specific content, not to change the appearance of the user's GitHub profile. The user explicitly does not want fancy repository cards or repository-specific decorations. That plugin was dropped.

The profile design instead uses separate metrics cards generated into separate SVG files, then embedded in the special username README repository.

## 3. Metrics action source

The workflow deliberately uses the official action:

```yaml
uses: lowlighter/metrics@latest
```

It does **not** currently execute `Lakshay-M07/metrics@master`. The forked `Lakshay-M07/metrics` repository is being used mainly as the repository that stores the generated SVG files and workflow/configuration files. This distinction was important during debugging.

The workflow file currently lives at:

`.github/workflows/metrics.yml`

Current fetched contents were verified directly from GitHub. The current workflow is documented below in Section 11.

## 4. Initial metrics workflow

The first metrics job is the base GitHub metrics card. Current relevant configuration:

```yaml
name: Metrics

on:
  schedule:
    - cron: "0 0 * * *"
  workflow_dispatch:

jobs:
  github-metrics:
    runs-on: ubuntu-latest
    environment:
      name: production
    permissions:
      contents: write
    steps:
      - name: Generate GitHub Metrics
        uses: lowlighter/metrics@latest
        with:
          token: ${{ secrets.METRICS_TOKEN }}
          user: Lakshay-M07
          output_action: commit
          filename: github-metrics.svg
          config_footer: no
```

The `production` environment was initially confusing because no environment existed. A `production` environment was then created/configured, and the workflow could run successfully. The successful state was later visually confirmed by a green deployment/environment status.

## 5. Metrics token

The workflow uses the GitHub Actions secret:

```yaml
${{ secrets.METRICS_TOKEN }}
```

The token itself must not be exposed in this document or repository. Only the secret reference is recorded.

## 6. Introduction plugin

Documentation source used:

`https://github.com/lowlighter/metrics/blob/master/source/plugins/introduction/README.md`

The user chose the default introduction configuration and a dark GitHub theme was discussed. The default introduction plugin output was generated successfully.

Important plugin naming:

```yaml
plugin_introduction: yes
```

The generated output file is:

```text
github-metrics-introduction.svg
```

The resulting SVG was inspected successfully. It displayed the user's GitHub account introduction information. The user decided to use the introduction concept as part of the profile, but later the actual profile README was redesigned manually rather than relying exclusively on this card.

## 7. Theme / config-theme incident

The user initially added:

```yaml
config_theme: github_dark
```

This caused the action to fail with an explicit warning/error that `config_theme` was an unexpected input. The supported inputs listed by the action did not include `config_theme`.

This was removed. The lesson recorded for future work is: do not add options unless they are explicitly supported by the current action version/documentation.

## 8. Forked action / Docker build incident

An attempt was made to use the user's forked action version, which reported:

```text
Error response from daemon: No such image: metrics:forked-3.35.0-beta
```

The action tried to rebuild its Docker image. The build then failed inside the Dockerfile while installing Chrome and dependencies, with a long command ending in:

```text
ERROR: failed to build: failed to solve: process "/bin/sh -c chmod +x /metrics/source/app/action/index.mjs ... && npm ci && npm run build" did not complete successfully: exit code: 1
```

The log showed many Debian `debconf` frontend messages, but the important failure was the Docker build returning exit code 1. Runner compatibility itself was explicitly shown as `compatible`.

The decision was to stop using the fork as the action source and return to:

```yaml
uses: lowlighter/metrics@latest
```

The fork remains useful as the repository containing the generated SVGs and workflow files.

## 9. Languages plugin

Documentation source used:

`https://github.com/lowlighter/metrics/blob/master/source/plugins/languages/README.md`

The user explicitly wanted:

- Basic languages plugin enabled.
- In-depth mode enabled.
- A separate SVG for the languages card.
- The card should show languages plus analytics.
- Most-used and recently-used sections were desired.

The documentation showed the in-depth example with:

```yaml
plugin_languages: yes
plugin_languages_indepth: yes
commits_authoring: firstname lastname, username, username@users.noreply.github.com
```

The user supplied their real GitHub commit identity information for matching:

```text
Lakshay-M07
lakshay.mohata@gmail.com
```

The workflow now uses:

```yaml
plugin_languages: yes
plugin_languages_indepth: yes
commits_authoring: Lakshay-M07, lakshay.mohata@gmail.com
plugin_languages_sections: most-used,recently-used
plugin_languages_details: bytes-size,percentage,lines
```

The generated file is:

```text
github-metrics-languages.svg
```

This card was successfully generated and visually inspected. It showed language statistics and a recently-used section.

The documentation also warned that in-depth mode significantly increases workflow time, clones repositories temporarily on the GitHub Action runner, and should be used responsibly.

## 10. Timezone

The user wanted `Asia/Kolkata` for time-based metrics. This was discussed using the `config_timezone` documentation. The habits card in particular showed `timezone Asia/Kolkata` in its generated footer when it rendered.

## 11. Habits plugin

Documentation source used:

`https://github.com/lowlighter/metrics/blob/master/source/plugins/habits/README.md`

The user wanted a habits card and specifically wanted the timezone configured to `Asia/Kolkata`.

The habits documentation described:

- `plugin_habits`
- `plugin_habits_from`
- `plugin_habits_skipped`
- `plugin_habits_days`
- `plugin_habits_facts`
- `plugin_habits_charts`
- `plugin_habits_charts_type`
- `plugin_habits_trim`
- `plugin_habits_languages_limit`
- `plugin_habits_languages_threshold`
- `plugin_habits_languages_threshold`

The habits card initially rendered with `Unexpected error`.

The important error extracted from the logs was:

```text
TypeError: Cannot destructure property 'author' of 'undefined' as it is undefined.
    at file:///metrics/source/plugins/habits/index.mjs:51:21
```

The workflow could still save an SVG but the generated image displayed `Unexpected error` under `Recent coding habits`.

The conclusion was that this was not caused by the user's basic configuration, but by an issue/limitation in the current upstream action/plugin behavior for the data it was processing. The user decided to skip the habits plugin for now and not risk breaking the stable cards.

## 12. Achievements plugin

Documentation source used:

`https://github.com/Lakshay-M07/metrics/blob/master/source/plugins/achievements/README.md`

The user wanted the **compact display** because it looked good.

The documentation options reviewed included:

```yaml
plugin_achievements: yes
plugin_achievements_threshold: C
plugin_achievements_secrets: yes
plugin_achievements_display: detailed | compact
plugin_achievements_limit: 0
plugin_achievements_ignored: ...
plugin_achievements_only: ...
```

The compact example from the documentation was:

```yaml
filename: metrics.plugin.achievements.compact.svg
token: ${{ secrets.METRICS_TOKEN }}
base: ""
plugin_achievements: yes
plugin_achievements_only: >-
  polyglot, stargazer, sponsor, deployer, member, maintainer, developer,
  scripter, packager, explorer, infographile, manager
plugin_achievements_display: compact
plugin_achievements_threshold: X
```

The user's current workflow instead has:

```yaml
filename: github-metrics-achievements.svg
base: ""
plugin_achievements: yes
plugin_achievements_display: compact
plugin_achievements_threshold: X
plugin_achievements_secrets: yes
plugin_achievements_limit: 0
```

This generated card failed with `Unexpected error`. The only visible plugin log around that time showed `Plugin errors`, followed by an image displaying the same unexpected error. The user emphasized that the code matched the documentation. The action version in the log was:

```text
Source: lowlighter
Version: 3.34.0
Image tag: v3.34
Is released version: 1
Using pre-built version v3.34
```

The user then decided to skip the achievements plugin for the time being and move on. The user later reiterated that the achievements fix should be handled carefully and separately, without breaking the working metrics cards.

## 13. Isometric calendar

The user chose the isometric contribution calendar. It was generated successfully and visually inspected.

The generated file is:

```text
github-metrics-isocalendar.svg
```

The workflow settings currently include:

```yaml
base: ""
plugin_isocalendar: yes
plugin_isocalendar_duration: full-year
```

The user explicitly said this plugin generated perfectly.

## 14. Current metrics repository workflow

The currently fetched `.github/workflows/metrics.yml` in `Lakshay-M07/metrics` contains these jobs under one job step sequence:

```yaml
- name: Generate GitHub Metrics
  uses: lowlighter/metrics@latest
  with:
    token: ${{ secrets.METRICS_TOKEN }}
    user: Lakshay-M07
    output_action: commit
    filename: github-metrics.svg
    config_footer: no

- name: Introduction
  uses: lowlighter/metrics@latest
  with:
    token: ${{ secrets.METRICS_TOKEN }}
    user: Lakshay-M07
    filename: github-metrics-introduction.svg
    base: ""
    plugin_introduction: yes

- name: Languages
  uses: lowlighter/metrics@latest
  with:
    token: ${{ secrets.METRICS_TOKEN }}
    user: Lakshay-M07
    filename: github-metrics-languages.svg
    base: ""
    plugin_languages: yes
    plugin_languages_indepth: yes
    commits_authoring: Lakshay-M07, lakshay.mohata@gmail.com
    plugin_languages_sections: most-used,recently-used
    plugin_languages_details: bytes-size,percentage,lines

- name: Achievements
  uses: lowlighter/metrics@latest
  with:
    token: ${{ secrets.METRICS_TOKEN }}
    user: Lakshay-M07
    filename: github-metrics-achievements.svg
    base: ""
    plugin_achievements: yes
    plugin_achievements_display: compact
    plugin_achievements_threshold: X
    plugin_achievements_secrets: yes
    plugin_achievements_limit: 0

- name: Isometric calendar
  uses: lowlighter/metrics@latest
  with:
    filename: github-metrics-isocalendar.svg
    token: ${{ secrets.METRICS_TOKEN }}
    base: ""
    plugin_isocalendar: yes
    plugin_isocalendar_duration: full-year
```

The file was fetched from GitHub and its blob SHA at the time of recording was:

`f383787c80f48eff167fc0710598d309bb456642`

## 15. Metrics footer

The generated `github-metrics.svg` displayed a footer:

```text
These metrics include private contributions
Last updated ... with lowlighter/metrics@3.34.0
```

The user did not want this footer in the card.

`config_footer: no` was added to the base metrics job, but the footer remained in the generated SVG. This established that the option was not being honored by the current `lowlighter/metrics@latest` action/version in this context.

Do **not** assume or reintroduce `config_footer` as a working solution without verification. The user ultimately decided to leave the footer as-is for now rather than destabilize the working setup.

## 16. Profile repository

The user's special profile repository is:

`https://github.com/Lakshay-M07/Lakshay-M07`

It contains `README.md` on the `main` branch and GitHub explicitly confirmed that this is the special repository whose README appears on the profile.

The initial profile README was just `Hi there 👋` plus GitHub's generated comment. The user wanted to replace it with a custom, modern developer profile.

## 17. Current profile intro design

The current profile introduction is centered and uses:

```md
# Hi, I'm Lakshay Mohata 👋

### AI & Data Science Student

Passionate about building intelligent software, solving real-world problems with AI, and contributing to open source.

**Large Language Models • Machine Learning • AI Engineering • Backend Development • Distributed Systems**

[LinkedIn](https://www.linkedin.com/in/lakshay-mohata-35024a320/) •
[Email](mailto:lakshay.mohata@gmail.com)
```

The user explicitly wanted the dot separators to remain in the line of topics and social links.

The terminal-style `lakshay@dev ~ % whoami` idea was explicitly removed.

The user did not want to emphasize Voice AI in the profile introduction.

## 18. Social links

The profile includes:

- LinkedIn: `https://www.linkedin.com/in/lakshay-mohata-35024a320/`
- Email: `mailto:lakshay.mohata@gmail.com`

The user wanted a simple social-link row rather than a `Reach me` style label.

## 19. Custom Tech Stack section

The user wanted a manually customized Tech Stack section rather than a metrics plugin. The final categories selected were:

1. AI & Machine Learning
2. Languages
3. Development
4. Backend

The user specifically removed Python and OpenCV from AI & ML at one point, then later requested a final AI & ML row using:

- PyTorch
- TensorFlow
- OpenCV

The current working profile code uses this for AI & ML:

```html
<p>
  <img src="https://skillicons.dev/icons?i=pytorch,tensorflow,opencv" />
</p>
```

The Languages row is:

```html
<p>
  <img src="https://skillicons.dev/icons?i=python,cpp,c,js,ts" />
</p>
```

The Development row is:

```html
<p>
  <img src="https://skillicons.dev/icons?i=react,vite,tailwind" />
</p>
```

The Backend row is:

```html
<p>
  <img src="https://skillicons.dev/icons?i=firebase,supabase" />
</p>
```

## 20. Local AI logos / assets

The user created an `assets/ai` directory in the profile repository and uploaded custom AI logos during experimentation. The original files included Hugging Face, Ollama, and OUMI images. These local logos had rendering problems because of black-on-dark backgrounds, aspect-ratio/scale differences, and image transparency.

The user later chose to remove Hugging Face, OUMI and Ollama from the final Tech Stack AI/ML row and use TensorFlow and OpenCV instead. Therefore, these custom local AI logos are no longer needed for the final Tech Stack display.

## 21. Skill icon rendering / alignment decisions

The user wanted the Tech Stack icons left aligned rather than centered. The centered design created too much empty space when a row contained only a few icons.

Important Markdown/HTML behavior learned during the process:

- A top-level `<div align="center">` will center all subsequent content until it is closed.
- It was important to close the intro's centered `<div>` before starting the Tech Stack section.
- The metrics images should be wrapped in their own separate `<div align="center">...</div>` so the metrics cards remain centered while the Tech Stack remains left aligned.

The profile currently uses the following conceptual layout:

```text
Centered intro

Tech Stack (left aligned)
  AI & ML
  Languages
  Development
  Backend

Centered GitHub Metrics cards
  github-metrics.svg
  github-metrics-languages.svg
  github-metrics-isocalendar.svg
```

## 22. Heading separator issue

The user noticed gray horizontal lines appearing under `#` / `##` / heading markup in the GitHub README preview.

Markdown headings rendered by GitHub can show an automatic separator/border in the profile page. The user wanted to remove those extra lines while keeping section titles large.

Several approaches were tried:

- `<h1>` / `<h2>` still showed heading-style lines in the profile rendering.
- Inline CSS on `<p>` was stripped/ignored.
- Plain Markdown heading tags could not simultaneously provide a custom intermediate size and no separator.

The user eventually accepted a visually close setup and moved on.

## 23. Current preferred profile metrics embed layout

The metrics SVGs are embedded from the metrics repository's raw GitHub URLs. Current links are:

```text
https://raw.githubusercontent.com/Lakshay-M07/metrics/master/github-metrics.svg
https://raw.githubusercontent.com/Lakshay-M07/metrics/master/github-metrics-languages.svg
https://raw.githubusercontent.com/Lakshay-M07/metrics/master/github-metrics-isocalendar.svg
```

The user experienced a zoomed-in display when `width="100%"` was used on the images. The preferred solution is to omit explicit `width="100%"` and allow the SVGs to display at their natural size.

The user also wanted extra vertical spacing between these cards. The current preferred use is `<br><br>` between cards, rather than excessive `<br>` counts.

## 24. Current README content at the end of the conversation

The current profile README structure, in its latest stated form, was:

```html
<div align="center">

# Hi, I'm Lakshay Mohata 👋

### AI & Data Science Student

Passionate about building intelligent software, solving real-world problems with AI, and contributing to open source.

**Large Language Models • Machine Learning • AI Engineering • Backend Development • Distributed Systems**

<br>

[LinkedIn](https://www.linkedin.com/in/lakshay-mohata-35024a320/) •
[Email](mailto:lakshay.mohata@gmail.com)

</div>

### Tech Stack
---

### AI & Machine Learning

<p>
  <img src="https://skillicons.dev/icons?i=pytorch,tensorflow,opencv" />
</p>

### Languages

<p>
  <img src="https://skillicons.dev/icons?i=python,cpp,c,js,ts" />
</p>

### Development

<p>
  <img src="https://skillicons.dev/icons?i=react,vite,tailwind" />
</p>

### Backend

<p>
  <img src="https://skillicons.dev/icons?i=firebase,supabase" />
</p>

---

<div align="center">

<img src="https://raw.githubusercontent.com/Lakshay-M07/metrics/master/github-metrics.svg" />

<br><br>

<img src="https://raw.githubusercontent.com/Lakshay-M07/metrics/master/github-metrics-languages.svg" />

<br><br>

<img src="https://raw.githubusercontent.com/Lakshay-M07/metrics/master/github-metrics-isocalendar.svg" />

</div>
```

Later, the user considered changing `### Tech Stack` to `## ⚡ Tech Stack` or an HTML title to enlarge it, while preserving a clean separator style. The exact final title formatting was still under refinement.

## 25. Profile card order and design target

The user wanted the overall profile page to be modern and clean, using multiple cards rather than one giant combined metrics card. The intended high-level visual order is:

1. Custom intro
2. Social links
3. Tech Stack
4. GitHub Metrics title
5. Main metrics card
6. Languages metrics card
7. Isometric contribution calendar

The user wanted enough spacing between the visual cards to prevent them from feeling cramped, but not excessive blank space.

## 26. Repository visibility

The user asked whether the profile repo and metrics repo could be private.

Conclusion used in the conversation:

- `Lakshay-M07/Lakshay-M07` must remain public for GitHub to display the profile README.
- `Lakshay-M07/metrics` should remain public with the current raw SVG embedding approach because the profile README needs to publicly load those SVGs.
- Secrets such as `METRICS_TOKEN` are kept in GitHub Secrets and are not committed as plaintext.

## 27. User preferences that directly affect this project

- Follow documentation closely, especially official `lowlighter/metrics` README files.
- Do not hallucinate configuration options; verify against the docs or actual workflow inputs.
- Check the code for errors before telling the user to add more.
- Prefer one step at a time when making changes.
- When a plugin or card breaks, diagnose the actual log before changing more things.
- Avoid unnecessary repository plugins or fancy repository-specific cards.
- Focus on the profile page, not on beautifying project repositories.
- Use multiple separate SVG cards for a modern clean profile.
- User wants a dark, GitHub-style aesthetic.
- User likes dot separators in introductory text/social rows.
- User wants the skills section left-aligned for compact groups.
- User does not want Voice AI emphasized in the profile introduction.

## 28. Current state / known outstanding items

Working successfully:

- Base `github-metrics.svg`
- Introduction SVG
- Languages SVG with in-depth analysis
- Isometric calendar SVG
- Custom profile README intro
- Social links
- Custom Tech Stack section

Known issues / intentionally paused:

- Habits plugin: generated `Unexpected error` due to a plugin runtime error involving undefined `author` data.
- Achievements plugin: generated `Unexpected error`; user chose to skip it for now.
- Metrics footer: remains visible despite `config_footer: no`; user chose to leave it for now.

The user explicitly said they did not want more plugins at that point and wanted to fix achievements later, carefully and without risking the working setup.

## 29. Last known GitHub workflow file citation/source

The current workflow was fetched directly from:

`https://github.com/Lakshay-M07/metrics/blob/master/.github/workflows/metrics.yml`

The fetched file contained 7 tool-output lines; the YAML content was in line 2 and the current blob SHA was `f383787c80f48eff167fc0710598d309bb456642`.

## 30. Important caution for future work

Do not automatically switch the action back to the user's fork or alter the upstream action source. The action currently works via `lowlighter/metrics@latest`, and previous attempts to use the fork as the action source caused a Docker build failure. The fork can still store the generated SVGs and workflow files.

Do not enable the failed habits/achievements plugins again until the exact cause is verified from current logs and documentation.

Do not treat `config_footer: no` as proven to work; it remained ineffective in the user's current setup.

Always preserve the current working cards when making subsequent profile changes.
