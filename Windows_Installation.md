# Installing Samatvam AI on Windows

There are two ways to get Samatvam AI running. Pick the first one unless you
intend to modify the code.

| | Who it is for | What you download | What you need beforehand |
|---|---|---|---|
| **A — Installer** | Everyone: participants, clinicians, reviewers, anyone who just wants to use the app | `Samatvam AI Setup.exe` (14 MB, or 21 MB for the self-contained build) | Nothing. Not even Python. |
| **B — Source checkout** | Developers and researchers who will retrain models or change the code | `git clone` + `py tools\fetch_models.py` | Python 3.11 or 3.12 |

---

## A — The installer (recommended)

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

## B — Source checkout (developers)

> **Use `py`, not `python`.** On many Windows machines `python` on `PATH`
> resolves to an unrelated interpreter (this project's own dev machine has a
> Python 2.7 from another tool shadowing it, which fails with a confusing
> `SyntaxError`). The Windows Python Launcher `py` always finds the real
> Python 3.x. Check with `py -c "import sys; print(sys.executable)"`.

```powershell
git clone https://github.com/OWNER/REPO.git
cd REPO

# 1. Dependencies for the detection app
py -m pip install -r detection_system\requirements.txt

# 2. Trained models (not in git — see docs/Release_Process.md)
py tools\fetch_models.py

# 3. Run
py -m detection_system.main
```

For the Streamlit research/training dashboard, install the full stack instead
(`py -m pip install -r requirements.txt`) and run
`py -m streamlit run research_system\app.py`.

You can also exercise the installer itself straight from the checkout, which is
the quickest way to test first-run changes without building an executable:

```powershell
py installer\launcher_main.py --status
py installer\launcher_main.py --offline --home %TEMP%\samatvam-test
```

---

## Command-line reference

Everything below works on `Samatvam AI Setup.exe`, on the installed
`%LOCALAPPDATA%\Samatvam AI\Samatvam AI.exe`, and on
`py installer\launcher_main.py`.

> **Reading the output.** The executable is a GUI application, so it has no
> console of its own. Run from PowerShell it borrows the shell's console — but
> because the shell does not wait for a GUI program, your prompt comes back
> before the output does. For clean, scriptable output use `--silent`, or wait
> for it explicitly:
>
> ```powershell
> & ".\Samatvam AI.exe" --silent --status
> Start-Process -Wait ".\Samatvam AI.exe" -ArgumentList "--silent","--status"
> ```
>
> Double-clicked with no console at all, informational output appears in a small
> window instead. Set `SAMATVAM_NO_GUI=1` to suppress that in automation.

| Flag | What it does |
|------|--------------|
| *(none)* | Install anything missing, then start the app |
| `--status` | Print what is installed, what is pending, and the package versions |
| `--silent` | Run on the console with no window (unattended deployment, scripts, CI) |
| `--setup-only` | Install but do not start the app |
| `--launch-only` | Start the app; fail rather than install anything |
| `--repair` | Re-run every step, keeping the download cache and your data |
| `--reinstall` | Forget the recorded state and set everything up from scratch |
| `--force deps` | Re-run one step (`runtime`, `payload`, `deps`, `models`, `finalize`, `all`) |
| `--torch cpu\|cu126\|none` | Choose the PyTorch build, or skip it |
| `--offline` | Never download; use assets found next to the installer |
| `--models-zip PATH` | Install the models from a local archive (`.tar.xz` or `.zip`) |
| `--app-zip PATH` | Install the app code from a local archive |
| `--embedded-python` | Always download a private Python instead of using an installed one |
| `--home DIR` | Install somewhere other than `%LOCALAPPDATA%\Samatvam AI` |
| `--no-shortcuts` / `--no-desktop-shortcut` | Skip shortcut creation |
| `--uninstall` [`--purge`] | Remove the installation (optionally including your data) |
| `--yes` / `-y` | Skip the confirmation. Required for an unattended `--purge` |
| `--verbose` | Echo all command output to the console |

### Unattended / lab deployment

Installing on a room of machines, with no interaction and no per-machine
download:

```powershell
# once, on a machine with internet access
py tools\package_release.py --version 1.0.0
py installer\build_launcher.py

# then, on each machine (the models archive sits next to the exe)
& ".\Samatvam AI Setup.exe" --silent --offline --torch cpu --no-desktop-shortcut
```

`--silent` exits with `0` on success, `1` on failure and `2` if cancelled, so it
drops straight into a deployment script.

---

## Troubleshooting

**"Windows protected your PC" / SmartScreen.** Expected: the exe is unsigned.
*More info → Run anyway*, or verify the SHA-256 against the release's
`SHA256SUMS.txt`.

**Antivirus quarantines the installer.** PyInstaller executables are a common
false positive. Add an exclusion for `%LOCALAPPDATA%\Samatvam AI`, or use the
source-checkout route.

**Setup fails while installing libraries.** Almost always the network. The hint
in the window names the actual cause. Behind a corporate proxy, set the proxy
before launching and allow `pypi.org`, `files.pythonhosted.org`, `github.com`
and `objects.githubusercontent.com`:

```powershell
$env:HTTPS_PROXY = "http://proxy.example.com:8080"
```

**"No compatible wheel exists for this Python version."** A Python newer than
the libraries support was picked. Force the known-good runtime:

```powershell
& "$env:LOCALAPPDATA\Samatvam AI\Samatvam AI.exe" --embedded-python --repair
```

**The app starts and immediately closes.** The launcher captures this and shows
the reason. The two logs to look at are
`%LOCALAPPDATA%\Samatvam AI\logs\app-launch.log` (startup output) and
`%LOCALAPPDATA%\Samatvam AI\data\logs\detection_system_*.log` (the app's own
log).

**No webcam facial analysis.** The app deliberately keeps working with
behavioural signals only. Check `--status` for unavailable optional modules: if
`mediapipe` is listed, reinstall it with `--force deps`. Facial analysis also
needs `detection_system\assets\face_landmarker.task`, which ships with the app.

**"No trained models were installed."** The models asset could not be reached.
Download `samatvam-models-<version>.tar.xz` from the release, put it in the
same folder as the installer, and run `--force models --offline`.

**"… cannot be downloaded: this installer was built before its GitHub
repository was set."** The installer was built from a checkout whose
`release_manifest.json` still had the `OWNER/REPO` placeholder, so there is no
release to download the models from. Everything else is already installed — only
the models step is outstanding — so supply the file locally and it finishes in
seconds:

```powershell
& ".\Samatvam AI Setup.exe" --models-zip <path to samatvam-models-1.0.0.tar.xz>
```

or drop that zip beside the installer and run it again. To stop it recurring,
either set the repository (`py tools\package_release.py --repository owner/repo`)
or build a self-contained installer with
`py installer\build_launcher.py --embed-models`.

**Everything is confusing — start clean.** `--reinstall` keeps your data;
`--uninstall --purge` removes it too.

---

## What the installer does *not* do

Worth being explicit, since it runs without administrator rights:

- it does not write to `Program Files`, `System32` or the Windows registry;
- it does not install a service, a driver or a startup task;
- it does not modify the system `PATH` or any existing Python installation;
- it does not send any telemetry. The only network requests are to
  `pypi.org` / `files.pythonhosted.org` (libraries), `github.com` (the app and
  model assets) and, only if no suitable Python exists, `python.org`.

Samatvam AI is a research instrument for stress awareness and mindfulness
guidance. It is not a medical device.
