# Code skills

A set of skills for Claude Code to fight the slop.

## Installation

Clone the repo into `~/.claude/skills/`

```
mkdir -p ~/.claude/skills
git clone https://github.com/joshnuss/code-skills.git ~/.claude/skills/code-skills
```

## Sources

- [Code Complete 2: Developer Best Practices](https://a.co/d/00J37bJU)
- [Design Patterns: Elements of Reusable Object-Oriented Software](https://a.co/d/05UJihQv)

## Skills

### Minimal Comments

Keeps comments to a minimum. Avoids comments that would be better described with code.

### Minimal Complexity

Run Cyclomatic complexity checks to ensure functions don't get super nested.

### Small Routines

Avoids long and windy functions. Keeps functions short and sweet.

### Creational Patterns

Reviews object construction code for a missing or overused Singleton, Factory Method, or Builder.

### Structural Patterns

Reviews wrapper and subsystem-facing code for a missing or overused Adapter, Decorator, or Facade.

### Behavioral Patterns

Reviews dispatch, event, and state-transition code for a missing or overused Strategy, Observer, Command, Template Method, or State.

## License

MIT
