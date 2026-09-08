# Rift

Finding code hidden inside AI model files, without ever running the file.

## Why this exists

Most PyTorch models are saved with Python's `pickle` format. The catch is that a pickle is not
just data, it is a list of instructions, and loading one runs those instructions. So loading a
model you downloaded from the internet can execute arbitrary code on your machine. The file
extension says "data"; the behaviour is closer to an executable you double-clicked.

Safer formats exist (`safetensors` stores only numbers, and recent PyTorch defaults to a
restricted loader), but the huge base of existing pickle files is not going away, and plenty of
code still turns the safety off because a model will not load otherwise. The gap is real and it
is being actively exploited.

## Where the name comes from

Through 2025, researchers repeatedly slipped malicious models straight past the popular scanners.
Every bypass had the same shape: the scanner reads the file and calls it safe, while the loader
reads the same bytes and runs hidden code. The attacker lives in the gap between those two
readings. Security people call this a parser differential.

That gap is the rift. Closing it is the whole idea: read the file the way the loader actually
reads it, so the two can never disagree in the first place. A scanner built on "detect the
disagreement" rather than "look for known-bad strings" does not fall to that family of bypasses.

## Status

Early, and in active development. This is a solo project I am building to understand
serialization security properly, not a finished product. Things will move around and break.

## Roadmap

1. **Static analyser.** Disassemble the pickles inside a model file and flag dangerous
   constructs, without executing anything. Cover the ways a pickle can be hidden (unusual
   extensions, compression, awkward archive structure), not just the obvious case.
2. **Test corpus and comparison.** Reproduce the known evasion techniques as sample files, then
   measure this scanner against the existing ones. The deliverable is a table of what each tool
   catches and misses.
3. **A learned detector (experiment).** Can a model that learns the shape of the instruction
   stream catch attacks a fixed rule set misses? Tested honestly, including the case where the
   simpler approach wins.

## Safety

This project deals with real code-execution payloads. Any malicious or test sample is generated
and handled inside an isolated virtual machine, with no network and no shared folders, and such
files are never committed to this repository (see `.gitignore`). The scanner never loads a
sample to inspect it, because loading is the trap. Do not run an untrusted model file outside a
sandbox.

## Repository layout

- `01_pickle_is_a_program.ipynb` — a walk through how a pickle carries and runs code
- `PLAN.md` — the project plan and learning roadmap
- `.gitignore` — blocks samples, payloads, and binary model files from ever being committed

## Background

- Python `pickletools` documentation, for reading pickle opcodes
- Hugging Face `safetensors`, the code-free model format
- The 2025 `picklescan` bypass advisories (CVE-2025-1716, -1889, -1944, -1945), which are the
  motivating examples for the parser-differential approach

## Scope

An educational security-research project. Anything found in the wild is handled through
responsible disclosure, and no weaponized, copy-paste payload is published here.
