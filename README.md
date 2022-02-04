# Simon-Algorithm-in-Cirq
An implementation of Simon's Algorithm using Google Quantum AI - Cirq.

This is a report for [LTAT.00.008 Theoretical Informatics Project] course in the University of Tartu.

Link to the implementation in Google Colab: [Cirq Simon's Algorithm](https://qiskit.org/textbook/ch-algorithms/simon.html)

Reference: [Qiskit Simon's Algorithm](https://colab.research.google.com/drive/1JDirSZv9dL2awlqEfiZWK0S38WaAmVT4?usp=sharing)

## 1. Introduction

Simon's algorithm, first introduced in Reference [1], was the first quantum algorithm to show an exponential speed-up versus the best classical algorithm in solving a specific problem. This inspired the quantum algorithms based on the quantum Fourier transform, which is used in the most famous quantum algorithm: Shor's factoring algorithm

### 1a. Simon's Problem

We are given an unknown blackbox function <img align="center" src="https://render.githubusercontent.com/render/math?math=f">, which is guaranteed to be either one-to-one <img align="center" src="https://render.githubusercontent.com/render/math?math=(1:1)"> or two-to-one <img align="center" src="https://render.githubusercontent.com/render/math?math=(2:1)">, where one-to-one and two-to-one functions have the following properties:

*   **one-to-one**: maps exactly one unique output for every input. An example with a function that takes 4 inputs is:
<p align="center">
<img align="center" src="https://render.githubusercontent.com/render/math?math=f(1) \rightarrow 1, f(2) \rightarrow 2, f(3) \rightarrow 3, f(4) \rightarrow 4">
</p>

*   **two-to-one**: maps exactly two inputs to every unique output. An example with a function that takes 4 inputs is:

<p align="center">
<img align="center" src="https://render.githubusercontent.com/render/math?math=f(1) \rightarrow 1, f(2) \rightarrow 2, f(3) \rightarrow 1, f(4) \rightarrow 2">
</p>

This two-to-one mapping is according to a hidden bitstring, <img align="center" src="https://render.githubusercontent.com/render/math?math=b">, where:

<p align="center">
given <img align="center" src="https://render.githubusercontent.com/render/math?math=x_1,x_2: \quad f(x_1) = f(x_2)">
<br/>
it is guaranteed : <img align="center" src="https://render.githubusercontent.com/render/math?math=\quad x_1 \oplus x_2 = b">
</p>

Given this blackbox <img align="center" src="https://render.githubusercontent.com/render/math?math=f">, how quickly can we determine if <img align="center" src="https://render.githubusercontent.com/render/math?math=f"> is one-to-one or two-to-one? Then, if <img align="center" src="https://render.githubusercontent.com/render/math?math=f"> turns out to be two-to-one, how quickly can we determine <img align="center" src="https://render.githubusercontent.com/render/math?math=b">? As it turns out, both cases boil down to the same problem of finding <img align="center" src="https://render.githubusercontent.com/render/math?math=b">, where a bitstring of <img align="center" src="https://render.githubusercontent.com/render/math?math=b={000...}"> represents the one-to-one <img align="center" src="https://render.githubusercontent.com/render/math?math=f">.

### 1b. Simon's Algorithm <a id='algorithm'> </a>

#### Classical Solution

Classically, if we want to know what <img src="https://render.githubusercontent.com/render/math?math=b"> is with 100% certainty for a given <img align="center" src="https://render.githubusercontent.com/render/math?math=f">, we have to check up to <img align="center" src="https://render.githubusercontent.com/render/math?math=2^{n-1} + 1">  inputs, where n is the number of bits in the input. This means checking just over half of all the possible inputs until we find two cases of the same output. Much like the Deutsch-Jozsa problem, if we get lucky, we could solve the problem with our first two tries. But if we happen to get an <img align="center" src="https://render.githubusercontent.com/render/math?math=f"> that is one-to-one, or get _really_ unlucky with an <img align="center" src="https://render.githubusercontent.com/render/math?math=f"> that’s two-to-one, then we’re stuck with the full <img src="https://render.githubusercontent.com/render/math?math=2^{n−1}+1">.
There are known algorithms that have a lower bound of <img align="center" src="https://render.githubusercontent.com/render/math?math=\Omega(2^{n/2})"> (see Reference 2 below), but generally speaking the complexity grows exponentially with <img src="https://render.githubusercontent.com/render/math?math=n">.

#### Quantum Solution

The quantum circuit that implements Simon's algorithm is shown below.

![image](https://user-images.githubusercontent.com/62504305/152556650-7a075ce1-b3b1-4d29-ad00-edc10e89baf6.png)

Where the query function, <img src="https://render.githubusercontent.com/render/math?math=\text{Q}_f"> acts on two quantum registers as:

<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=\lvert x \rangle \lvert a \rangle \rightarrow \lvert x \rangle \lvert a \oplus f(x) \rangle">
</p>
  
In the specific case that the second register is in the state $|0\rangle = |00\dots0\rangle$ we have:

<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=\lvert x \rangle \lvert 0 \rangle \rightarrow \lvert x \rangle \lvert f(x) \rangle">
</p>

The algorithm involves the following steps.
<ol>
   <li> Two <img src="https://render.githubusercontent.com/render/math?math=n">-qubit input registers are initialized to the zero state: 
    
<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=\lvert \psi_1 \rangle = \lvert 0 \rangle^{\otimes n} \lvert 0 \rangle^{\otimes n}">
</p>

 </li>
   <li> Apply a Hadamard transform to the first register:
    
<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=\lvert \psi_2 \rangle = \frac{1}{\sqrt{2^n}} \sum_{x \in \{0,1\}^{n} } \lvert x \rangle\lvert 0 \rangle^{\otimes n}">
</p>
     
   </li>
    
   <li> Apply the query function <img src="https://render.githubusercontent.com/render/math?math=\text{Q}_f">: 
    
<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=\lvert \psi_3 \rangle = \frac{1}{\sqrt{2^n}} \sum_{x \in \{0,1\}^{n} } \lvert x \rangle \lvert f(x) \rangle">
</p>
 
   </li>
    
   <li> Measure the second register. A certain value of <img src="https://render.githubusercontent.com/render/math?math=f(x)"> will be observed. Because of the setting of the problem, the observed value <img src="https://render.githubusercontent.com/render/math?math=f(x)"> could correspond to two possible inputs: <img src="https://render.githubusercontent.com/render/math?math=x"> and <img src="https://render.githubusercontent.com/render/math?math=y = x \oplus b">. Therefore the first register becomes:
    
<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=\lvert \psi_4 \rangle = \frac{1}{\sqrt{2}}  \left( \lvert x \rangle + \lvert y \rangle \right)">
</p>

   where we omitted the second register since it has been measured. 
   </li>
    
   <li> Apply Hadamard on the first register:
    
<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=\lvert \psi_5 \rangle = \frac{1}{\sqrt{2^{n+1}}} \sum_{z \in \{0,1\}^{n} } \left[  (-1)^{x \cdot z} + (-1)^{y \cdot z} \right]  \lvert z \rangle">
</p>


   </li>
    
   <li> Measuring the first register will give an output only if:
    
<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=(-1)^{x \cdot z} = (-1)^{y \cdot z}">
</p>


   which means:
<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=x \cdot z = y \cdot z">
<br/>
<img src="https://render.githubusercontent.com/render/math?math=x \cdot z = \left( x \oplus b \right) \cdot z">
<br/>
<img src="https://render.githubusercontent.com/render/math?math=x \cdot z = x \cdot z \oplus b \cdot z">
<br/>
<img src="https://render.githubusercontent.com/render/math?math=b \cdot z = 0 \text{ (mod 2)}">
</p>
      
   A string <img src="https://render.githubusercontent.com/render/math?math=z"> will be measured, whose inner product with <img src="https://render.githubusercontent.com/render/math?math=b = 0">. Thus, repeating the algorithm <img src="https://render.githubusercontent.com/render/math?math=\approx n"> times, we will be able to obtain <img src="https://render.githubusercontent.com/render/math?math=n"> different values of <img src="https://render.githubusercontent.com/render/math?math=z"> and the following system of equation can be written:
       
    
<p align="center">
<img src="https://render.githubusercontent.com/render/math?math=\begin{cases} b \cdot z_1 = 0 \\ b \cdot z_2 = 0 \\ \quad \vdots \\ b \cdot z_n = 0 \end{cases}">
<br/>
</p>
       
   From which <img src="https://render.githubusercontent.com/render/math?math=b"> can be determined, for example by Gaussian elimination.
    </li>
</ol>

So, in this particular problem the quantum algorithm performs exponentially fewer steps than the classical one. Once again, it might be difficult to envision an application of this algorithm (although it inspired the most famous algorithm created by Shor) but it represents the first proof that there can be an exponential speed-up in solving a specific problem by using a quantum computer rather than a classical one.

## 2. References <a id='references'></a>

1. Daniel R. Simon (1997) "On the Power of Quantum Computation" SIAM Journal on Computing, 26(5), 1474–1483, [doi:10.1137/S0097539796298637](https://doi.org/10.1137/S0097539796298637)
2. Guangya Cai and Daowen Qiu. Optimal separation in exact query complexities for Simon's problem. Journal of Computer and System Sciences 97: 83-93, 2018, [https://doi.org/10.1016/j.jcss.2018.05.001](https://doi.org/10.1016/j.jcss.2018.05.001)

3. [Intro to Quantum Computation: Lecture 8 - Simon's algorithm and applications to cryptography](https://www.youtube.com/watch?v=TQ5U5IO-Z7g)
