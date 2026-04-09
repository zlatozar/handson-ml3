A Skill (.github/skills/<name>/SKILL.md) is a passive instruction fragment:

- It is loaded as context — essentially extra instructions injected into the prompt.
- It has no autonomous behavior — it cannot call tools, read files, or take steps on its own.
- It is topic-triggered (via description:) or explicitly invoked via @skill-name.

Best for: defining vocabulary, rules, personas, or output formats (e.g., your @data-validation, @python-pro).