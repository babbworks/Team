# heybabb

A command line companion for Babb Works. Ask questions, explore tools, and find out what's being built — straight from your terminal.

---

## Install

```bash
git clone https://github.com/babbworks/heybabb
cd heybabb
python3 -m venv .venv
source .venv/bin/activate
pip install -e .
```

Requires Python 3.11+.

---

## Commands

```
heybabb                          greet Babb
heybabb tools                    list everything Babb Works is building
heybabb tool <name>              full breakdown of a specific tool
heybabb now                      what the team is working on right now
heybabb version                  knowledge base build info
heybabb ask "<question>"         ask anything in natural language
heybabb ask --ai "<question>"    same, powered by Claude
```

---

## Examples

```bash
heybabb tool bitpads
heybabb ask "what problem does bitpads solve?"
heybabb ask "who made you?"
heybabb now
```

---

## AI Mode

`heybabb ask --ai` uses the Claude API for richer, more conversational responses. It requires an Anthropic API key:

```bash
export ANTHROPIC_API_KEY=your_key_here
heybabb ask --ai "how does bitpads handle double-entry encoding?"
```

Without `--ai`, Babb answers offline from his compiled knowledge base — no network required.

---

## About

Babb is built and maintained by [Babb Works](https://babb.works).
