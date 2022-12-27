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
The adder is implemented as an unrolled loop for a greater instruction memory
usage.
Due to the limited code size of a function block of 64 KiB, the maximum adder
size within one function block is 512 bit.
This leads to a program with three adder function blocks that are each called
30 times.
The ripple-carry-adder is based on full adder blocks, that consist of three
XOR, two AND and one OR operations.

Due to its simple nature and its limited practical relevance the adder is to be
categorized as a toy benchmark program.
It is part of the collection to cover the frequently used boolean logic
instructions of PLC programs.

# Cyclic Redundancy Check (crc)

The CRC program implements a 32 bit linear shift feedback register according to
IEEE 802.3 with the following generator polynomial:


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
state transition network uses a switch-case style with integer comparison
operations.
The generation uses the following arguments:

    state-count      | 128
    iterations       | 16384
    transition-count | 3
    input_width      | 32
    output_width     | 32
    seed             | 42
    max_net_length   | 5
    min_net_prob     | 0.1

# Data Handling (data)

The data handling application is based on functions that are offered by the
*OSCAT* library, too.
All data is processed and stored as a 32 bit floating point type.
The benchmark program is split into the following steps.

 1. Initialize the input data array with increasing float values
 2. Copy the input data array into a temporal data array
 3. Permute the temporal data array with the Fischer-Yates algorithm.
 4. Find the minimum, maximum and the arithmetic mean of the permuted data and
    compute the sum, the product, the standard diviation and the variance.
 5. Copy the permuted data into a second temporal array for sorting.
 6. Use the quick-sort algorithm to sort the permuted data of the second
    temporal array.
 7. Compare the sorted data array with the input data array.

All these steps are implemented in a single function block with no sub function
calls.
For the data storage three data blocks are used.
Each of the data blocks stores 4096 float values.
In addition to that the recursion free quick-sort algorithm requires two
additional data blocks for intermediate results.

# PDI Controller (pid)

# Motion Controller (motion)

# Fast Fourier Transform (fft)

To cover the signal processing performance of the PLCs a non-recursive FFT
computation based on the Cooley-Tukey-algorithm is used.
The recursion free algorithm is required due to the limited levels of recursion
a PLC offers (for some models it is less than 20).
The program covers floating point arithmetic as well as trigonometric and
cyclometric computations.
Due to the missing general exponential function $y = a^x$ in Siemens PLCs this
operation is implemented by $y = a^x = e^{ln\left(a\right) \cdot x}$.

