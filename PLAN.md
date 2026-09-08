# Rift

**A static scanner for malicious AI model files. A solo research project by Isuru Panditharatne.**

> The name: every known bypass in this space lives in the *rift* between two programs
> that read the same file and disagree about what it means (the scanner sees data, the
> loader runs code). Closing that rift is the project's thesis.

---

## The one-line version

AI model files can run hidden code the moment you open them. The tools meant to catch
this were repeatedly bypassed in 2025. I am building a scanner that closes the gap that
lets those bypasses through, and testing whether machine learning can catch attacks a
plain blacklist never will.

## Why it matters

Model files use a format (pickle) that can carry hidden instructions, so downloading a
model from a public hub is really downloading a program that runs on load. The popular
scanner, `picklescan`, was broken several times in 2025 (CVE-2025-1716, -1889, -1944,
-1945, plus JFrog and December findings). Every bypass works the same way: **the scanner
and the loader disagree about what is in the file, and the attacker hides in that gap.**
That single idea is the spine of the project.

## What I will build

Three layers. Each one is a finished result on its own, so the project is worth showing
even if I stop early.

- **Layer 1 · The scanner.** Open a model file safely, find every hidden program inside
  it (whatever the file extension or compression), and report what it would run, without
  ever executing it. Cover PyTorch, Keras, joblib and raw pickle files. The key move:
  read the file the way the loader actually reads it, and flag any file where my view and
  the loader's view disagree.
- **Layer 2 · The test kit.** Build my own booby-trapped sample files from the published
  2025 bypasses, then run my scanner, `picklescan` and `modelscan` side by side. Deliver
  one table: what each tool catches and misses.
- **Layer 3 · The AI part.** Train a model to recognise the *shape* of a dangerous file
  rather than specific bad words, so it can flag brand-new attacks. Then test honestly
  whether it actually beats the simpler approach, and report it plainly if it does not.

## What "done" looks like

A command-line tool, a written report, and the comparison table. If a real malicious
model turns up on a public hub while I test at scale, I report it responsibly and that
becomes a disclosure I can point to.

## Timeline (evenings and weekends)

| Stage | Work | Rough time |
|------|------|-----------|
| 1 | Learn the pickle format, build the safe reader and signature checks | 2–3 weekends |
| 2 | Reproduce the 2025 bypasses, build the test corpus and comparison table | 1 weekend |
| 3 | Train and honestly evaluate the ML detector | open-ended |
| 4 | Write it up, publish the tool and report | 1 weekend |

## Two safety rules I will not break

1. **Everything runs in a sealed practice machine** (a VM, no internet, no access to my
   real files). I never load a sample to "check" it, because loading is the trap.
2. **Responsible disclosure.** If I find something live, I tell the hub and the tool
   maintainers first, and never publish a working, copy-pasteable attack.

## Where Claude helps me

I write the code and make the decisions; Claude is my pair. Specifically I will lean on it to:

- explain the pickle format and each opcode until I actually understand it
- review my scanner logic and point out cases I have missed
- help me reproduce the published bypasses safely and set up the isolated test machine
- sanity-check my evaluation so the ML numbers are honest, not cherry-picked
- help me write the report and disclosure clearly

---

*Project name: Rift. Standalone project, unrelated to my MSc dissertation or
current portfolio work. Started September 2026.*
