# Working rules (keep token use low)

## Long-running jobs
- Run long jobs in the background and wait for the finish only. No progress
  monitors that report every N items; at most one check-in per ~10 minutes.
- Before a long run, test on a tiny subset (1–3 items) first.
- Log to a file; on failure read only the last ~30 lines (`tail -n 30`), not the whole log.
- If a job dies silently, find the cause before rerunning. Check the obvious
  first: out of memory, disk full, WSL/VM shutdown, sleep/idle timeout.
  Do not rerun the same command more than twice without a new diagnosis.
- On WSL, keep long jobs alive (e.g. run under `nohup`/`tmux`, and keep a
  WSL terminal open or raise `vmIdleTimeout` in `.wslconfig`).

## Output size
- Never print full JSON or big tables. Print a short summary (counts, min/max,
  a few sample rows) and save the full result to a file.
- Pipe noisy commands through `head`/`tail`/`grep`.

## Images
- Look at an image only when the numbers can't answer the question.
- Make previews small (≤1024 px, few tiles per contact sheet). Don't re-open
  an image already viewed unless it changed.

## Code edits
- Edit the part that changes; don't rewrite whole scripts.
- Put reusable code in a script file and call it, instead of pasting it again.

## Research / writing
- Write findings into files under `research_notes/` as you go; reply in chat
  with a short summary and the file path, not the full text.
