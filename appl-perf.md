# Application Performance Tools on Linux

## Linux Command Line Tools

* iostat

## Valgrind Suite of Tools

### Valgrind References

[C++ Performance Analysis & Profiling Tools](https://kusemanohar.info/2012/08/13/c-performance-analysis-profiling-tools/)

## VTune Suite of Tools

## Perf

Perf performance analysis utilizes hardware counters (when available - not so on class 2 hypervisors such as VirtualBox).    Install via:

    % sudo apt-get install linux-tools-common linux-tools-generic linux-tools-`(uname -r)`

This installs a version of perf that is compatible wth the running linux version.

Applications should be built with '-g' to allow perf to read the DWARF information regarding symbols and variables.

Note:  There are installation options to allow perf to be used in user-space.  Otherwise, perf must be run as root:

    % sudo perf .....

Select a process/program to analyze:

    % perf record <program>
    % perf record <PID>
    % perf record -a   ## profile every process until you hit ctrl-C

Important command line arguments:

    -F:  sampling frequency
    -g:  record stack traces
    -e:  choose events to record

Summary usage:
    % perf --help
    % perf stat <program>
    % perf record <program>  ## saves analysis data to 'perf.data' in current directory
    % perf report -i perf.data

The following use 'perf.data'

    % perf report
    % perf annotate
    % perf script

General:
    % perf list   ## shows all events

    % perf stat -e <eventlist> <program>  ## event name from 'perf list'
    % perf stat -ddd <program>  ## detailed CPU/hardware-level events
    % perf stat -e cache-references,cache-misses <program>  ## cache performance

    % perf trace <program>  ## analyze system calls
    % perf record -e cpu-clock,faults <program>   ## analyze cpu time and faults
    % perf record -e syscalls:sys_enter_connect -ag  ## trace programs that connect to server

    % perf annotate --stdio <program>  ## generate source debug when compiled with -ggdb

    %perf probe -x <executable> --add 'prb=func:4 xxx zzz'   ##  place probe on line 4 of func
    %perf probe -x <executable> --del "*"   ## delete all probes (necessary after build)

### Flamegraph Flow

Flamegraphs allow visual analysis of CPU, memory, Off CPU, Network, Disk, and Differential (comparison between commits) of application and kernel performance.

Install via:  

    % gitclone https://github.com/brendangregg/FlameGraph.git

Flow to get the flamegraph execution profile of an application:

    % perf record -F 99 -a -g <CMD>
    % perf script > out.perf
    % ./FlameGraph/stackcollapse.pl out.perf > out.folded
    % ./FlameGraph/flamegraph.pl out.folded > OUT.svg
    % firefox OUT.svg

To filter based on a function call and below:

    % grep <funcname> out.perf | ./FlameGraph/flamegraph.pl > FUNCNAME.svg
    % firefox FUNCNAME.svg

Note:  Differential flamegraphs are useful for debugging CPU performance regressions from between checkins.  See [Brendan Gregg's Blog](http://www.brendangregg.com/blog/2014-11-09/differential-flame-graphs.html)

### Off-CPU Performance Anaysis

Off-CPU performance analysis is a generic approach for analyzing blocking events.  Measure the amount of time a thread spent off-CPU such as waiting (sleeping) on I/O, locks, timers, runnable waiting for CPU, runnable waiting for page/swap-ins, etc.

Brendan Gregg's introduction to Off-CPU analysis can be found [here](http://www.brendangregg.com/offcpuanalysis.html).

Agent ZH's adaptation of flamegraphs to use Off-CPU analysis can be found [here](http://agentzh.org/misc/slides/off-cpu-flame-graphs.pdf).  Note that the tools referenced in the talk found [here](https://github.com/openresty/openresty-systemtap-toolkit) use [SystemTap](sys-perf.md) a lot.

### Notable Perf References

[How to analyze your system with perf and Python](https://opensource.com/article/18/7/fun-perf-and-python)

[Perf documentation](http://github.com/torvalds/linux/tree/master/tools/perf/Documentation).

[BCC Project](https://github.com/iovisor/bcc) For writing advanced profiling tools using eBPF

[User-space introspection with Linux perf](http://notes.secretsauce.net/notes/2015/01/28_user-space-introspection-with-linux-perf.html).  This covers 'perf probe'


## Performance Analysis Methodolgy

### References

[Top-Down Performance Analysis Methodology](https://easyperf.net/blog/2019/02/09/Top-Down-performance-analysis-methodology).  

[pmu-tools](https://github.com/andikleen/pmu-tools).  pmu tools is a collection of tools for profile collection and performance analysis on Intel CPUs on top of Linux perf. 

[Toplev](https://github.com/andikleen/pmu-tools/wiki/toplev-manual).  Identify the micro-architectural bottleneck of a workload.  Part of pmu-tools.

[Andy Klein's Blog](http://halobates.de/).   Also look for 'pmu tools part I' and 'pmu tools part II'