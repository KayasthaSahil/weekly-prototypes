# Weekly Brief, 2026-11-23

**Winner: Reviewers can't tell what an AI-agent PR really did, or whether it is safe to merge.**

**Evidence limits:** Search returned titles and links only. Pages were read through a fetch helper, so Reddit and X were not reached directly. Many quotes come from authors who sell a related tool. Several pages show no year. Pain evidence is moderate to good (the same complaint from several independent places). The India small-business niche had only founder pitches, so I treat it as thin.

**Repo check:** The repo was reachable. This replaces an earlier test brief for the same date (ToolGate, an MCP approval proxy). That brief is still in git history, so I did not repeat it. No other past weeks exist, so there is no rising signal yet. This week and last both sit in "friction when working with coding agents", but they are different problems.

## 1. Problem
Coding agents make PRs faster than people can read them. The reviewer gets a big diff with no clear statement of intent, no sign of what was actually run, and tests that may only confirm the agent's own mistakes. Review is now the bottleneck, and trust is the hard part. Who hurts: tech leads and maintainers on teams using Claude Code, Cursor or Codex. When: every day. How badly: high, because merges slow down or people start rubber-stamping.

Quotes:
- "But the easier it gets to open a PR, the more PRs developers have to review." (Alice Moore, Builder.io, May 21, 2026, updated Oct 2) [link](https://builder.io/blog/developers-drowning-in-ai-prs)
- "my PRs went from 20-line diffs to 800-line sprawls, and my review process hasn't kept up." (Prateek Saxena, Product Hunt thread, about mid-2026) [link](https://www.producthunt.com/p/mergeprotect)
- "AI generated tests asserted that the defective behavior was correct" was a factor in every serious defect review this year. (HN user bunderbunder, about Oct 2026, paraphrased in the page summary) [link](https://news.ycombinator.com/item?id=50061356)

Also: "I was the bottleneck, and my team's velocity was capped by my reading speed." (freeCodeCamp author, quoted in [this post](https://dev.to/max_quimby/10x-prs-1x-reviewers-the-code-quality-bottleneck-166o), Jun 2026). The same post cites LinearB 2026: AI PRs wait 4.6x longer for review. That is a second-hand figure I did not check.

## 2. Why current tools fail
- AI review bots (Copilot review, Claude Code Review, Greptile) try to find bugs. Users say many flags are noise: "Half are the model hallucinating." (Aakash Gupta on X, via the dev.to post above). A bot approval can also satisfy a "needs approval" rule without a human reading anything ([Builder.io](https://builder.io/blog/developers-drowning-in-ai-prs)).
- Claude Code Review is reported at $15-25 per review (same dev.to post, not verified).
- PR caps only limit how many PRs arrive, not how hard each one is to read.
- Tools like Code Board and MergeProtect look at risk or call graphs, but both appear small and unproven (0 reviews, 9 followers on MergeProtect).
- None of them show the reviewer what the agent actually did during the session.

## 3. Three ways to solve it

### Approach A: Reviewer's guide bot (GitHub App)
**Idea:** On every PR, a bot posts one comment: what the change does in plain words, the order to read files in (riskiest first), and what is untested. It never approves or rejects.
**For:** Reviewers on teams that already use GitHub.
**Architecture:** Next.js on Vercel for the webhook and GitHub App, a worker (Python, LangChain) that fetches the diff, runs tree-sitter for changed functions and their callers, an LLM writes the guide, Postgres stores results. Flow: webhook, diff, analysis, comment, update on new commits.
**Hard part:** Making the guide trustworthy and short enough. A wrong summary is worse than none.
**Verdict:** Easiest to ship, but the closest to existing review bots. Gap is moderate.

### Approach B: Proof bundle recorded on the author's machine
**Idea:** A CLI wrapper records the agent session (commands run, tests run, files read) and attaches a signed "proof" to the PR: which changed lines the tests actually executed, and whether the new tests fail when the change is reverted.
**For:** Developers who open agent PRs and want them accepted fast, plus their reviewers.
**Architecture:** Python CLI hooking Claude Code and Cursor hooks or logs, coverage on changed lines, a revert-and-rerun check for new tests, output as JSON plus a markdown PR section, a small GitHub Action that verifies and displays it. Flow: session log, coverage and revert check, bundle, PR check.
**Hard part:** Supporting many agents and test stacks, and stopping people from faking the bundle.
**Verdict:** Most different from what exists and it targets the "tests confirm the bug" complaint directly, but setup effort per repo is real.

### Approach C: Auto-split big agent changes into small stacked PRs
**Idea:** Before pushing, the tool splits one large agent change into a stack of small PRs that each do one thing and each pass tests.
**For:** The person who pushes the PR, not the reviewer.
**Architecture:** Python or Node CLI, git hunk analysis, an LLM groups hunks by purpose, a dependency check so each step builds and passes tests, then it pushes a branch stack through the GitHub API. Flow: diff, group, validate each step, push stack.
**Hard part:** Many agent changes are tangled, so clean splits often don't exist and each step must still compile.
**Verdict:** Attacks size directly, but stacked-PR tools exist and the splits may be poor. Riskiest.

## 4. Recommendation
Build **Approach B** first, as a small CLI plus GitHub Action that checks only two things: do the tests touch the changed lines, and do the new tests fail without the change. That is a clear, checkable claim, not an opinion from an LLM. Add a short Approach A style summary later if people ask for it.

**Biggest risk:** GitHub and the agent vendors could ship the same "what did the agent do" feature natively, and recording hooks differ across agents. A second risk is that most evidence is from people selling tools.

## 5. Scores (1-5, my judgment)
| Problem | Pain | Gap | Buildable in 4 weeks | Skills fit |
|---|---|---|---|---|
| Agent PR review trust | 5 | 3 | 3 | 4 |
| Cheap replay of failed agent runs | 3 | 4 | 4 | 5 |
| Secrets exposed to coding agents | 4 | 2 | 4 | 3 |
| Indian small-shop bookkeeping on WhatsApp | 3 | 2 | 3 | 4 |

## 6. Runner-ups
- **Replay failed agent runs without paying for them again:** record a run once and replay it against the same tool outputs. People say "Nobody has really solved this problem yet" and "It's not reproducible on demand", but all sources sell their own tool, so evidence is weak. [link](https://dev.to/rudrendu_paul/how-we-built-a-way-to-replay-ai-agent-failures-without-paying-to-reproduce-them-4917)
- **Keeping secrets away from coding agents:** `.claudeignore` and `.gitignore` don't stop reads ([The Register, Jan 28, 2026](https://www.theregister.com/2026/01/28/claude_code_ai_secrets_files/)), but at least four tools already exist (Cloak, Tene, Klavex, octarine), so the gap is small.

## 7. India small business and wildcard
Only founder pitches turned up, such as "Most billing apps are too complex for a kirana store owner" ([Khata Bhai](https://www.producthunt.com/p/khata-bhai/khata-bhai)) and "bloated, complicated, or just overkill" ([KhataPlus](https://www.producthunt.com/p/khataplus/khataplus)). These are makers' framing, not shop-owner voices, and Vyapar, Khatabook, Tally and Zoho already exist. A WhatsApp-first bookkeeping bot is the recurring idea, but I found no shop owners complaining directly. No wildcard with solid evidence this week.
