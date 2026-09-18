# Setting up a machine for the SCSE3040 lab practicals

*Do this once, before Practical P01. It takes about fifteen minutes on a good
connection.*

These instructions are written for a **Windows laptop with 8 GB of RAM**, which
is what most students in this course have. Notes for macOS and Linux are at the
bottom.

---

## What you are installing, and why

| | Why |
|---|---|
| **Python 3.12 or newer** | The language everything is written in. |
| **A virtual environment** | A private folder of libraries for this course only, so installing something here cannot break another project. This is Practical P01's whole subject. |
| **Git** | Saves every version of your work. Used from P01 onwards. |
| **Jupyter** | Opens and runs the `.ipynb` notebooks. |
| **Docker Desktop** | **Only needed for P08.** Skip it until then. |

---

## 1. Install Python

Download the installer from <https://www.python.org/downloads/> and run it.

> **On the very first screen, tick "Add python.exe to PATH".** If you miss it,
> nothing else in this guide will work and you will have to reinstall.

Check it worked. Open **PowerShell** (press the Windows key, type `powershell`,
press Enter) and run:

```powershell
python --version
```

You should see `Python 3.12.x` or higher.

> **If a Microsoft Store window opens instead**, Windows has given you a stub
> rather than a real Python. Go to *Settings -> Apps -> Advanced app settings ->
> App execution aliases* and turn **off** both `python.exe` and `python3.exe`,
> then try again.

## 2. Install Git

Download from <https://git-scm.com/download/win> and run it. Accept every
default. Then check:

```powershell
git --version
```

## 3. Get the course files

```powershell
cd C:\
git clone <the repository address your instructor gives you> MLOps
cd MLOps
```

## 4. Make the virtual environment

From inside the `MLOps` folder:

```powershell
python -m venv .venv
```

That creates a folder called `.venv`. It is your private Python for this
course. You never edit anything inside it.

## 5. Install the libraries

```powershell
.venv\Scripts\python -m pip install --upgrade pip
.venv\Scripts\python -m pip install -r requirements-lock.txt
.venv\Scripts\python -m pip install jupyterlab nbformat ipykernel httpx
```

This downloads about 400 MB. Be patient once; it is not repeated.

`requirements-lock.txt` pins exact versions, so everybody in the room gets
identical libraries. That is the point of it, and P01 explains why.

## 6. Register the notebook kernel

```powershell
.venv\Scripts\python -m ipykernel install --sys-prefix --name python3 --display-name "Python 3 (SCSE3040)"
```

This tells Jupyter which Python to run notebooks with. Without it, notebooks
may silently use a different Python and half the imports will fail.

## 7. Check everything

```powershell
cd labs\P01-workbench
..\..\.venv\Scripts\python -m jupyter lab P01.ipynb
```

Your browser opens with the first practical. Run the first two cells.

**You are ready when Step 0 prints `All good.` and Step 1 prints
`Inside .venv : True`.**

If `Inside .venv` says `False`, you started Jupyter with the wrong Python.
Close it and use the exact command above, including the `..\..\.venv\Scripts\`
part.

---

## 8. Docker Desktop --- only before Practical P08

Do **not** install this at the start of the semester. It runs in the
background, takes about 2 GB of memory and will slow down every other lab.

When P08 approaches:

1. Download from <https://www.docker.com/products/docker-desktop/> and install.
2. Restart your laptop when it asks.
3. Start Docker Desktop and wait until it says **Engine running**.
4. Check it:

   ```powershell
   docker version
   ```

   You need to see **both** a `Client:` and a `Server:` section. Client only
   means the engine is not running yet.

### Making Docker survive on 8 GB

Open Docker Desktop *Settings -> Resources* and set **Memory to 2 GB**. Then,
on lab day, close Chrome and anything else large before you start. Docker plus
a browser plus Jupyter on 8 GB is genuinely tight.

### For the instructor, before the P08 session

Run this on every lab machine the day before:

```powershell
docker pull python:3.12-slim
```

Thirty students downloading the base image simultaneously will saturate the
campus connection and the lab will stall. Pre-pulling makes their first build
take seconds instead of minutes.

---

## macOS and Linux

Everything is the same except the paths. Use forward slashes and
`.venv/bin/` instead of `.venv\Scripts\`:

```bash
python3 -m venv .venv
.venv/bin/python -m pip install -r requirements-lock.txt
.venv/bin/python -m pip install jupyterlab nbformat ipykernel httpx
.venv/bin/python -m ipykernel install --sys-prefix --name python3 --display-name "Python 3 (SCSE3040)"
```

On macOS install Python and Git with Homebrew (`brew install python git`) or
from the official sites. On Linux both are almost certainly present already.

---

## When something goes wrong

| Symptom | Cause and fix |
|---|---|
| `python` opens the Microsoft Store | The Store stub is intercepting it. Turn off the app execution aliases (step 1). |
| `'python' is not recognized` | "Add to PATH" was not ticked. Reinstall Python and tick it. |
| `ModuleNotFoundError` for a library you installed | You are running the wrong Python. Always use `.venv\Scripts\python`, never a bare `python`. |
| `Inside .venv : False` in P01 | Jupyter was launched from the wrong Python. Use the full command in step 7. |
| Jupyter shows no `Python 3 (SCSE3040)` kernel | Step 6 was skipped. Run it, then restart Jupyter. |
| `pip` fails with an SSL or proxy error | Campus network. Try a phone hotspot, or ask for the offline wheel bundle. |
| Everything is very slow | Close Chrome. On 8 GB, a browser with twenty tabs is your real problem. |

If you are still stuck, bring the **exact error text** to the lab --- a
screenshot of the red block is enough. Do not retype it from memory.
