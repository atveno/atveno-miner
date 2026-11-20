# atveno-miner

`atveno-miner` mines Atveno on NVIDIA GPUs through a mining pool of your
choice. This repository publishes prebuilt binaries only; it contains no source
code.

- Mines on every NVIDIA card in the machine by default.
- Checks every GPU against reference vectors before it mines, and re-verifies
  every share before submitting it.

## Requirements

| | |
| --- | --- |
| Operating system | Linux x86-64 with glibc 2.34 or newer (Ubuntu 22.04 or newer). On Windows 10 or 11, run it in WSL 2: see [Mining on Windows](WINDOWS.md). There is no macOS build. |
| GPU | NVIDIA RTX 20 series (Turing) or newer. AMD and Intel GPUs are not supported. |
| NVIDIA driver | Version 580 or newer. You need only the driver, not the CUDA Toolkit. |
| Pool | The address of a mining pool that supports Atveno (`stratum+tcp://HOST:PORT`), written as `<pool-url>` below. Pools are run independently of this project; choose one yourself. |
| Wallet | An EVM address (`0x` followed by 40 hex digits), written as `<wallet>` below. |

Check the driver:

```bash
nvidia-smi
```

It must list every card and a driver version of 580 or newer. To install the driver on Ubuntu, run
`sudo ubuntu-drivers install` and reboot.

## Download

Download the miner from the
[latest release](https://github.com/atveno/atveno-miner/releases/latest):

```bash
curl -LO https://github.com/atveno/atveno-miner/releases/latest/download/atveno-miner
```

If it says `curl: command not found`, install curl, then run the download
command again:

```bash
apt update && apt install -y curl
```

This works as root. As any other user, run
`sudo apt update && sudo apt install -y curl` instead.

Make it executable and check that it runs:

```bash
chmod +x atveno-miner
```

```bash
./atveno-miner --version
```

`--version` prints the version and the build commit, for example
`atveno-miner 0.1.0 git_commit=…`. Download the miner only from this
repository's releases.

## Fill In Your Values

**Every command below contains placeholders. Replace each one with your own
value, angle brackets included, before you run the command.**

| Placeholder | Replace with | Example |
| --- | --- | --- |
| `<pool-url>` | The address published by the pool you chose | `stratum+tcp://pool.example.com:3333` |
| `<wallet>` | Your own wallet address. Rewards are paid to it, so check it carefully | `0x1234567890abcdef1234567890abcdef12345678` |
| `<worker>` | A name for this machine, so you can tell your machines apart on the pool | `rig01` |

So `--pool-login <wallet>.<worker>` becomes, for example,
`--pool-login 0x1234567890abcdef1234567890abcdef12345678.rig01`. The wallet in
the example is not a real one: never copy it, or your rewards are lost.

## Mine on GPUs

The miner uses every card the driver reports:

```bash
./atveno-miner --pool-url <pool-url> --pool-login <wallet>.<worker>
```

To mine on some cards only, list their indexes with `--gpu-devices`:

```bash
./atveno-miner --pool-url <pool-url> --pool-login <wallet>.<worker> --gpu-devices 0,2
```

Repeat `--pool-url` to add backup pools. They are tried in order when the
first one fails, and the miner returns to the first one when it recovers.

## Run in the Background

To keep the miner running after you close the terminal or SSH session, start
it with `nohup`. Its output goes to `miner.log`:

```bash
nohup ./atveno-miner --pool-url <pool-url> --pool-login <wallet>.<worker> > miner.log 2>&1 &
```

Watch the log (press `Ctrl-C` to stop watching; the miner keeps running):

```bash
tail -f miner.log
```

Stop the miner:

```bash
pkill atveno-miner
```

The miner does not start again by itself after a reboot; run the `nohup`
command again.

## Options

These are all the options.

| Option | Meaning |
| --- | --- |
| `--pool-url <URL>` | Required. Pool address, `stratum+tcp://HOST:PORT`. Repeat it to add backup pools, tried in order. |
| `--pool-login <WALLET[.WORKER]>` | Required. Your wallet and an optional worker name. A mixed-case wallet must match its EIP-55 checksum; all lower case always works. The worker name is up to 64 characters from `A-Z a-z 0-9 . _ -`. |
| `--gpu-devices <INDEXES>` | Cards to mine on, as comma-separated indexes such as `0` or `0,2`. Default: every card. If the indexes do not match the order `nvidia-smi` lists, put `CUDA_DEVICE_ORDER=PCI_BUS_ID` before the command. |
| `--threads <COUNT>` | Mine on the CPU with this many threads (at most 64) instead of on the cards. |
| `--status-bind <ADDRESS:PORT>` | Serve the miner's status (`/status`, JSON) and metrics (`/metrics`, Prometheus) on this address. Bind it to `127.0.0.1` or a private network only. |
| `--log-format <FORMAT>` | `terminal` (default) shows the view described below, in local time. `json` and `compact` print every event with all its details, in UTC; use them to collect logs. |
| `--benchmark-gpu` | Measure the selected cards and exit, without mining. |
| `-h`, `--help` / `-V`, `--version` | Print help or the version. |

For more detailed logs, put `RUST_LOG=atveno::miner=debug` before the command.

## What You Should See

When it starts, the miner prints a summary: its version, the pool, your wallet
and worker, one line per card ending in `self-test OK`, and the time zone of
the timestamps. If any card fails its self-test, the miner does not start.

After that, every line starts with the local time and a tag such as `[pool]`,
`[share]` or `[gpu0]`:

| Line | Meaning |
| --- | --- |
| `[pool]  connected to …` | Logged in to the pool. |
| `[pool]  new job  height …  share diff …` | New work, usually with every new block. |
| `warning: …` or `error: …` | Something needs attention; see [Troubleshooting](#troubleshooting). |

Every 60 seconds the miner prints a table, for example:

```text
2026-11-24 12:01:00 ── uptime 0:01:00 ─ height 216 ─ share diff 316.5M ──
 GPU  Hashrate     Temp  Fan  Power  Eff.        A / R / I
 0    312.6 MH/s   64C   40%  250W   1.25 MH/W   150 / 4 / 0
 1    332.1 MH/s   64C   40%  249W   1.33 MH/W   168 / 4 / 0
 2    309.1 MH/s   64C   40%  249W   1.24 MH/W   130 / 1 / 0
 ALL  953.8 MH/s              748W   1.28 MH/W   448 / 9 / 0   (9 stale: a new block arrived first; normal)
 GPU 0, 1, 2 held at their power limit (250 W)
```

| Column | Meaning |
| --- | --- |
| `Hashrate` | Each card's mining speed since the previous table. |
| `Temp`, `Fan`, `Power`, `Eff.` | Temperature, fan speed, power draw, and hashrate per watt. |
| `A / R / I` | Shares accepted, rejected and invalid since start. A few rejections marked stale are normal: a new block arrived before the share. |
| Last lines | A card held at its power limit is normal while mining. A card slowed down by heat or above the temperature limit needs better cooling. |

Right after start, `warning: dropping the oldest queued shares` may appear: a
GPU finds more shares than the pool accepts until the pool raises your share
difficulty. This stops by itself. The pool's own statistics are the authority
on your hash rate and earnings.

## Troubleshooting

| Symptom | Cause and fix |
| --- | --- |
| `syntax error near unexpected token`, or `pool-url: No such file or directory` | A placeholder such as `<pool-url>` was left in the command. Replace every placeholder with your own value, as in [Fill In Your Values](#fill-in-your-values). |
| `Permission denied` | The file is not executable: `chmod +x atveno-miner`. |
| `cannot execute binary file: Exec format error` | Not a Linux x86-64 machine. Check `uname -m`. |
| `version 'GLIBC_2.34' not found` | The system is older than Ubuntu 22.04. Use a newer Linux distribution. |
| `cuda gpu backend requires NVIDIA driver library libcuda.so.1` | The NVIDIA driver is missing, or the machine was not rebooted after installing it. Install the driver, reboot and check `nvidia-smi`. |
| The GPU self-test fails, or the miner stops because the CPU rejected a GPU share | The card, driver or overclock is faulty. Remove any overclock, update the driver and try again. |
| `wallet checksum mismatch` | The wallet's upper and lower case letters are wrong. Write it all in lower case. |
| `the pool refused the login` | The pool does not accept the wallet or worker name. Check both, or ask the pool. |
| `pool entry is not usable` with `connecting failed` | The pool cannot be reached. Check the address and port in `--pool-url`, and that your firewall allows outgoing connections to it. The miner keeps retrying by itself. |
| `unsupported protocol` at login | The pool speaks a different protocol version than this miner. Ask the pool which miner version it supports. |
| `UNSUPPORTED_POW_GENERATION` at login, or `upgrade the miner` | The pool needs a newer miner. Waiting does not help: download the latest release. |
| `the pool has no job source` | The pool has no work to hand out. Nothing to fix on your side; tell the pool. |
| Repeated `work was not renewed in time` | Work from the pool stopped arriving. A firewall or load balancer between you and the pool may be closing idle connections. |
| Many `invalid_pow` or `low_difficulty` rejections | The miner and the pool disagree on the protocol. Stop the miner and tell the pool, with the output of `./atveno-miner --version`. |
