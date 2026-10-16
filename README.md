# opencode-skills

A collection of [opencode](https://opencode.ai) Agent Skills. Each skill is a folder
containing a `SKILL.md` file with YAML frontmatter (`name`, `description`, ...) plus any
supporting scripts or references.

## Install

Clone this repo and copy (or symlink) the skill folders into one of the locations
opencode discovers:

- Project: `.opencode/skills/<name>/SKILL.md`
- Global: `~/.config/opencode/skills/<name>/SKILL.md`
- Project Claude-compatible: `.claude/skills/<name>/SKILL.md`
- Global Claude-compatible: `~/.claude/skills/<name>/SKILL.md`
- Project agent-compatible: `.agents/skills/<name>/SKILL.md`
- Global agent-compatible: `~/.agents/skills/<name>/SKILL.md`

### macOS / Linux

```sh
git clone https://github.com/hatehydroboss/opencode-skills.git
mkdir -p ~/.config/opencode/skills
cp -r opencode-skills/skills/* ~/.config/opencode/skills/
```

### Windows (PowerShell)

```powershell
git clone https://github.com/hatehydroboss/opencode-skills.git
New-Item -ItemType Directory -Force "$env:USERPROFILE\.config\opencode\skills" | Out-Null
Copy-Item -Recurse -Force ".\opencode-skills\skills\*" "$env:USERPROFILE\.config\opencode\skills"
```

## Skills

See the [`skills/`](./skills) directory. Each folder name matches the `name` field in its
`SKILL.md` frontmatter.
