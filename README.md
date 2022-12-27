# plc_benchmark

A set of PLC programs written in STL.
This set contains programs with different instruction set focus.
Starting with boolean logic focused programs like the **ripple-carry-adder**
and the **crc** computation.
The integer instructions are covered by the finite state machine implementation
**fsm** and the data handling program **data**.
For floating point and signal processing the programs **pid** controller,
**motion** controller and fast Fourier transform **fft** are available.

All programs are coded in STL for Siemens PLC 1500 and 300.

# Ripple-Carry-Adder (add)

The ripple-carry-adder program is used the assess the PLC's boolean operations
computation speed.
The adder takes two 15360 bit wide unsigned operands and computes their sum.
The adder is implemented as an unroled loop for a greater instruction memory
usage.
Due to the limited code size of a function block of 64 KiB, the maximum adder
size within one function block is 512 bit.
This leads to a program with three adder function blocks that are each called
30 times.
The ripple-carry-adder is based on full adder blocks, that consist of three
XOR, two AND and one OR operations.

Due to its simple nature and its limited practial relevance the adder is to be
categorized as a toy benchmark program.
It is part of the collection to cover the frequently used boolean logic
instructions of PLC programs.

# Cyclic Redundancy Check (crc)

The CRC program implements a 32 bit linear shift feedback register according to
IEEE 802.3 with the following generator polynom:


    g(x) = x^31 + x^26 + x^23 + x^22 + x^16 + x^12 + x^11 + x^10 + x^8 + x^7 + x^5 + x^4 + x^2 + x^1 + 1


The input data is generated prior to the computation and stored to a data
block.
From there it is read out bit by bit.
The CRC algorithm is part of the OSCAT library and part of real PLC programs.

# Finite State Machine (fsm)

A synthetic finite state machine program is used to benchmark the boolean
logic, integer logic and branch instructions.
The program is split into an input network, a state transition network and an
output network.
A [state machine generator](https://github.com/JohannesKutning/fsm_generator)
is used to generate the synthetic state machine and derive the STL source code.
The input and output networks are based on boolean logic instruction and the
state transition network uses a swtich-case style with integer comparision
operations.
The generation uses the follwoing arguments:

    state-count      | 128
    iterations       | 16384
    transition-count | 3
    input_width      | 32
    output_width     | 32
    seed             | 42
    max_net_length   | 5
    min_net_prob     | 0.1

# Data Handling (data)

# PDI Controller (pid)

# Motion Controller (motion)

# Fast Fourier Transform (fft)

