[README.md](https://github.com/user-attachments/files/33919112/README.md)

# ELITE Password Recovery

A local, offline Windows desktop tool for recovering the password of documents
you own or are authorized to access. It tests candidate passwords against the
file's real encryption check and never modifies the original document.

![ELITE Password Recovery](docs/screenshot.png)

## Download and install

Run **`ELITE-Password-Recovery-Setup-1.0.0.exe`** and follow the wizard. The
installer is per-user, so it does not require administrator rights. It creates
Start Menu and optional desktop shortcuts.

The installer is not code-signed. Windows SmartScreen may show a warning the
first time; choose **More info → Run anyway** if you trust this build.

The portable application folder is also available at
`dist\ELITE-Password-Recovery\` (run `ELITE-Password-Recovery.exe` from there).

## Supported documents

| Format | Encryption | Notes |
|---|---|---|
| PDF | Standard Security Handler R2, R3, R4 (RC4/AES-128), R5, R6 (AES-256) | Includes user and optional owner-password testing |
| Word / Excel / PowerPoint (`.docx`, `.xlsx`, `.pptx`) | ECMA-376 Standard and Agile | Agile is intentionally slow |
| Word / Excel / PowerPoint 97-2003 (`.doc`, `.xls`, `.ppt`) | RC4, RC4 CryptoAPI, XOR obfuscation | Via msoffcrypto-tool |
| ZIP | WinZip AES-256 and traditional ZipCrypto | Verifies the smallest encrypted member |
| 7z | AES-256, with or without encrypted headers | Slower because the archive header is re-read |

Not supported: RAR archives, OpenDocument (`.odt`, `.ods`, `.odp`), PDF
certificate/public-key security, and Office files that use unsupported
extensions. Those files are reported as unsupported instead of being tested
blindly.

## Using the app

1. **Choose the document.** The format and encryption type are detected
   automatically.
2. **Pick a method.**
   - **Dictionary** — test a wordlist, the built-in top-10,000 common-password
     list, and optionally words from the filename. Mutations add case changes,
     common suffixes, leetspeak, years, and two-digit suffixes.
   - **Brute force** — test every combination of the selected characters.
     Search space grows exponentially.
   - **Mask** — describe a known shape. `?l` = a-z, `?u` = A-Z, `?d` = 0-9,
     `?s` = symbols, `?a` = printable, `?h` = hex, `?1`–`?9` = custom sets.
     Example: `?u?l?l?l?d?d?d?d`.
3. **Start recovery.** Worker processes test candidates in parallel. Progress,
   speed, elapsed time, and ETA are shown live. The recovered password appears
   in the result card and can be copied to the clipboard.
4. **Open the document normally** with the recovered password.

### Practical tips

- Start with **Dictionary + Light mutations**. It is the fastest option and
  finds most forgotten passwords.
- Add personal words (names, teams, pets, dates, old passwords) to a text file,
  one candidate per line, and select it as the wordlist.
- Keep brute-force maximum length small. Each extra character multiplies the
  work by the alphabet size.
- Office **Agile** files are deliberately expensive (about 20–100 checks per
  second on a typical CPU). More CPU threads help, but a huge search space is
  still infeasible.
- The **PDF owner password** option also tests the document's owner password,
  at roughly half the normal speed.
- Everything runs locally. There are no accounts, telemetry, or network calls.

## Command line

```powershell
.\.venv\Scripts\python.exe -m elite_recovery.cli "C:\path\file.pdf" `
    --wordlist "C:\path\words.txt" --rules light --workers 8
```

Useful flags: `--info`, `--builtin`, `--filename`, `--mask "?u?l?l?l?d?d"`,
`--brute --charset lower,digits --min 1 --max 6`, `--owner`, `--json`.

## Build from source

Requirements: Windows 10/11, Python 3.13, Inno Setup 6 (for the installer).

```powershell
python -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\build.ps1
```

`build.ps1` does two things:

1. Builds a one-file GUI executable with PyInstaller
   (`dist\ELITE-Password-Recovery.exe`).
2. Compiles the installer with Inno Setup into
   `%USERPROFILE%\Downloads\ELITE-Password-Recovery-Setup-1.0.0.exe`.

## Project layout

```
app.py                     GUI entry point and frozen-build self-test
elite_recovery/
  gui.py                   Tkinter interface
  candidates.py            Wordlists, mutations, brute force, masks
  engine.py                Multiprocessing coordinator and workers
  verifiers.py             PDF/Office/ZIP/7z verifier factory
  pdf_crypto.py            Fast PDF Standard Security Handler checks
  cli.py                   Command-line interface
  resources.py             Bundled resource and settings paths
wordlists/
  common-passwords.txt     Pwdb Top 10,000 (SecLists, MIT)
assets/                    Icon, logo, installer artwork
installer/                 Inno Setup script
tools/                     Fixture generators and GUI/engine tests
```

## Performance notes

Verification is done in C-backed libraries where possible, but the cost depends
on the format's own key-derivation design:

- PDF R2–R4: thousands of candidates per second across several cores.
- PDF R6: hundreds per second per core; AES-256 hashing dominates.
- Office Agile: tens per second; 100,000 hash iterations per candidate.
- ZIP AES: hundreds per second; PBKDF2 per candidate.
- 7z: roughly 10 per second per core because the header is re-read.

The app is CPU-only. There is no GPU/hashcat integration.

## Legal and privacy

Use ELITE Password Recovery only on documents that you own or have explicit
permission to access. Attempting to open someone else's protected files may be
illegal. You are responsible for how you use this software.

All processing happens on the local PC. No document, password, wordlist, or
usage data is uploaded. Settings are stored in
`%APPDATA%\ELITE Password Recovery\settings.json`; recovered passwords are not
saved there.

The software is provided as-is, without warranty. Recovery is not guaranteed.

## Third-party components

See `THIRD_PARTY_NOTICES.md` for licenses and links. The built-in wordlist is
the Pwdb Top 10,000 list from
[SecLists](https://github.com/danielmiessler/SecLists) (MIT).
