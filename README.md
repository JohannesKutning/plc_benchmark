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

$g(x) = x^{32} + x^{26} + x^{23} + x^{22} + x^{16} + x^{12} + x^{11} + x^{10} + x^{8} + x^{7} + x^{5} + x^{4} + x^{2} + x^{1} + 1$

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

| Parameter        | Value |
|------------------|-------|
| state-count      | 128   |
| iterations       | 16384 |
| transition-count | 3     |
| input_width      | 32    |
| output_width     | 32    |
| seed             | 42    |
| max_net_length   | 5     |
| min_net_prob     | 0.1   |

# Data Handling (data)

The data handling application is based on functions that are offered by the
[*OSCAT*](https://store.codesys.com/oscat-basic.html?___store=en) library, too.
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

As an additional benchmark program for floating point arithmetic the time
discret PID control offers a real application background.
The program implements the control loop shown in the following figure including
a PT2 element and the PID controller element.

![PID Loop](/doc/pid_loop.png?raw=true)

At first the program computes the input signal to the system with 8192 values
and the following signal form.
Afterwards the control loop including the PT2 element and the PID controller
element are computed for each input value.
The figure shows both the input **s** and the control loop ouput signal **y**.

![PID Signals](/doc/pid_signals.png?raw=true)

# Motion Controller (motion)

The motion control benchmark program contains a cascaded control loop of a DC
motor.
The motor is simulated with two serial connected PT1 elements and the program
uses three control loops.
The inner loop controls the motor's acceleration **a** via the current.
The second loop controls the motor's velocity **v** and the outer loop controls the
motors position **s**.
A schematic of the system is shown in the figure below.

![Motion Control Loop](/doc/motion_control_loop.png?raw=true)

The input signal is generated by the program prior to the control loop
execution and stored in a data block.
It is split into 20 parts and each part is defined either by a quadratics,
trigonometric or a straight line.
The signal generation uses the following definition:

```math
w_s(n) =
\begin{cases}
	 7,63 \cdot 10^{5} \cdot n^2                                  \, &\text{für }    0 \le n <  256 \\
	-7,63 \cdot 10^{5} \cdot \left( n -  512 \right)^2 + 10       \, &\text{für }  256 \le n <  512 \\
	   10                                                         \, &\text{für }  512 \le n <  768 \\
	-3,81 \cdot 10^{5} \cdot \left( n -  768 \right)^2 + 10       \, &\text{für }  768 \le n < 1280 \\
	 3,81 \cdot 10^{5} \cdot \left( n - 1792 \right)^2 - 10       \, &\text{für } 1280 \le n < 1792 \\
	  -10                                                         \, &\text{für } 1792 \le n < 2048 \\
	 3,81 \cdot 10^{5} \cdot \left( n - 2048 \right)^2 - 10       \, &\text{für } 2048 \le n < 2560 \\
	-3,81 \cdot 10^{5} \cdot \left( n - 3072 \right)^2 + 10       \, &\text{für } 2560 \le n < 3072 \\
		5 \cdot \cos\left( \sfrac{2\pi}{512}  \cdot x \right) + 5 \, &\text{für } 3072 \le n < 3584 \\
	   -5 \cdot \cos\left( \sfrac{2\pi}{1024} \cdot x \right) + 5 \, &\text{für } 3584 \le n < 4096 \\
		5 \cdot \cos\left( \sfrac{2\pi}{1024} \cdot x \right) - 5 \, &\text{für } 4096 \le n < 4608 \\
	   -5 \cdot \cos\left( \sfrac{2\pi}{512}  \cdot x \right) - 5 \, &\text{für } 4608 \le n < 5120 \\
	 3,81 \cdot 10^{5} \cdot \left( n - 5120 \right)^2 - 10       \, &\text{für } 5120 \le n < 5632 \\
	-3,81 \cdot 10^{5} \cdot \left( n - 6144 \right)^2 + 10       \, &\text{für } 5632 \le n < 6144 \\
	   10                                                         \, &\text{für } 6144 \le n < 6400 \\
	-3,81 \cdot 10^{5} \cdot \left( n - 6400 \right)^2 + 10       \, &\text{für } 6400 \le n < 6912 \\
	 3,81 \cdot 10^{5} \cdot \left( n - 7424 \right)^2 - 10       \, &\text{für } 6912 \le n < 7424 \\
	  -10                                                         \, &\text{für } 7424 \le n < 7680 \\
	 7,63 \cdot 10^{5} \cdot \left( n - 7680 \right)^2 - 10       \, &\text{für } 7680 \le n < 7936 \\
	-7,63 \cdot 10^{5} \cdot \left( n - 8192 \right)^2            \, &\text{für } 7936 \le n < 8192 \\
\end{cases}
```



# Fast Fourier Transform (fft)

To cover the signal processing performance of the PLCs a non-recursive FFT
computation based on the Cooley-Tukey-algorithm is used.
The recursion free algorithm is required due to the limited levels of recursion
a PLC offers (for some models it is less than 20).
The program covers floating point arithmetic as well as trigonometric and
cyclometric computations.
Due to the missing general exponential function $y = a^x$ in Siemens PLCs this
operation is implemented by $y = a^x = e^{ln\left(a\right) \cdot x}$.

The program generates the 4096 value long discret real input signal by adding a
64 Hz sine singnal, a 128 Hz sine signal and a random noise signal.
The sample time of the FFT is set to 1 kHz.

$X[n] = 0.7 \cdot \sin\left( 2 \pi \cdot \frac{64 Hz}{1 kHz} \cdot n\right) + \cdot \sin\left( 2 \pi \cdot \frac{128 Hz}{1 kHz} \cdot n\right) + rand(n)$

The noise source is a 14 bit wide LSFR with an output value transformation into
the interval $[-1,1]$.
This leads to the following input singal form:

![FFT Input Signal](/doc/fft_input_signal.png?raw=true)

The generated input signal is stored in a data block and the following FFT
computation function uses this data block for its intermediate results.
The result is stored to a data block and contains the following data:

![FFT Input Signal](/doc/fft_output.png?raw=true)

