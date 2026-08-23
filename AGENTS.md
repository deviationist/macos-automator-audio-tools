# AGENTS.md

Guidance for AI agents working in this repository.

## What this repo is

Four macOS Automator applications that wrap FFmpeg (and `mp3val`) for DJ audio
prep. There is no build step and no source tree — each `.app` **is** the
deliverable, committed to git as a bundle.

| App | Purpose |
| --- | --- |
| `FFmpeg Audio Converter.app` | FLAC/WAV → AIFF, M4A → MP3 (AAC) or AIFF (ALAC) |
| `FFmpeg Audio Rate Downsampler.app` | Re-encode to a chosen max sample rate, same container |
| `FFmpeg Audio Normalizer.app` | `loudnorm` normalization, same container |
| `Audio File Evaluator.app` | Find corrupt MP3s with `mp3val`, offer to trash them |

## Where the logic lives

All behaviour is in `<App>.app/Contents/document.wflow` — an **XML plist** holding
the Automator action list. Everything else in the bundle (`MacOS/`, `Resources/`,
`_CodeSignature/`) is Apple's stock application stub; never hand-edit it.

Read the actions out of a workflow like this:

```bash
plutil -extract actions json -o - "FFmpeg Audio Converter.app/Contents/document.wflow" \
  | python3 -c "
import json,sys
for a in json.load(sys.stdin):
    act = a['action']
    print('---', act.get('ActionName'))
    for k, v in act.get('ActionParameters', {}).items():
        print(f'[{k}] {v}')
"
```

### Editing rules

- Edit `document.wflow` as **text**, with an exact-match replacement (Python
  `str.replace` with a `count == 1` assertion). Do **not** round-trip it through
  `plistlib.dump` — that reformats the whole file and produces an unreadable diff.
- Inside the XML, shell operators are entity-escaped: `>` is `&gt;`, `&` is `&amp;`,
  `<` is `&lt;`. Your search string must use the escaped form; so must the
  replacement.
- Indentation inside the scripts is **tabs**.
- Run `plutil -lint <file>` after every edit. A malformed plist means the app
  silently refuses to open in Automator.
- Modifying `document.wflow` invalidates the bundle's ad-hoc signature. This is
  already the case on `main` and does not stop the apps from running; do not try
  to "fix" it by re-signing.

## Testing changes

Never test by double-clicking the app — extract the script and run it directly.
Stub out the two side-effecting commands so tests touch neither the user's Trash
nor the UI. Because every script starts with
`export PATH=$HOME/bin:$HOME/.local/bin:/usr/local/bin:/opt/homebrew/bin:$PATH`,
pointing `HOME` at a fake home with `bin/trash` and `bin/osascript` shims is
enough to intercept them:

```bash
mkdir -p /tmp/fakehome/bin
printf '#!/bin/zsh\necho "TRASHED: $@"\n'  > /tmp/fakehome/bin/trash
printf '#!/bin/zsh\ncat >/dev/null; echo "ALERT: $2"\n' > /tmp/fakehome/bin/osascript
chmod +x /tmp/fakehome/bin/*
HOME=/tmp/fakehome zsh extracted.zsh true ./fixture.m4a   # $1 = deleteFilesAfterConversion
```

Build fixtures with FFmpeg's `lavfi` source rather than shipping binaries:

```bash
ffmpeg -f lavfi -i "sine=frequency=440:duration=1" -c:a alac fixture.m4a
```

Cover the cases that have actually caused bugs here: both codecs in a `.m4a`,
filenames with spaces / quotes / non-ASCII, a truncated or zero-byte file, a
pre-existing output file, and both answers to the delete-source prompt.
Also run `zsh -n` on the extracted script.

## Conventions the scripts follow

- Shell is `/bin/zsh`; each script re-exports `PATH` because Automator does not
  inherit the user's login environment.
- Dependencies are checked with `hash <cmd>` and installed via `brew` in a
  dedicated early action; Homebrew's own absence aborts with an AppleScript dialog.
- The final shell action receives `deleteFilesAfterConversion` as `$1` (from the
  alert dialog) and the dropped files as `"${@:2}"`.
- FFmpeg is always invoked `-y -nostdin` with `2>/dev/null`. Both flags matter:
  Automator gives the script no usable stdin, so without them a prompt for
  overwrite confirmation makes the run fail silently.
- The source file is trashed **only** inside the `if ffmpeg ...; then` success
  branch, and only via `trash` (never `rm`).
- When the correct action cannot be determined, skip the file and keep the
  original. Guessing is worse than doing nothing, because a wrong guess can
  transcode a lossless file and then trash the source.

## Cross-cutting invariants

These hold across all four apps. Breaking one is a regression, not a style choice.

- **Extensions are matched case-insensitively.** Lowercase the extension before
  comparing (`"${${filePath##*.}:l}"`), never pattern-match `*.mp3` directly —
  that silently skips `.MP3` and `.M4A`. Derive output names from
  `"${filePath%.*}"`, not from `${filePath/%.flac/.aiff}`-style substitution,
  which is case-sensitive in the same way.
- **Every app exports the same PATH**:
  `$HOME/bin:$HOME/.local/bin:/usr/local/bin:/opt/homebrew/bin:$PATH`.
  `/opt/homebrew/bin` is required — Automator does not inherit the login shell's
  environment, so without it nothing works on Apple Silicon.
- **FFmpeg is always `-y -nostdin`.** See the note below on why this is not
  cosmetic.

## Known quirks (deliberate, or not yet fixed)

- `Audio File Evaluator.app` handles only MP3, by design (`mp3val` is MP3-only).
- The Converter transcodes lossless ALAC to MP3 only as a fallback for
  non-ALAC codecs; an unreadable codec is skipped rather than guessed.

## Why `-y -nostdin` is load-bearing

Verified against ffmpeg 9.0.1: when the output file already exists and stdin is
not interactive (which is always, under Automator), ffmpeg prints
`Not overwriting - exiting` and **exits 0**. A success exit code means an
`if ffmpeg ...; then trash "${filePath}"; fi` wrapper trashes the source without
having written anything. Any new ffmpeg call in this repo must pass both flags.

## Docs

`README.md` is user-facing (installation, usage, per-app behaviour). Update its
conversion table whenever a script's input/output mapping changes.
