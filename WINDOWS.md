# Mining on Windows

`atveno-miner` is a Linux program; there is no `.exe`. On Windows it runs in
WSL 2 (Windows Subsystem for Linux), which runs Ubuntu alongside Windows and
gives it your NVIDIA cards through the normal Windows driver.

## Requirements

| | |
| --- | --- |
| Windows | Windows 11, or Windows 10 version 21H2 or newer, on a 64-bit x86 PC. Windows on ARM is not supported. |
| Virtualization | Turned on in the BIOS or UEFI settings (named Intel VT-x, AMD-V or SVM). Most PCs have it on already. |
| GPU | NVIDIA RTX 20 series (Turing) or newer. |
| NVIDIA driver | The normal Windows driver, version 580 or newer. WSL uses this driver; install no other. |
| Pool and wallet | A pool address and a wallet, as in the [README](README.md#requirements). |

## 1. Update the NVIDIA Driver

Install the latest driver for your card from NVIDIA's website or the NVIDIA
app, then restart Windows.

The Windows commands in this guide work in Command Prompt (`cmd`) and in
PowerShell; use either. Open one (Start menu, type `cmd` or `PowerShell`)
and check the driver:

```text
nvidia-smi
```

It must list every card and a driver version of 580 or newer.

## 2. Install WSL and Ubuntu

Installing WSL changes Windows, so this step needs an administrator window.
Type `cmd` or `PowerShell` in the Start menu, right-click it and choose
**Run as administrator**. Every other Windows command in this guide works in
a normal window. Run:

```text
wsl --install
```

On a PC without WSL, this first run installs only WSL and asks you to restart
Windows. After the restart, run `wsl --install` again: this time it installs
Ubuntu. If WSL was already there, the first run installs Ubuntu directly.

When Ubuntu is installed, it asks you to create a user name and password.
They are for Ubuntu only; choose any.

Check that Ubuntu runs on WSL 2:

```text
wsl -l -v
```

The list must show Ubuntu with `2` in the `VERSION` column. If it says that
no distribution is installed, run `wsl --install` again.

## 3. Open Ubuntu

From here on, every command runs in Ubuntu, not in Command Prompt or
PowerShell. Open **Ubuntu** from the Start menu; the prompt ends with `$`.

In the Ubuntu window, `Ctrl-V` does not paste. Right-click to paste a
command. To copy text, select it with the mouse, then right-click.

Check that Ubuntu sees your cards:

```bash
nvidia-smi
```

It must list the same cards as in Windows. Do not install an NVIDIA driver
inside Ubuntu, for example with `ubuntu-drivers` or `apt install nvidia-…`:
WSL already uses the Windows driver, and a second driver breaks it.

## 4. Download

Download the miner from the
[latest release](https://github.com/atveno/atveno-miner/releases/latest):

```bash
curl -LO https://github.com/atveno/atveno-miner/releases/latest/download/atveno-miner
```

Make it executable and check that it runs:

```bash
chmod +x atveno-miner
```

```bash
./atveno-miner --version
```

## 5. Fill In Your Values

**Every command below contains placeholders. Replace each one with your own
value, angle brackets included, before you run the command.**

| Placeholder | Replace with | Example |
| --- | --- | --- |
| `<pool-url>` | The address published by the pool you chose | `stratum+tcp://pool.example.com:3333` |
| `<wallet>` | Your own wallet address. Rewards are paid to it, so check it carefully | `0x1234567890abcdef1234567890abcdef12345678` |
| `<worker>` | A name for this machine, so you can tell your machines apart on the pool | `rig01` |

The wallet in the example is not a real one: never copy it, or your rewards
are lost. Pools are run independently of this project; choose one yourself.

## 6. Mine

The miner uses every card:

```bash
./atveno-miner --pool-url <pool-url> --pool-login <wallet>.<worker>
```

To mine on some cards only, list their indexes with `--gpu-devices`:

```bash
./atveno-miner --pool-url <pool-url> --pool-login <wallet>.<worker> --gpu-devices 0,2
```

Press `Ctrl-C` to stop. The [README](README.md#what-you-should-see) explains
what the miner prints, and lists [all options](README.md#options).

The `Fan` column shows `-` when the card does not report its fan speed, as
on many laptops. Mining is not affected.

## Keep It Running

- Keep the Ubuntu window open while mining; you can minimize it. Closing it
  stops the miner, even one started with `nohup`.
- Stop Windows from sleeping: in **Settings → System → Power** (on laptops
  **Power & battery**), set the time before sleep when plugged in to
  **Never**.
- After Windows restarts, for example for an update, open Ubuntu and run the
  mining command again.

## Troubleshooting on Windows

| Symptom | Cause and fix |
| --- | --- |
| `wsl --install` fails with `0x80370102` or a message about virtualization | Virtualization is off. Turn on Intel VT-x, AMD-V or SVM in the BIOS or UEFI settings, then run `wsl --install` again. |
| `wsl` is not recognized | Windows is too old. Install the Windows updates, or upgrade to Windows 11. |
| `nvidia-smi: command not found` in Ubuntu, or `cuda gpu backend requires NVIDIA driver library libcuda.so.1` | Ubuntu cannot reach the Windows driver. Update the Windows NVIDIA driver, run `wsl --update` and then `wsl --shutdown` in Command Prompt or PowerShell, and open Ubuntu again. |
| `version 'GLIBC_2.34' not found` | The Ubuntu in WSL is older than 22.04. In Command Prompt or PowerShell, run `wsl --install -d Ubuntu-24.04`, then use **Ubuntu 24.04** from the Start menu. |
| Anything else | See [Troubleshooting](README.md#troubleshooting) in the README. |
