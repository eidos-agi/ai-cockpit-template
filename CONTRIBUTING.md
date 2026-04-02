# Contributing to AI Cockpit

Thanks for your interest in contributing.

## Quick Start

```bash
gh repo clone eidos-agi/ai-cockpit-template
cd ai-cockpit-template
pip install -e ".[dev]"
pytest tests/ -v
```

## Development

- Python 3.10+
- Tests: `pytest tests/ -v`
- The CLI entry point is `src/ai_cockpit/cli.py`
- Template files (bundled in the pip package) live in `src/ai_cockpit/template/`
- Skills (for Claude Code) live in `.claude/skills/`

## What to Contribute

- **New skills** — Add to `.claude/skills/<skill-name>/skill.md` and copy to `src/ai_cockpit/template/.claude/skills/`
- **CLI improvements** — Edit `src/ai_cockpit/cli.py`, add tests
- **Bug fixes** — Always welcome
- **Documentation** — README, doc guides, inline help text

## Pull Requests

- One feature per PR
- Add tests for CLI changes
- Update CHANGELOG.md
- Run `pytest tests/ -v` before submitting

## Code Style

- No type annotations required (but welcome)
- Keep the CLI as a single file — it's a feature, not a limitation
- ANSI colors via escape codes, not dependencies
- Textual is the only heavy dependency — keep it that way

## License

By contributing, you agree that your contributions will be licensed under the MIT License.
