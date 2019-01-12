---
Author: Jason Lowe-Power
Title: Testing DINO CPU
---

This file explains how to run the tests, how to create unit tests, how to create instruction directed tests, and how to run the simulator.

# Testing DINO CPU

Start sbt in the REPL (Read-eval-print loop), by running the following code in the base DINO directory.

```
singularity run library://jlowepower/default/dinocpu
```

To run the tests, you execute the `SingleCycleCPUTester` suite as follows.

```
dinocpu:sbt> testOnly dinocpu.SingleCycleCPUTester
```
If you want to run only a single application from this suite of tests, you can add a parameter to the `test` `sbt` task.
You can pass the option `-z` which will execute any tests that match the text given to the parameter.
You must use `--` between the parameters to the `sbt` task (e.g., the suite to run) and the parameters for testing.
For instance, to only run the subtract test, you would use the following:

```
dinocpu:sbt> testOnly dinocpu.SingleCycleCPUTester -- -z sub
```


# CPU Test Case
The `InstTests` object in`src/test/scala/cput-tests/InstTests.scala`contains lists of different instruction test cases for use with different CPU models.Each list is a different set of instruction types and corresponds to a RISC-V program in `src/test/resources/risc-v`.

Each test case looks like:
 - binary to run in src/test/resources/risc-v 
 - number of cycles to run for each CPU type
 - initial values for registers
 - final values to check for registers
 - initial values for memory
 - final values to check for memory
 - extra name information
 

  ```
 CPUTestCase(  "binary_name",
                Map("single-cycle" -> n_single, "five-cycle" -> n_fivecycle, "pipelined" -> n_pipelined),
                Map(rs1 -> data1, rs2 -> data2),
                Map(0 -> 0, rd -> (data1 ? data2) , ...),
                Map(), Map())
                
  ```
  
  
add2.riscv :

```
  .text
  .align 2       # Make sure we're aligned to 4 bytes
  .globl _start
_start:
    add a0, s4, t0 # (reg[10] = reg[20] + reg[5])

    nop
    nop
    nop
    nop
    nop
_last:
```

In the CPUTestCase below, we run the binary `add2`, which is compiled from `add2.riscv` (shown above) both of which are in `src/test/resources/risc-v`. The `add2.riscv` adds contents of `t0` (`reg[5]`) and `s4` (`reg[20]`) and stores it in `a0` (`reg[10]`).

  ```
 CPUTestCase(  "add2",
                Map("single-cycle" -> 1, "five-cycle" -> 5, "pipelined" -> 5),
                Map(5 -> 1234, 20 -> 5678),
                Map(0 -> 0, 10 -> 6912),
                Map(), Map())
                
  ```
  
  - `Map("single-cycle" -> 1, "five-cycle" -> 5, "pipelined" -> 5)` runs the single-cyle implementation for one cycle and the five-cycle, pipelined implementations for five cycles.
 - `Map(5 -> 1234, 20 -> 5678)` initializes `t0` (`reg[5]`) to 1234 and `s4` (`reg[20]`) to 5678.
 - `Map(0 -> 0, 10 -> 6912)` checks if the contents of `reg[0]` and `a0` are 0 and 6912 respectively.
