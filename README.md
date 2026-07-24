# webdev-frontend-skills

A collection of self-contained agent skills for frontend web development.

## Skills

| Skill | Description |
| --- | --- |
| [`echarts`](skills/echarts/SKILL.md) | Create powerful interactive charts with Apache ECharts, balancing ease of use with extensive customization. |
| [`framework-astro-dev`](skills/framework-astro-dev/SKILL.md) | Core Astro knowledge plus hard-won gotchas: CSS scoping, dev-server proxy Host rewriting, render timing races, and canvas chart pitfalls. |

<!-- Append new skills as rows above this line. -->

## Installation

Install with the [skills CLI](https://github.com/vercel-labs/skills):

```bash
# list available skills
npx skills add r3b1s/webdev-frontend-skills --list

# install the ECharts skill
npx skills add r3b1s/webdev-frontend-skills --skill echarts
```

## Structure

Each skill is self-contained and individually installable:

```
skills/
  <skill-name>/
    SKILL.md        # frontmatter + instructions and examples
    metadata.json   # catalog metadata and references
```

The `SKILL.md` frontmatter name matches its directory name. Skills should link
only to files within their own directory.
