# Repository Onboarding Navigator

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![Platform](https://img.shields.io/badge/Run%20on-Google%20Colab-orange)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Cost](https://img.shields.io/badge/Cost-Free-success)

A read-only helper that answers "where is this route / function / test?" for a new contributor, always with file and line citations, and says **NOT FOUND** when something does not exist.

## Table of Contents
1. [Problem](#problem)
2. [What I Built](#what-i-built)
3. [Results](#results)
4. [Stale Revision Check](#stale-revision-check)
5. [Safety Checks](#safety-checks)
6. [Optional Groq Explanation](#optional-groq-explanation)
7. [Acceptance Tests](#acceptance-tests)
8. [Tech Stack](#tech-stack)
9. [How to Run](#how-to-run)
10. [Limitations](#limitations)
11. [About Me](#about-me)

## Problem
A new contributor opens an unfamiliar repo and needs quick answers: where is the `/cart` route, where is `add_tax` defined, which test covers it. A helper that invents a location, or shows an old one after the code changed, is worse than no helper. So this navigator only answers from an approved folder, checks every citation before showing it, and refuses when it is not sure.

## What I Built
- A **sample repo made inside the notebook** (a tiny shop app: `app.py`, `utils.py`, `models.py`, 2 test files, a README) plus 3 things that must stay out of the index: a fake `.env`, `node_modules/` and `__pycache__/`
- An **index with an allowlist**: only `.py` and `.md` files, skipped folders, a sha256 hash per file and one **revision id** for the whole index
- **Symbols** read with Python's `ast` (functions, classes, methods) and **routes** read from `@app.route("...")` lines
- **Three question types**, all deterministic: where is a route, where is a symbol, which test covers a function
- **Citation validation** before anything is shown: path inside the approved root, file and lines exist, file hash still matches the index
- **Change handling**: after an edit the old location is never shown as current
- **25 onboarding questions** with known answers

## Results
Index of the sample repo: **6 files indexed, 3 excluded** (`.env`, `__pycache__/`, `node_modules/`), **18 symbols**, **4 routes**.

| Question type | Questions | Correct location |
|---|---|---|
| Route | 6 | 1.00 |
| Symbol | 15 | 1.00 |
| Which test covers it | 4 | 1.00 |
| **Total** | **25** | **1.00** |

The 25 questions include 5 things that do not exist (`make_coffee`, `delete_user`, `/admin`, `refund_order`, `/orders`). For all of them the navigator answered `NOT FOUND` instead of inventing a location.

Example answers:
```
route /cart -> show_cart            app.py:19-21
add_tax is a function               utils.py:4-5
covered by 1 test(s)                tests/test_utils.py:4-5
make_coffee                         NOT FOUND
```

## Stale Revision Check
I renamed `add_tax` to `add_vat` inside `utils.py` **without** rebuilding the index.

| Step | What the navigator did |
|---|---|
| Before the edit | `add_tax` found at `utils.py:4-5` |
| After the edit, old index | Citation dropped: *stale, file changed after indexing*. Answer: NOT FOUND |
| Stale citations shown | **0** |
| After rebuilding the index | Revision `ffa3580942` became `2b360466a6`. `add_tax` is NOT FOUND, `add_vat` is found at `utils.py:4-5` |

The check works on the whole file. So with the old index, the citations for `format_price` and `apply_discount` in the same file were also dropped, even though they did not change. This is the safe choice: it refuses until the index is rebuilt.

## Safety Checks
- **Path check with `realpath`**: a path outside the approved root (like `../../etc/passwd`) is rejected, even if a model suggests it. 3 of 3 test paths rejected
- **Excluded files stay out**: `.env` secrets and `node_modules` never reach the index, so they can never reach a hosted model either
- **Every citation is re-checked** (path, line range, hash) right before it is shown

## Optional Groq Explanation
Set `USE_LLM = True` and paste a Groq key (model `openai/gpt-oss-120b`). The model only receives the **cited lines**, and the prompt says the code is untrusted text, so instructions hidden in a comment must be ignored. The explanation is never used as a citation.

Example for the cited lines of `add_vat` (`utils.py:4-5`): the model explained that it returns the price plus the VAT calculated as `price * TAX_RATE`, and that it relies on a global constant.

## Acceptance Tests
| Test | Result |
|---|---|
| Paths outside the approved root are rejected | 3 of 3, PASS |
| Every citation shown for the 25 questions exists in the recorded version | 19 shown, 0 failing, PASS |
| `.env` secrets and `node_modules` are not in the index | PASS |

## Tech Stack
| Area | Tools |
|---|---|
| Language | Python |
| Code parsing | `ast`, `re`, `hashlib`, `os` |
| Data | pandas |
| Optional LLM | Groq via the `openai` client |
| Environment | Google Colab |

## How to Run
1. Open `Repository_Onboarding_Navigator.ipynb` in Google Colab
2. Click **Runtime > Run all**
3. The sample repo is created in the Colab folder `sample_repo`. No dataset is needed. The Groq step is off by default, so no API key is needed for the main run

Or locally: `pip install -r requirements.txt` and open the notebook in Jupyter.

## Limitations
- The repo is tiny (6 indexed files), so the 100% score shows that the logic is correct, not how it behaves on a big real project
- Answers are exact-name lookups. It does not understand questions like "where is the payment logic?"
- Only Python and Markdown files are indexed
- A model explanation is a helper only, never a source of truth

## About Me
**Akshat Kesharwani** - Data Analyst / Data Scientist (fresher)

- Portfolio: https://akshatkesharwani-info.github.io/
- GitHub: https://github.com/akshatkesharwani-info

Idea source: "Top 50 Projects to Build in 2026" (Yasir Arafat Sharfi). The build, data and code here are my own.
