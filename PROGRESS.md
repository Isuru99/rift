# Rift, progress and handoff notes

A short status file so a fresh chat session can pick up without re-deriving
everything.

## How I want to work on this (important)

- I type the code myself and run it. Do NOT hand me finished notebooks to run.
- Show code in small blocks in chat, one step at a time, and explain what each
  block does and what output to expect before moving on.
- Explain the concepts plainly. I am learning this, not just collecting code.

## Environment

- Windows host: used for the harmless learning steps. Only Python standard
  library so far (pickle, pickletools, zipfile). Python 3.14.
- Kali Linux VM (Hyper-V, isolated, no network, "clean baseline" checkpoint):
  reserved for handling REAL malicious samples later (Layer 2). Not needed yet.

## Done

- Lesson 1: a pickle is a program.
  - Saw pickle.dumps / pickle.loads, and disassembled bytes with pickletools.dis.
  - Understood the pickle stack machine (instructions push/pop a stack).
  - Key idea locked in: STACK_GLOBAL grabs any function, REDUCE calls it. That
    pair is the signature of code execution. Harmless (builtins.print) and
    hostile (os.system) differ only in a couple of strings.

- Lesson 2: a model file is a zip full of pickles, and the scanner-vs-loader
  "rift".
  - Built a fake .pt by hand with pickle + zipfile (no torch). A .pt is just a
    zip: archive/data.pkl (the pickle) plus raw tensor bytes and small text
    files. The .pt extension is only a label; content is what matters.
  - Wrote naive_scan (reads ONLY archive/data.pkl) and thorough_scan (reads
    EVERY entry, decides "is this a pickle" by content: first byte 0x80 = PROTO).
  - Detection uses pickletools.genops (reads opcodes, never loads). Flag set:
    GLOBAL / STACK_GLOBAL / REDUCE.
  - Demonstrated the rift: split_model.pt has a clean data.pkl but a second bad
    pickle at archive/extra.pkl. naive_scan says "safe", thorough_scan catches
    it. Same file, two answers = the parser differential.
  - Note for Layer 1: the same function shows up under different names
    (builtins.print vs __builtin__ print under protocol 2). A name blacklist
    must cover every spelling or it can be bypassed.

## Next

- Layer 1 proper: turn thorough_scan into a real tool. Harden looks_like_pickle
  (0x80 is a weak signal), fix the find_suspicious tuple order so reports are
  exact, and stop trusting file extensions everywhere (decide by content).

## Repo

- Private GitHub repo: rift. Files: README.md, PLAN.md, and lesson notebooks.
- .gitignore already blocks samples, payloads, and binary model files from ever
  being committed.

## Note on the existing notebooks

- 01_pickle_is_a_program.ipynb and 02_model_file_is_a_zip.ipynb were generated
  early. Going forward I am writing the code by hand in chat instead, so treat
  those two files as reference, not the path we are taking.
