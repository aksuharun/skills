# Commit Message Rules

## The 7 Rules

1. **Separate subject from body with a blank line**
2. **Limit the subject line to 50 characters** (72 absolute max)
3. **Capitalize the subject line**
4. **Do not end the subject line with a period**
5. **Use the imperative mood in the subject line**
6. **Wrap the body at 72 characters**
7. **Use the body to explain _what_ and _why_, not _how_**

---

## Rule Details

### Imperative mood (Rule 5)

Subject lines must complete this sentence: _"If applied, this commit will **[subject]**"_

Good examples:
- `Refactor subsystem X for readability`
- `Remove deprecated methods`
- `Fix failing login test`

Bad examples (fail the test):
- ~~`Fixed bug with Y`~~ (past tense)
- ~~`Changing behavior of X`~~ (present participle)
- ~~`More fixes for broken stuff`~~ (vague description)

### Subject vs. body

Use a one-liner when the change is self-explanatory:
```
Fix typo in introduction to user guide
```

Add a body when context is needed:
```
Derezz the master control program

MCP had become intent on world domination. This commit throws
Tron's disc into MCP (causing its deresolution) and turns it
back into a chess game.
```

### Body content (Rule 7)

Focus on **why** the change was made and **what** changed at a high level. Code explains *how*; the commit message explains *why*.

Good body content:
- What problem this solves
- Why this approach was chosen
- Side effects or non-obvious consequences
- Issue tracker references at the bottom (`Resolves: #123`)

### Full example

```
Summarize changes in around 50 characters or less

More detailed explanatory text, if necessary. Wrap it to about 72
characters or so.

Explain the problem that this commit is solving. Focus on why you
are making this change as opposed to how.

- Bullet points are okay, too
- Use a hyphen or asterisk, preceded by a single space

Resolves: #123
See also: #456, #789
```
