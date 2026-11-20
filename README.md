# Agent skills

[Agent skills](https://agentskills.io/specification) for working with coding agents:

- **archquiz** (`archquiz-index`, `archquiz-quiz`): quizzes you on how your codebase works.
- **code-review**: reviews a branch and grades every finding from P0 to P3, where P0 and P1 should block the merge.

## Install

Copy the folders you want from `skills/` to where your agent loads skills, such as `~/.claude/skills/` for Claude Code.

## archquiz

1. **Map your system:** run the `archquiz-index` skill. It proposes the areas worth understanding and writes them to `.archquiz/` once you confirm. Run it again monthly, or when a quiz says the index is out of date.
2. **Get quizzed:** run the `archquiz-quiz` skill. Ten minutes of questions written from the current code, scored per area; areas you get wrong come back sooner.

### Why

Agents keep adding parts you never learned. The quiz finds the parts of your system you no longer understand, your [comprehension debt](https://codemanship.wordpress.com/2025/09/30/comprehension-debt-the-ticking-time-bomb-of-llm-generated-code/), before you have to fix them.

### How it works

- The index names areas (checkout, payments, order export, deployment, ...) and where in the code each one starts, but never describes them, so it doesn't go stale.
- Every quiz is written from the current code, and every wrong answer becomes study notes, a corrected answer key or a fix.
- If your repository has a CODEOWNERS file, each person is quizzed on the parts they own, including those owned by their teams. Without one, you're quizzed on the whole system.
- Both skills write only to `.archquiz/`. Quiz history is gitignored by default.

## code-review

Run the `code-review` skill on a branch. It reviews everything since the branch split from its base, including uncommitted changes, and writes the findings to `.code-review/<run>/findings.yaml`, where the run names the branch, the commit and whether the working tree was dirty. Every run gets its own folder, and `.code-review/` ignores itself.

- Each finding has a severity, a file and line, a title, why it matters and an optional suggestion.
- Severity comes from one question: should this stop the merge? P0 and P1 mean yes; P2 and P3 mean no.
- The skill never edits your code, commits or changes git state.
- To use it as a gate, fail the CI job when the findings file has any P0 or P1 finding.
- [chief](https://github.com/nareto/chief) runs the same review in a loop of fresh agents until a pass finds nothing new, and has a [GitHub workflow example](https://github.com/nareto/chief/blob/master/examples/github-review.yml) that posts the findings on the pull request and fails the check on P0 or P1.

## License

[Apache 2.0](LICENSE)
