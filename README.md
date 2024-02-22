# Ising ZZ model 1D with transverse field
This project implements an Ising model in a 1-dimensional spin 1/2 chain using TenPy Python's library.

## Use
The main code is "Ising_Model.ipynb". Just run this code to start the simulation of the 1D Ising Model. The model and algorithm parameters can be adjusted as necessary to study different systems and regimes.

## Model description
The one-dimensional Ising model consists of a spin 1/2 chain interacting under the influence of an external magnetic field. Each spin can be in only two possible states $s_z=\pm 1/2$. The energy of the system is determined by the Hamiltonian, i.e. by the interactions between adjacent spins and the external field. The aim is to optimise the wave function to find the fundamental state of the system using the DMRG algorithm.

As stated in the problem requirements, interactions between spins must be mediated by couplings to next-nearest neighbours (nNN), but I have also implemented the possibility of having interactions to nearest neighbours (NN) by simply changing the corresponding parameter. The most general hamiltonian implemented in the code is given by:

$$ H= -J_1\sum_{i=1}^{L-1} \sigma_i^z \sigma_{i+1}^z-J_2\sum_{i=1}^{L-2} \sigma_i^z \sigma_{i+2}^z -\sum_{\beta=\{x,y,z\}} \sum_{i=1}^L B^\beta \sigma^\beta_i$$

, where L is the spin chain lenght. $J_1$ , $J_2$ are the NN and nNN coupling constants, respetively. The last local terms corresponds to the transverse magnetic $B$ field whose direction, in principle, can be set in any direction in the sphere . Since the system is rotationally symmetric, we could have chosen other axes for the spin-spin interactions. By default, $J_1=0$ and $\vec{B}=(B,0,0)$ are set in the code to have only nNN interactions with a transverse field in the X-axis.

## Code description
The code in "Ising_Model.ipynb" has te following structure:

  - Importing the necessary libraries
  - IsingZZX model and parameters
  - DMRG
  - Energy, magnetization and magnetic susceptibility plots
  - Appendix

### IsingZZX and parameters
TenPy is a versatile python library MPS-based focused on quantum systems. It allows you to create MPS's and MPO's which describe a specific system with interactions. One can start by defining which kind of sites conform our system (spins in our case), how they distribute topologically defining a lattice and adding the coupling and local terms which describes the hamiltonian.

In this way, also by mapping the lattice in some MPS order, TenPy automates the building of the hamiltonian in the MPO form and allows us to easily build a wavefunction with a MPS structure. In addition, I created a method "self.initial_state()" for easily generate an MPS (see the method for more details).

A description of the parameters used in the model is given in the correspondient section, including lattice and MPS boundary conditions.

### DMRG
In this block I search the ground state for each $|\vec{B}|$ value. Also, I perform loops for different hopping values in order to see how does the critical point change, but you change it. These and other parameters are set considering a lattice size order of roughly 100 spins.

In each loop, the Ising Model restarts changing the hopping and field values. Next, we initialize $|\psi_0\rangle$ which is the wavefunction that DMRG will later try to minimize. This is crucial because in these types of systems, we may start from a point where it is hard for the algorithm to minimize, due to constantly reaching local or degenerate minima. That's why I have finally chosen Neel's state or a disordered state (mode=[1,1]) to try to reach the global minimum in the first calculation. To perform the optimization in a a more efficient way, we take as input the optimized wavefunction from the last step, since the change in the parameters is small.  

On the other hand, in this task I have learned that DMRG has more difficulties to converge for $L$ small (0-70) than larger ones. I guess the symmetry breaking that originates the 2 potential wells are less prominent at these sizes and DMRG just doesn't know where the minimum is, or maybe (more probable) I'm doing something wrong. It turns out that in the opposite case, the DMRG's behaviour is really different. When symmetry breaking takes place,i.e., the state with +Z magnetization has the same energy than the state with -Z, for large enough $L$ and little enough bond dimension $\chi$, DMRG tends to choose one of both states since preserve superposition leads to a harder truncation.

Finally, I chose the two-site DMRG to help reaching the golbal minimum, since the single-site version could fall in local minima, but would be worth it to explore if the results are the same because single-site DMRG is faster. 

The rest of the programme  fully tunable plots of the energy and magnetic susceptibility.


# Rest of the tasks

## Exercise 2
**Explain How you would construct a Matrix Product Operator for the Hamiltonian, and how
you use it in your DMRG code.**

The problem of constructing a Hamiltonian describing a many-body system in MPO form reduces to expressing the sums of operators acting on different sites as contractions of tensors in which each tensor corresponds to and operates on a site, and is described by operators acting on the same site.

In the case of having only nearest-neighbour couplings we could express each of the MPO matrices in a lower dimension. As the range of interactions increases, the bound dimension of tensors increases in order to represent the Hamiltonian.

In our case, the maximum range of interactions is 2 (nNN) and the Hamiltonian will be described by:

![HnNN_MPO](https://github.com/IvanHuarte/Ising_DMRG_Ivan_Huarte_Franceschini/assets/136732928/4add7859-13c7-4845-ae42-fae774bb5f3a)

And the MPO for the general hamiltonian of the model(NN + nNN):
![H_NN_nNN_MPO](https://github.com/IvanHuarte/Ising_DMRG_Ivan_Huarte_Franceschini/assets/136732928/f24e7267-a8fd-4b67-8db5-aca0f10d40b5)

As can be seen in the H_MPO section of the code appendix, one can visualise the arrays within an MPO, and even the operators that make it up. In fact, in the second code block, if we change the index $i$ of *IM.H_MPO._W[5][i]* we can visualise the columns of each of the tensors of the MPO. In fact, it can be seen that the creator of TenPy has chosen the opposite convention when defining the tensors.

Similarly, we can see other information that is stored in the Array class such as the dimension of each index, the order of the bound and physical indices of the tensor, and information related to the charge conservation of the tensor.

With all this in mind, one could build each of the tensors with Tenpy's npc.Array() and pass the W arrays to Tenpy's MPO() class with the corresponding sites of the model, to ensure that the charge is compatible.

Having the Hamiltonian in MPO form already belonging to a model, we can run the DMRG as in the simulation.

The DMRG takes the hamiltonian and the initial wavefunction and sweeps along the chain, varying the wavefunction at each step. Depending on the version of the DMRG the steps may vary but they are based on taking each site (or two) and optimising it locally. This is repeated along the chain (back and forth) which represents a sweep.

In each local optimisation, the MPS site is not fully optimised, but only a little, to prevent the algorithm from convoluting the MPS.

At the end of the optimization, if everything is fine $|\psi\rangle=|\psi_{gr}\rangle$ and the energy is computed $E_{gr}=\langle\psi_{gr}|H|\psi_{gr}\rangle$

## Exercise 3
**Discuss which parts of the code could potentially be accelerated using quantum
computers**

In the context of quantum computing, the Hamiltonian in the form of MPO can be implemented in a quantum circuit as long as the MPO tensors are unitary.

The unitary matrices that represent the operations in the circuit encode all the information about the interactions between the qubits. Although the DMRG algorithm itself is a classical method, quantum computing can provide advantages for running it.

The Hamiltonian in a quantum system can be directly implemented using a quantum circuit. The representation of the Hamiltonian acting on a wave function in the form of a quantum circuit would take the following form:

![QCircuit](https://github.com/IvanHuarte/Ising_DMRG_Ivan_Huarte_Franceschini/assets/136732928/9ee58ea4-cab3-443d-a371-6156993d5123)

On the other hand, when the computational complexity increases with the size of the problem. In NP-hard optimization problems there are classic techniques such as tensor networks that can be mapped to a quantum circuit and take advantage of the capacity of qubits to solve the problem. The advantage of this method lies in the parallel calculation that is carried out operating with qubits, and that all the information related to the entanglement between the MPS sites is encoded in the intrinsic entanglement between the qubits of the network. This represents a huge advance, especially in highly entangled quantum systems.

Finally, there are different variational quantum algorithms (VQA) that perform the same function as DMRG, in which the wave function is optimized iteratively until reaching the fundamental state.


### Exercises 4 & 5

![GS_Energy](https://github.com/IvanHuarte/Ising_DMRG_Ivan_Huarte_Franceschini/assets/136732928/dfa0a789-6085-47da-8655-db317f36b12b)

![GS_X](https://github.com/IvanHuarte/Ising_DMRG_Ivan_Huarte_Franceschini/assets/136732928/725243b7-f852-48a3-a999-4877457552e9)


## Bonus: iDMRG
I also implemented a code segment at the bottom which runs iDMRG. Unfortunately, I have no more time to see the results and adjust the settings to do a good performance. 

Thank you very much. I enjoyed the task and learned a few things.
Bests.
Iván

