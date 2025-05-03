# Pantheon of Congestion Control
The Pantheon contains wrappers for many popular practical and research
congestion control schemes. The Pantheon enables them to run on a common
interface, and has tools to benchmark and compare their performances.
Pantheon tests can be run locally over emulated links using
[mahimahi](http://mahimahi.mit.edu/) or over the Internet to a remote machine.

Our website is <https://pantheon.stanford.edu>, where you can find more
information about Pantheon, including supported schemes, measurement results
on a global testbed so far, and our paper at [USENIX ATC 2018](https://www.usenix.org/conference/atc18/presentation/yan-francis)
(**Awarded Best Paper**).
In case you are interested, the scripts and traces
(including "calibrated emulators") for running the testbed can be found in
[observatory](https://github.com/StanfordSNR/observatory).

To discuss and talk about Pantheon-related topics and issues, feel free to
post in the [Google Group](https://groups.google.com/forum/#!forum/pantheon-stanford)
or send an email to `pantheon-stanford <at> googlegroups <dot> com`.

## Disclaimer
This is research software. Our scripts will write to the file system in the
`pantheon` folder. We never run third party programs as root, but we cannot
guarantee they will never try to escalate privilege to root.

You might want to install dependencies and run the setup on your own, because
our handy scripts will install packages and perform some system-wide settings
(e.g., enabling IP forwarding, loading kernel modeuls) as root.
Please run at your own risk.

## Preparation
To clone this repository, run:

```
git clone https://github.com/StanfordSNR/pantheon.git
```

Many of the tools and programs run by the Pantheon are git submodules in the
`third_party` folder. To add submodules after cloning, run:

```
git submodule update --init --recursive  # or tools/fetch_submodules.sh
```

## Dependencies
We provide a handy script `tools/install_deps.sh` to install globally required
dependencies; these dependencies are required before testing **any** scheme
and are different from the flag `--install-deps` below.
In particular, we created the [Pantheon-tunnel](https://github.com/StanfordSNR/pantheon-tunnel)
that is required to instrument each scheme.

You might want to inspect the contents of
`install_deps.sh` and install these dependencies by yourself in case you want to
manage dependencies differently. Please note that Pantheon currently
**only** supports Python 2.7.

Next, for those dependencies required by each congestion control scheme `<cc>`,
run `src/wrappers/<cc>.py deps` to print a dependency list. You could install
them by yourself, or run

```
src/experiments/setup.py --install-deps (--all | --schemes "<cc1> <cc2> ...")
```

to install dependencies required by all schemes or a list of schemes separated
by spaces.

## Setup
After installing dependencies, run

```
src/experiments/setup.py [--setup] [--all | --schemes "<cc1> <cc2> ..."]
```

to set up supported congestion control schemes. `--setup` is required
to be run only once. In contrast, `src/experiments/setup.py` is
required to be run on every reboot (without `--setup`).

## Running the Pantheon
To test schemes in emulated networks locally, run

```
src/experiments/test.py local (--all | --schemes "<cc1> <cc2> ...")
```

To test schemes over the Internet to remote machine, run

```
src/experiments/test.py remote (--all | --schemes "<cc1> <cc2> ...") HOST:PANTHEON-DIR
```

Run `src/experiments/test.py local -h` and `src/experiments/test.py remote -h`
for detailed usage and additional optional arguments, such as multiple flows,
running time, arbitrary set of mahimahi shells for emulation tests,
data sender side for real tests; use `--data-dir DIR` to specify an
an output directory to save logs.

## Pantheon analysis
To analyze test results, run

```
src/analysis/analyze.py --data-dir DIR
```

It will analyze the logs saved by `src/experiments/test.py`, then generate
performance figures and a full PDF report `pantheon_report.pdf`.

## Running a single congestion control scheme
All the available schemes can be found in `src/config.yml`. To run a single
congestion control scheme, first follow the **Dependencies** section to install
the required dependencies.

At the first time of running, run `src/wrappers/<cc>.py setup`
to perform the persistent setup across reboots, such as compilation,
generating or downloading files to send, etc. Then run
`src/wrappers/<cc>.py setup_after_reboot`, which also has to be run on every
reboot. In fact, `test/setup.py` performs `setup_after_reboot` by
default, and runs `setup` on schemes when `--setup` is given.

Next, execute the following command to find the running order for a scheme:
```
src/wrappers/<cc>.py run_first
```

Depending on the output of `run_first`, run

```
# Receiver first
src/wrappers/<cc>.py receiver port
src/wrappers/<cc>.py sender IP port
```

or

```
# Sender first
src/wrappers/<cc>.py sender port
src/wrappers/<cc>.py receiver IP port
```

Run `src/wrappers/<cc>.py -h` for detailed usage.

## How to add your own congestion control
Adding your own congestion control to Pantheon is easy! Just follow these
steps:

1. Fork this repository.

2. Add your congestion control repository as a submodule to `pantheon`:

   ```
   git submodule add <your-cc-repo-url> third_party/<your-cc-repo-name>
   ```

   and add `ignore = dirty` to `.gitmodules` under your submodule.

3. In `src/wrappers`, read `example.py` and create your own `<your-cc-name>.py`.
   Make sure the sender and receiver run longer than 60 seconds; you could also
   leave them running forever without the need to kill them.

4. Add your scheme to `src/config.yml` along with settings of
   `name`, `color` and `marker`, so that `src/experiments/test.py` is able to
   find your scheme and `src/analysis/analyze.py` is able to plot your scheme
   with the specified settings.

5. Add your scheme to `SCHEMES` in `.travis.yml` for continuous integration testing.

6. Send us a pull request and that's it, you're in the Pantheon!
7. 


# 📘 Pantheon Congestion Control Evaluation

This repository contains my implementation and analysis of congestion control protocols using the [Pantheon framework](https://github.com/StanfordSNR/pantheon) and [Mahimahi](https://github.com/ravinet/mahimahi) network emulator. The goal is to benchmark different congestion control algorithms under controlled conditions and evaluate them based on throughput, latency, and packet loss.

---

## 🔧 Environment Setup

### 1. Clone the Repository
```bash
git clone https://github.com/SreejaReddytr/pantheonredo.git
cd pantheonredo
```

### 2. Install Dependencies
```bash
sudo apt update
sudo apt install -y \
    build-essential python2.7 python-pip \
    git libssl-dev libprotobuf-dev protobuf-compiler \
    mahimahi texlive-full cmake g++ \
    libboost-all-dev libgflags-dev libgoogle-glog-dev
```

### 3. Enable IP Forwarding
```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

### 4. Setup Pantheon
```bash
cd src
python2 experiments/setup.py --setup
python2 experiments/setup.py --all
```

---

## 🚦 Experiment Setup

We used **Mahimahi** to simulate two different network environments:

### ✨ Scenario 1: Low Latency, High Bandwidth
- **50 Mbps** bandwidth
- **10 ms** round-trip delay
- **Duration:** 60 seconds
- **Protocols:** `cubic`, `fillp`, `vivace`

```bash
python2 experiments/test.py local \
  --schemes "cubic fillp vivace" \
  --data-dir experiments/test_results_scenario1 \
  --runtime 60 \
  --uplink-trace traces/50mbps.trace \
  --downlink-trace traces/50mbps.trace \
  --prepend-mm-cmds "mm-delay 5" \
  --extra-mm-link-args "--uplink-queue=droptail --downlink-queue=droptail --uplink-queue-args=packets=500 --downlink-queue-args=packets=500"
```

### 🚨 Scenario 2: High Latency, Low Bandwidth
- **1 Mbps** bandwidth
- **200 ms** round-trip delay
- **Duration:** 60 seconds
- **Protocols:** `cubic`, `fillp`, `vivace`

```bash
python2 experiments/test.py local \
  --schemes "cubic fillp vivace" \
  --data-dir experiments/test_results_scenario2 \
  --runtime 60 \
  --uplink-trace traces/1mbps.trace \
  --downlink-trace traces/1mbps.trace \
  --prepend-mm-cmds "mm-delay 100" \
  --extra-mm-link-args "--uplink-queue=droptail --downlink-queue=droptail --uplink-queue-args=packets=500 --downlink-queue-args=packets=500"
```

---

## 📊 Analysis & Graphs

After running the experiments, generate analysis and graphs:

```bash
python2 analysis/analyze.py --data-dir experiments/test_results_scenario1
python2 analysis/report.py --data-dir experiments/test_results_scenario1

python2 analysis/analyze.py --data-dir experiments/test_results_scenario2
python2 analysis/report.py --data-dir experiments/test_results_scenario2
```

Each scenario folder will contain:
- Delay and throughput plots for each scheme
- Summary comparison charts
- `pantheon_report.pdf` with auto-generated analysis

---

## 🧠 Summary of Observations

- **Cubic** was the most aggressive in both scenarios, delivering high throughput but resulting in high delay and packet loss, especially under constrained bandwidth.
- **Vivace** attempted to push bandwidth like Cubic but suffered from extreme queuing delays, reaching RTTs of over 9 seconds.
- **FillP** demonstrated moderate performance with a good balance between delay and throughput.

For more detailed analysis, refer to the final report PDF below.

---

## 📁 Project Structure

```
pantheonredo/
├── src/
│   └── experiments/
│       ├── test_results_scenario1/
│       ├── test_results_scenario2/
│       └── traces/
├── analysis/
│   └── report.py
├── README.md
└── pantheon_report.pdf (generated)
```

---

## 📎 Final Report & Submission

- ✅ [Final Report PDF](link-to-your-generated-pantheon-report.pdf)
- ✅ [GitHub Repo](https://github.com/SreejaReddytr/pantheonredo)

---

## 🤝 Acknowledgments

- Stanford Network Research Group – Pantheon framework
- MIT – Mahimahi emulator
- My peers and LLMs that helped troubleshoot Python 2.7, YAML errors, and build problems on modern systems.

---

## 📌 Notes

- Use **Python 2.7** only. Pantheon does not support Python 3.x.
- Mahimahi must be compiled from source if not available via apt.
- This repo is made public for reproducibility and grading.

