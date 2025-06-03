# erdos_quantum_computing

This project implements a 4x4 Sudoku solver using Grover search.

## Grover's search

Let $f : \mathbb{F}_2^n \to \mathbb{F}_2$ be a function that takes value $1$ at exactly one bitstring $x^*$. The aim is to recover this bitstring $x^*$ given the function.

Classically without further knowledge of the function $f$, the problem can be solved in $\mathcal{O}(2^n)$ time.  Grover's algorithm solves this problem in $\mathcal{O}(2^{n/2})$ time. It is a quadratic speed up.

Let $U$ be the "marker" quantum circuit that plays the role of the function $f$. That is,
$$
U|x\rangle =
\begin{cases}
-|x\rangle, & x = x^*\\
|x\rangle, & oth.
\end{cases}
$$

In the algorithm we treat $U$ as a blackbox, the runtime of $U$ is constant.

Let $|s\rangle$ be the "random state":
$$
|s\rangle = H^{\otimes n} |0\rangle_n.
$$

Define gate $S := H^{\otimes n} \circ X^{\otimes n} \circ C^{n-1}Z \circ X^{\otimes n} \circ H^{\otimes n}$. Then by construction, we have
$$
S|s\rangle = -|s\rangle, \quad
S|s^{\perp}\rangle = |s\rangle \text{ for any } \langle s^{\perp} | s \rangle = 0.
$$

Let $G := -S \circ U$ and let $|\psi_k\rangle = G^k|s\rangle$. Then Grover's theorem proves
$$
\mathbb{P}(\text{measure } |x^*\rangle \text{ in } |\psi_k\rangle)
= |\langle x^* | \psi_k \rangle
= \sin^2 \left(  (2k + 1) \sin^{-1}(2^{-n/2})  \right).
$$

If we let $K := \lfloor \frac{\pi}{4 \sin(2^{-n/2})}  - \frac{1}{2} \rceil \approx \frac{pi}{2} 2^{n/2}$, then
$$
\mathbb{P}(\text{measure } |x^*\rangle \text{ in } |\psi_k\rangle)
= 1 - \mathcal{O}(2^{-n/2}).
$$

## Sudoku

A $n^2 \times n^2$ Sudoku problem has input a $n^2 \times n^2$ board with some cells filled in with integers between $1$ and $n$. The goal is to fill the rest cells with integers from $1$ to $n$ such that:
* all numbers in each row are distinct
* all numbers in each column are distinct
* all numbers in each $n \times n$ square are distinct.

## Approach

We solve the case with $n = 2$. Each entry is represented by $2$ qubits. We need to build an oracle $U$ given an input Sudoku problem that takes $k$ Quantum Registers as input, where $k$ is the number of cells to fill in.

To be able to check the validity of a given input we need a circuit to check a list of Quantum Registers (2 qubit each) have unique values. This is implemented in the function `check_valid_circuit`. This circuit depends on a subroutine that checks if two Quantum Registers have the same value. This is implemented in the function `check_equal_circuit` and covered to the gate named `EQ`.

After we checked whether each row, each column and each $2 \times 2$ block have unique values, we flip the input if all of them are unique.

Finally we compute the number of iterations $K$ we need to perform the diffuser+oracle circuits.
