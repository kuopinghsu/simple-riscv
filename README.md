# SRV32 - Simple 3-stage pipeline RISC-V processor

[![Codacy Badge](https://app.codacy.com/project/badge/Grade/d14f73089d014a87beb91960e9340617)](https://www.codacy.com/manual/kuopinghsu/srv32/dashboard?utm_source=github.com&amp;utm_medium=referral&amp;utm_content=kuopinghsu/srv32&amp;utm_campaign=Badge_Grade)

This is a simple RISC-V 3-stage pipeline processor and supports FreeRTOS.
I wrote this code to understand the RV32IM instruction set, just for fun.
The performance is 1.821 DMIPS/MHz and 2.681 Coremark/MHz.
This is not a RISC-V core available for production.

## Features

1.  Three-stage pipeline processor
2.  RV32IM instruction sets
3.  Pass RV32IM <A Href="https://github.com/riscv/riscv-compliance">compliance test</A>
4.  Trap exception
5.  Interrupt handler
6.  <A Href="https://github.com/kuopinghsu/FreeRTOS-RISCV">FreeRTOS</A> support
7.  ISS simulator

## Building toolchains

Install RISCV toolchains.

    # Ubuntu packages needed:
    sudo apt-get install autoconf automake autotools-dev curl libmpc-dev \
        libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo \
        gperf libtool patchutils bc zlib1g-dev git libexpat1-dev
    
    git clone --recursive https://github.com/riscv/riscv-gnu-toolchain
    cd riscv-gnu-toolchain
    
    mkdir build; cd build
    ../configure --prefix=/opt/riscv --enable-multilib
    make -j$(nproc)

## Files list

| Folder    | Description                                     |
| --------- | ----------------------------------------------- |
| coverage  | Code coverage report                            |
| doc       | Instruction set document                        |
| rtl       | RTL files                                       |
| sim       | Icarus Verilog/Verilator simulation environment |
| sw        | Benchmark, diags ... etc.                       |
| syn       | Synthesis environment for Yosys                 |
| tests     | Compliance tests                                |
| testbench | testbench, memory model                         |
| tool      | ISS (instruction set simulator)                 |

## RTL Simulation

Support Verilator (default) and Icarus Verilog.

To run Verilator,

    # Ubuntu package needed to run the RTL simulation
    sudo apt install verilator
    
    # Modify the Makefile, and set verilator ?= 1
    vim sim/Makefile
    or
    make verilator=1

To run Icarus Verilog,

    # Ubuntu package needed to run the RTL simulation
    sudo apt install iverilog
    
    # Modify the Makefile, and set verilator ?= 0
    vim sim/Makefile
    or
    make verilator=0

Only running make without parameters will get help.

    $ make
    make all         build all diags and run the RTL sim
    make all-sw      build all diags and run the ISS sim
    make tests-all   run all diags and compliance test
    make coverage    generate code coverage report
    make build       build all diags and the RTL
    make dhrystone   build Dhrystone diag and run the RTL sim
    make coremark    build Coremark diag and run the RTL sim
    make clean       clean
    make distclean   clean all

Supports following parameter when running the simulation.

    +no-meminit do not memory initialize zero
    +dump       dump VCD (Icarus Verilog) / FST (Verilator) file
    +trace      generate tracelog

For example, following command will generate the VCD dump.

    cd sim && ./sim +dump

Use +trace to generate a trace log, which can be compared with the log file of the ISS simulator to ensure that the RTL simulation is correct.

    cd sim && ./sim +trace

## ISS (Instruction Set Simulator)

The rvsim is an instruction set simulator (ISS) that can generate trace logs for comparison with RTL simulation results. It can also set parameters of branch penalty to run benchmarks to see the effect of branch penalty. The branch instructions of hardware is two instructions delay for branch penalties.

    Usage: rvsim [-h] [-b n] [-p] [-l logfile] file

           --help, -n              help
           --branch n, -b n        branch penalty (default 2)
           --predict, -p           static branch prediction
           --log file, -l file     generate log file

           file                    the elf executable file

The ISS simulator and hardware supports RV32IM instruction sets.

## Benchmarks

This is the RV32IM simulation result on GCC10.

### Dhrystone

The call tree generated by <A Href="https://github.com/kuopinghsu/callgraph-gen">graphgen</A>.

<img src="images/dhrystone.svg" alt="Dhrystone" width=640>

> Generated by command 'graphgen -t -r main -i printf,puts,putchar dhrystone.dis'

<img src="images/dhrystone.png" alt="Dhrystone" width=640>

### Coremark

The call tree generated by <A Href="https://github.com/kuopinghsu/callgraph-gen">graphgen</A>.

<img src="images/coremark.svg" alt="Coremark" width=640>

> Generated by command 'graphgen -t -r main -i printf,puts,putchar coremark.dis'

<img src="images/coremark.png" alt="Coremark" width=640>

> Note: Coremark requires a total time of more than 10 seconds, but this will result in a longer simulation time. This Coremark value provides a reference when the iteration is 4.

### Benchmark with different configurations

| Name      | GCC8 RV32I                      | GCC8 RV32IM                     | GCC10 RV32I                     | GCC10 RV32IM                    | GCC10 RV32IM (*1)               | GCC10 RV32IM (*2)              |
|-----------| ------------------------------- | ------------------------------- | ------------------------------- | ------------------------------- | ------------------------------- | ------------------------------ |
| Dhrystone | 1.610 DMIPS/MHz<br>1.188 CPI    | 1.797 DMIPS/MHz<br>1.179 CPI    | 1.628 DMIPS/MHz<br>1.191 CPI    | 1.821 DMIPS/MHz<br>1.181 CPI    | 1.844 DMIPS/MHz<br>1.166 CPI    | 2.151 DMIPS/MHz<br>1.000 CPI   |
| Coremark  | 0.970 Coremark/MHz<br>1.400 CPI | 2.268 Coremark/MHz<br>1.257 CPI | 0.970 Coremark/MHz<br>1.399 CPI | 2.681 Coremark/MHz<br>1.253 CPI | 3.059 Coremark/MHz<br>1.096 CPI | 3.348 Coremark/MHz<br>1.000CPI |

> Note 1: this is a measurement made by ISS simulator for GCC10 with static branch prediction.

> Note 2: this is a measurement made by ISS simulator for GCC10 with no branch penalty. This is the best case for single issue and in order execution on RV32IM. The 1.0 CPI is the upper limit of optimal performance.

> Note 3: After using LTO (Link Time Optimization), Dhrystone scored an unreasonable score of 3.73 DMIPS/MHz on GCC10 RV32IM. To make the benchmark reasonable, I did not use the -flto option. If the -flto option is used, the score will be higher.

## Cycles per Instruction Performance

Two instructions branch penalty if branch taken, CPI is 1 for other instructions. The average Cycles per Instruction (CPI) is approximately 1.2 on Dhrystone diag.

This core is three-stage pipeline processors, which is Fetch & Decode (F/D), execution (E) and write back (WB).

*   Register Forwarding

The problem with data hazards, introduced by this sequence of instructions can be solved with a simple hardware technique called forwarding. When the execution result accesses the same register, the execution result is directly forwarded to the next instruction.

<img src="images/forwarding.svg" alt="Register Forwarding" width=320>

*   Branch Penalty

When the branch is taken during the execute phase, it needs to flush the instructions that have been fetched into the pipeline, which causes a delay of two instructions, so the extra cost of the branch is two.

<img src="images/branch.svg" alt="Branch Penalty" width=320>

## Memory Interface

One instruction memory and one data memory. The instruction memory is read-only for one read port, while data memory is two port, one for reading and one for writing.

## Synthesis

Provide the Yosys synthesis script on the syn folder.

## Compliance tests

Compliance test for ISS simulator and RTL. This is the compliance test form RISC-V Foundation Compliance Task Group.
The github repository is at <https://github.com/riscv/riscv-compliance>. Running the following command will clone the repository into tests folder and do the compliance test.

    make tests              # run the compliance test for RTL
    make tests-sw           # run the compliance test for ISS simulator

Default is to run compliance test v1, run the following command to run compliance test v2.

    make test_v=2 tests     # run the compliance test for RTL
    make test_v=2 tests-sw  # run the compliance test for ISS simulator

## FreeRTOS support

Reference code on <https://github.com/kuopinghsu/FreeRTOS-RISCV>.

    # build FreeRTOS and demo
    git clone --recursive https://github.com/kuopinghsu/FreeRTOS-RISCV
    cd FreeRTOS-RISCV && make

This is an example to run the "queue" demo.

    # make directory
    mkdir ${path_of_srv32}/sw/queue

    # copy queue.elf
    cp Demo/examples/queue.elf ${path_of_srv32}/sw/queue/.

    # update the example in sw folder
    cd ${path_of_srv32}/sw && make update

    # run the example for RTL and ISS simulator
    cd ${path_of_srv32} && make queue

## Coverage Report

    # Ubuntu package needed to generate coverage report
    sudo apt install lcov

Following command will generate the code coverage report in coverage/html
directory.

    % make coverage

This is the coverage report of RTL by lcov, which get 100% code coverage.

<img src="images/coverage.png" alt="coverage">

This is the coverage report of ISS.

<img src="images/coverage_iss.png" alt="coverage-ISS">

## Known issues

*   Memory can not respond non-valid, that is, the memory should always accept the command from CPU.

## TO-DO

*   merge the memory interface into one memory for one port
*   static branch predictor
*   support RV32C compress extension
*   serial multiplier and divider
*   Nuttx & Zephyr porting

## License

MIT license
