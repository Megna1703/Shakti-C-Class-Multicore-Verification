## Shakti C-Class Multicore Verification 
### Introduction 
The Shakti C-Class is a controller class processor featuring a 5-stage in-order design. The processor supports the standard RV64GCSUN ISA. When instantiated with four C-Class cores, a multicore configuration is achieved. For cache coherency, the MSI protocol is employed alongside a customized bus protocol. Ensuring the correct functioning of a processor is one of the cornerstones of modern microprocessor development. Functional verification is used for this purpose. On this page, I present some tests for verification of Shakti C-Class
Multicore using RISCV-DV tests. 

### RISCV-DV Tests
RISCV-DV is an open-source, Python-based instruction generator designed for the verification of RISC-V processors. 
* It supports multiple hardware threads and generates tests where each test produces a program with distinct assembly instructions. 
* RISCV-DV tests also offer co-simulation support with Spike, an Instruction Set Simulator (ISS). 
* For verification, I conducted simulations using Spike, which emulates an ideal RISC-V core, and compared the results with simulations from the Shakti core. Spike simulation of Shakti C-Class with the generated assembly
programs.

### Generating tests using Pyflow simulator 
There are several tests in the *baselist.yaml*. Each test case employed for verification is likely to stimulate distinct functionalities within the assembler. Here are some tests that I used for verifying. 

#### RISCV_ARITHMETIC_BASIC_TEST :
It is specifically designed to generate assembly instructions for basic arithmetic operations on an RISC-V processor. The test generates assembly instructions for fundamental arithmetic operations like addition, subtraction, multiplication, and division. It doesn't have support for floating-point instructions.

The following command is used to generate arithmetic basic tests using *pyflow*.



```
python3 run.py --test=riscv_arithmetic_basic_test --simulator=pyflow --steps gen

```
Based on the num_of_tests, it will generate many numbers of assembly tests with the given instruction count named *riscv_arithmetic_basic_test_0.S* and *riscv_arithmetic_basic_test_1.S* and so on.

#### RISCV_JUMP_STRESS_TEST :
It is specifically designed to generate assembly instructions that stress-test the jump functionality of
a RISC-V processor. The test generates a large number (stressful amount) of assembly instructions for various jump types like unconditional and conditional jumps.
The following command is used to generate jump stress tests using pyflow. 
```
python3 run.py --test=riscv_jump_stress _test --simulator=pyflow --steps gen
```
Based on the num_of_tests, it will generate that many number of assembly tests with the given instruction
count named as *riscv_jump_stress_test_0.S* and *riscv_jump_stress_test_1.S* and so on.

### Simulating on Spike 
Spike, the RISC-V ISA Simulator, implements a functional model of one or more RISC-V harts. Spike simulation results are used to compare with Shakti’s results for verification. 
The previously generated assembly files are converted into a binary file to simulate on Spike using the following command:
```
riscv64-unknown-elf-gcc -nostartfiles -T link.ld riscv_arithmetic_basic_test_0.S

```
The previous command will create a binary file *a.out*. This file can be used to run on spike using the
following command. This will create the log file named *spike.log*. -p4 is a flag to simulate 4 cores.

```
spike -l --log=spike.log -p4 a.out
```
After running on Spike, we can generate a dump file which can be used for comparison with Shakti’s
dump file. The command used is :

```
spike -l --log=spike.dump -p4 a.out
```
### Execution on Shakti Multicore Setup 
The assembly tests cannot be directly run on Shakti CPU. The test has to be converted into hex files using
the elf2hex command :

```
elf2hex 8 33554432 a.out 2147483648 > code.mem
```
Then the following command will generate a dump of Shakti’s execution –
```
./out +rtldump
```
The multiharts are simulated on Shakti multicore setup and *rtl.0.dump, rtl.1.dump, rtl.2.dump, rtl.3.dump* files are generated for the respective cores.

### Comparision of dump files 
The *spike.log* created will create one single file with memory traces of all 4 cores. A python script
*gendump.py* is used to separately obtain memory dump for each core.
Now the memory dump file for each core is compared using a Python script *diff.py* (it uses the *diff* command) to compare and store the difference in another file. 

### Test Coverage 
Coverage is crucial in verification because it provides a metric for how thoroughly you’ve tested a design.
By analyzing coverage metrics, you can create tests specifically targeting those functionalities.

#### Coverage for riscv_arithmetic_basic_test: 
The *spike.log* file is used to create the coverage report using the following command :
```
python3 cov.py –dir arithmetic_basic_test/asm_test/ --simulator pyflow –enable_visualization
```
This will create a file *Coverage.txt*. The number of groups in the table indicates the number of different instructions used in the tests. 
* In the *riscv_arithmetic_basic_test* all basic arithmetic instructions are used except for floating point instructions and jump instructions.
* The coverage scores for floating and jump instructions are 0.
* 
#### Coverage for riscv_jump_stress_test:
The *spike.log* file is used to create the coverage report using the following command :
```
python3 cov.py –dir jump_test/asm_test/ --simulator pyflow –enable_visualization
```
In *riscv_jump_stress_test* all jump instructions are used. The jump instruction scores are non-zero.


