# CLAUDE.md — AI Assistant Guide for this Repository

## Project Overview

This repository contains a **Korean-language desktop GUI application** for automated manuscript text processing. The program ("제목 자동 삽입 프로그램", meaning "Automatic Title Insertion Program") is a single-file Python Tkinter application that batch-processes `.txt` manuscript files by inserting titles, performing text substitutions, appending hashtags/tags, filtering words, and saving results.

**Target users:** Korean content creators who need to process large batches of text documents with consistent titling, tagging, and word replacement.

---

## Repository Structure

```
/
├── 1.py              # Entire application — single-file, 635 lines
└── CLAUDE.md         # This file
```

There are **no subdirectories**, no package configuration files, and no test suite.

---

## Technology Stack

| Component | Details |
|-----------|---------|
| Language | Python 3 |
| GUI Framework | `tkinter` + `ttk` (standard library) |
| External Dependencies | **None** — pure stdlib only |
| Required Libraries | `tkinter`, `os`, `random` |

No `requirements.txt`, `setup.py`, `pyproject.toml`, or any other dependency file exists because the application has zero external dependencies.

---

## Running the Application

```bash
python3 1.py
```

A Tkinter window (800×800px, scrollable) opens immediately. There is no CLI interface, no configuration file to edit, and no build step.

---

## Code Architecture

The entire application is implemented as a single class inside `1.py`:

### Class: `TitleInserterApp` (lines 6–633)

**`__init__(self, root)`** — Initializes all state variables and calls `create_widgets()`:
- `self.manuscript_files` — list of `.txt` file paths to process
- `self.title_list` — list of title strings loaded from a file
- `self.tag_files` — list of dicts (up to 50), each with: `path`, `tags`, `label`, `preview`, `insert_mode`
- `self.tag_settings` — per-tag-file settings: `hashtag_var`, `position`, `random_position`
- `self.filter_words` — list of words to remove from content
- Numerous `tk.StringVar` / `tk.BooleanVar` for UI state binding

**`create_widgets(self)`** (lines 49–261) — Builds the full scrollable UI using a `Canvas` + `Scrollbar` wrapper. Sections (as `ttk.LabelFrame`):
1. 원고 파일 관리 — Manuscript file management
2. 제목 파일 관리 — Title file management with preview
3. 치환 기능 — Text substitution settings
4. 태그 파일 관리 — Tag file management (1–50 files, two load modes)
5. 필터링 파일 관리 — Filter word file management
6. 저장 경로 설정 — Save path configuration
7. 설정 — General settings (title mode, repeat, append word, tag options, manual hashtag input)

**Key methods:**

| Method | Lines | Purpose |
|--------|-------|---------|
| `on_tag_load_mode_change` | 263–265 | Triggers UI rebuild when load mode radio changes |
| `load_batch_tag_files` | 267–298 | Opens multi-file dialog, loads up to 50 tag files at once |
| `update_tag_files` | 300–370 | Dynamically creates/destroys tag file widgets based on count |
| `add_manuscript_files` | 372–378 | Opens file dialog, appends unique files to list |
| `remove_selected_file` | 380–388 | Removes currently selected file from listbox |
| `clear_all_files` | 390–393 | Clears all manuscript files |
| `load_title_file` | 395–409 | Loads title file, shows 5-line preview |
| `load_tag_file` | 411–425 | Loads a single tag file by index |
| `load_filter_file` | 427–437 | Loads filter word file |
| `select_save_path` | 439–444 | Opens directory dialog for absolute save path |
| `get_save_file_path` | 446–467 | Computes output path based on mode (original/absolute/relative) |
| `get_first_word_from_title` | 469–471 | Extracts first whitespace-delimited token from a title string |
| `replace_text_with_count` | 473–478 | Wraps `str.replace(target, replacement, count)` |
| `execute_processing` | 480–628 | Main processing loop — validates inputs then processes all manuscript files |
| `main` | 630–633 | Entry point — creates `tk.Tk()` and starts `mainloop()` |

---

## Core Processing Logic (`execute_processing`)

For each manuscript file (indexed `i`):

1. **Read** the file as UTF-8 text
2. **Title index**: `i % len(title_list)` — cycles through titles across files
3. **Text substitution** (if enabled): replace `replace_target` with first word of current title, up to `replace_count` times using `str.replace(..., count)` — not a loop, preventing re-replacement of already-replaced text
4. **Title section construction**:
   - *Single mode*: one title repeated 1–N times (or random 1..N if random repeat is on)
   - *Multiple mode*: a slice of `max_repeat_count` consecutive titles starting at `(i * max_repeat_count) % len(title_list)`
   - Optional `append_word` is concatenated after each title
   - Two blank lines added after title section
5. **Filter**: if enabled, each filter word is replaced with `""` in content
6. **Tag insertion**: for each tag file (fixed-insert mode or settings-based):
   - Selects `tag_count` consecutive tags starting at `(i * tag_count) % len(tags)`
   - Position: `top` (insert at line 0), `middle` (insert at midpoint), or `bottom` (append with `tag_spacing` blank lines)
   - Random position selects uniformly from `["top", "middle", "bottom"]`
   - Manual hashtags from the text widget are also appended
7. **Save** to computed path as `{basename}_processed.txt` (optionally prefixed with first title word)

---

## UI Language and Conventions

- **All UI text, variable names, comments, and docstrings are in Korean**
- When editing or adding UI strings, maintain Korean text to match existing style
- When writing code comments or docstrings for new methods, Korean is preferred for consistency with the existing codebase
- English is acceptable in CLAUDE.md and any developer-facing documentation

---

## Key Design Decisions

- **Single-file design**: The application is intentionally contained in one file (`1.py`). Do not split into modules without explicit user instruction.
- **Dynamic widget creation**: Tag file slots (1–50) are created/destroyed dynamically via `update_tag_files()`. Always call this method after changing `tag_file_count_var`.
- **No global state**: All state is stored as instance variables on `TitleInserterApp`. No module-level globals are used.
- **Error handling**: File operations use try/except with `messagebox.showerror` for user-facing errors. Batch processing continues on individual file errors (`continue` in the loop).
- **Tkinter variable binding**: All form fields use `StringVar`/`BooleanVar` for two-way binding. Do not read widget text directly; read through the associated `Var`.
- **Encoding**: All file I/O uses `encoding='utf-8'` explicitly. Maintain this for any new file operations.

---

## Recent Changes

| Commit | Date | Description |
|--------|------|-------------|
| `2bcb012` | 2026-02-08 | **Bug fix**: replaced iterative while-loop replacement with `str.replace(target, replacement, count)` to prevent replacement results from being re-replaced in subsequent iterations |

---

## Development Guidelines

### Making Changes

1. Read and understand existing code in `1.py` before modifying
2. Keep all changes within `1.py` unless a new file is strictly necessary
3. Test GUI interactions manually since there is no automated test suite
4. Use `tk.BooleanVar`, `tk.StringVar` for any new toggles/inputs — do not bypass Tkinter's variable system

### Adding New Features

- **New UI section**: Add a new `ttk.LabelFrame` in `create_widgets()`, placed logically between existing sections. Update `grid(row=...)` values for subsequent rows.
- **New processing step**: Add it inside the `execute_processing()` for-loop, after the existing steps
- **New settings variables**: Initialize in `__init__` alongside similar variables

### Code Style

- Follow PEP 8 naming: `snake_case` for method and variable names
- Instance variable naming pattern: `{feature}_{type}_var` for Tkinter variables (e.g., `replace_enabled_var`, `tag_count_var`)
- Dict-of-dicts pattern for complex multi-attribute objects (see `tag_files` list structure)
- Input validation with `try/except ValueError` before numeric operations — show `messagebox.showerror` on failure
- Status bar updates via `self.status_var.set(...)` to communicate state changes to the user

### What to Avoid

- Do not use `widget.get()` to read entry fields directly — use the associated `StringVar.get()`
- Do not add external dependencies — the zero-dependency design is intentional
- Do not use `global` state
- Do not skip input validation for numeric fields
- Do not change file encoding from UTF-8

---

## No Build / Test / CI Infrastructure

There is currently **no automated test suite**, **no CI/CD pipeline**, and **no linter configuration**. Manual testing by running `python3 1.py` and exercising the GUI is the only verification method.

If tests are added in the future, use `pytest` with `pytest-tk` or mock Tkinter for unit-testable logic extracted from the GUI class.
