# Lite

*Lite* (`lite_feeder`) is a lightweight monitoring daemon that runs inside a container, reads process and system statistics from the Linux `/proc` filesystem and sends them to [OpenTSDB](http://opentsdb.net/) (e.g., the time-series database of [BDWatchdog](https://github.com/UDC-GAC/BDWatchdog)). It replaces the atop-based pipeline of BDWatchdog's MetricsFeeder, sending the same metrics with the same names, tags and units, and it also reports the CPU wait of each process.

## Metrics

| Group | Metrics                                                              | Tags               |
| ----- | -------------------------------------------------------------------- | ------------------ |
| CPU   | `sys.cpu.usage` (%)                                                  | host               |
| cpu   | `sys.cpu.kernel`, `sys.cpu.user`, `sys.cpu.idle`, `sys.cpu.wait` (%) | host, core         |
| MEM   | `sys.mem.free` (MB), `sys.mem.usage` (%)                             | host               |
| SWP   | `sys.swap.free` (MB)                                                 | host               |
| NET   | `sys.net.in.mb`, `sys.net.out.mb` (Mbit/s), `sys.net.usage` (%)      | host, device       |
| PRC   | `proc.cpu.user`, `proc.cpu.kernel` (ticks/s)                         | host, pid, command |
| PRM   | `proc.mem.virtual`, `proc.mem.resident` (MB)                         | host, pid, command |
| PRD   | `proc.disk.reads.mb`, `proc.disk.writes.mb` (MB/s)                   | host, pid, command |
| PRW   | `proc.cpu.wait` (ticks/s)                                            | host, pid, command |

`proc.cpu.wait` is the time that the threads of a process were runnable but waiting for a CPU (their run-queue delay in `/proc/<pid>/task/<tid>/schedstat`), in the same units as `proc.cpu.user`. Added up for the processes of a container, it gives its CPU pressure as `wait / (user + kernel + wait)`, with either cgroup v1 or v2.

## Build

```bash
gcc -O2 -o lite_feeder lite_feeder.c
```

## Usage

```text
lite_feeder [-i interval] [-H opentsdb_host] [-P opentsdb_port] [-g groups] [-s] [-c count]
```

| Option     | Description                                                                                                                                                 |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `-i`       | Sampling interval in seconds (default: 5). Samples are aligned to multiples of the interval.                                                               |
| `-H`, `-P` | OpenTSDB host and port (default: `OPENTSDB_URL` and `OPENTSDB_PORT` in `$BDWATCHDOG_PATH/services_config.yml`, by default in `/opt/BDWatchdog`, or `opentsdb:4242`). |
| `-g`       | Comma-separated groups of metrics (default: all of them).                                                                                                   |
| `-s`       | Print the data points as JSON, one per line, instead of sending them.                                                                                       |
| `-c`       | Stop after this number of samples (default: run forever).                                                                                                   |

Examples:

```bash
lite_feeder -i 1 -H 10.0.0.1 -P 4242   # Send every metric each second to the OpenTSDB at 10.0.0.1:4242
lite_feeder -i 1 -g PRC,PRW            # Only per-process CPU usage and CPU wait
lite_feeder -i 1 -s -c 5               # Print five samples instead of sending them
```
