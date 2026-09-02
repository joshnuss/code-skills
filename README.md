# Code skills

A set of skills for Claude Code to fight the slop.

## Installation

Clone the repo into `~/.claude/skills/`

```
mkdir -p ~/.claude/skills
git clone https://github.com/joshnuss/code-skills.git ~/.claude/skills/code-skills
```

## Sources

- [Code Complete 2](https://a.co/d/00J37bJU)

## Skills

### Minimal Comments

Keeps comments to a minimum. Avoids comments that would be better described with code.

### Minimal Complexity

Run Cyclomatic complexity checks to ensure functions don't get super nested.

## License

MIT
