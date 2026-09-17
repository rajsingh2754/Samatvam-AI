# Installing Samatvam AI on Windows

There are two ways to get Samatvam AI running. Pick the first one unless you
intend to modify the code.

| | Who it is for | What you download | What you need beforehand |
|---|---|---|---|
| **A — Installer** | Everyone: participants, clinicians, reviewers, anyone who just wants to use the app | `Samatvam AI Setup.exe` (14 MB, or 21 MB for the self-contained build) | Nothing. Not even Python. |


---

## The installer (recommended)

### 1. Download

Go to the project's **Releases** page and download **`Samatvam AI Setup.exe`**.

### 2. Run it

Double-click it. Windows SmartScreen may show *"Windows protected your PC"*
because the executable is not code-signed — click **More info → Run anyway**.
(Code signing requires a paid certificate; the SHA-256 of every release asset is
published in `SHA256SUMS.txt` on the release if you want to verify the download
instead.)

### 3. First run — the one-time setup

A small window appears explaining what is about to happen, with two choices:

- **Install PyTorch** — needed only for the *deep* models (GRU, TCN,
  TCN-Attention, CNN-LSTM, InceptionTime, gated / attention fusion). The model
  the app selects by default is a **classical** bundle
  (`random_forest_calibrated`) that does not need PyTorch at all. Leave it
  ticked for the full system; untick it to save about 200 MB and some install
  time.
  - *CPU build* (~200 MB) — works everywhere. Recommended.
  - *NVIDIA CUDA build* (~2.5 GB) — only worth it with an NVIDIA GPU.
- **Create a desktop shortcut** — on by default.

Click **Install**. Setup then, with a progress bar and a live log you can expand
with *Show details*:

1. **Prepares a Python runtime.** It looks for a Python 3.11 or 3.12 already
   installed on the machine (via the Python Launcher, the registry and `PATH`,
   probing each one rather than trusting its name) and builds a *private*
   virtual environment from it. If there is no suitable Python it downloads the
   official embeddable CPython 3.12 from python.org, so you never have to
   install Python yourself. The version range is narrow on purpose: the app's
   dependencies are pinned to the exact versions its trained models were built
   with.
2. **Installs the application files.**
3. **Downloads and installs the libraries** — PySide6, scikit-learn, OpenCV,
   MediaPipe, XGBoost, pandas/NumPy and so on. This is the long step: budget
   roughly 400–700 MB of download and 5–15 minutes on a normal connection.
4. **Downloads the trained models** and verifies their SHA-256 checksum.
5. **Creates shortcuts, registers with Windows** (so it appears in
   *Settings → Apps → Installed apps* with a working Uninstall button),
   **verifies the installation** (it actually imports the app and confirms a
   model can be selected) **and starts Samatvam AI.**

If something goes wrong, the window stays open with a plain-language
explanation, the specific thing to try, and buttons to reopen the log or retry.
Retrying is fast — finished downloads are cached.

### 4. Every launch after that

Start it from the **Start Menu** or the **desktop shortcut**. No window, no
progress bar, no downloads: the launcher checks in milliseconds that everything
is still in place and starts the app directly.

You can delete the downloaded `Samatvam AI Setup.exe` — on first run it copies
itself to the install folder and the shortcuts point at that copy.

### Where things are installed

Everything goes in one per-user folder. **No administrator rights are required
and nothing outside this folder is modified.**

```
%LOCALAPPDATA%\Samatvam AI\
├── Samatvam AI.exe      the installed launcher (what your shortcuts run)
├── app\                the application code
├── runtime\            the private Python environment
├── data\               YOUR DATA — models, session history, feedback, settings
├── cache\              downloaded archives (safe to delete any time)
├── logs\               setup and launch logs
└── state.json          what is currently installed
```

The important line is `data\`: it sits **outside** `app\`, so upgrading the
application never touches your session history, feedback database, calibration
baseline or exported CSVs.

To move the whole installation elsewhere, set the `SAMATVAM_HOME` environment
variable (or pass `--home "D:\Samatvam AI"`) before the first run.

### Upgrading from a version installed before the rename

Early builds installed to `%LOCALAPPDATA%\SamatvamAI\` (the product name run
together) and called the launcher `SamatvamAI.exe`. Both are now written
properly, with the space.

There is nothing to do: the first launch of a current build **moves the existing
installation across** — a folder rename, so it is instant and your data, models
and settings come with it. The shortcuts and the *Apps & features* entry are
rewritten to the new path, and the old launcher is deleted. You may be asked to
allow the app through SmartScreen again, because the executable's name changed.

If the relocated Python environment does not start afterwards — unusual, but
possible — the launcher notices and rebuilds it on that same run rather than
failing.

## Uninstalling

Samatvam AI registers itself with Windows, so remove it the way you remove any
other application. Three routes, all equivalent:

**1 — Settings (the normal way).**
*Start → Settings → Apps → Installed apps →* **Samatvam AI** *→ ⋯ → Uninstall*.
(On Windows 10: *Settings → Apps → Apps & features*, or *Control Panel →
Programs and Features*.)

**2 — Start Menu.** *Start → Samatvam AI →* **Uninstall Samatvam AI**.

**3 — Command line.**

```powershell
& "$env:LOCALAPPDATA\Samatvam AI\Samatvam AI.exe" --uninstall
```

All three open the same confirmation window, which lists exactly what will be
removed and how much space it frees, with one choice to make:

- **Leave the box unticked (default)** — removes the private Python
  environment, every library it installed, the application files, the download
  cache, the shortcuts and the Windows entry. **Your data is kept** in
  `%LOCALAPPDATA%\Samatvam AI\data`: session history, the feedback database,
  your settings and calibration baseline, exported CSVs and the models. Reinstall
  later and it all comes back.
- **Tick "Also delete my data"** — removes that too. This cannot be undone. If
  you have research data in there, export it first
  (*Feedback → Export evaluation report*, *History → Export CSV*).

### What gets removed, exactly

Everything Samatvam AI ever installed lives inside `%LOCALAPPDATA%\Samatvam AI\`,
so removing that folder removes all of it — including **all the Python
dependencies**, which were installed into a private environment inside it
(`runtime\env`) and never system-wide.

Not touched, because the installer never modified them:

- your own Python installation, if the environment was built from one;
- the system `PATH`, any environment variables, any services or scheduled tasks;
- anything under `Program Files`, `System32` or `HKEY_LOCAL_MACHINE`;
- pip's own download cache in `%LOCALAPPDATA%\pip\cache` (shared with your other
  Python work — delete it yourself if you want the disk space back).

The only registry value it creates is its own uninstall entry under
`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Uninstall\SamatvamAI`,
which the uninstaller deletes.

### Unattended removal

```powershell
# keeps user data, no prompts
& "$env:LOCALAPPDATA\Samatvam AI\Samatvam AI.exe" --uninstall --silent --yes

# removes everything including user data
& "$env:LOCALAPPDATA\Samatvam AI\Samatvam AI.exe" --uninstall --purge --silent --yes
```

`--purge` **requires** `--yes` when run non-interactively — deleting research
data is the one irreversible step, so it will refuse rather than assume. Exit
code `0` means fully removed, `1` means something was left behind.

### If something is left behind

The uninstaller reports any file it could not delete rather than claiming
success. The usual cause is Samatvam AI still running and holding a library
open: close it and run the uninstaller again. Failing that, deleting
`%LOCALAPPDATA%\Samatvam AI\` in File Explorer is a complete removal — the only
thing left would be the Start Menu folder and the registry entry, which
`--uninstall` cleans up even when the files are gone.

---

