##################################################################################
##################################################################################
####################### PHYSICAL SYSTEM (optional reading)  ######################
##################################################################################
##################################################################################

# Dynamics of a chain of laser-driven interacting atoms — dataset

This dataset contains the time evolution of local magnetizations and pairwise correlations in a
chain of 20 interacting atoms driven by a laser. It contains 500 simulations (trajectories). All of
them start from the same initial state and differ only in the laser settings. The data is generated
by `generate_dataset_ed.jl` (Julia, QuantumToolbox).

## The physical problem in brief

The quantum system: we consider 20 atoms arranged in a line. Each atom is treated as a qubit: a
quantum system with two possible states, which we can think of as "down" and "up". The atoms
interact with their nearest neighbors. The interaction makes the energy depend on whether two
neighboring atoms are in the same state or in opposite states. Its strength is fixed and sets the unit of energy and of time. 

The laser: all atoms are illuminated by the same laser field, described by two parameters:

- Omega is related to the amplitude (intensity) of the laser. It sets how strongly the laser
  drives each atom back and forth between its two states. Larger Omega means faster flipping and
  stronger creation of quantum superpositions.
- Delta is related to the frequency of the laser, more precisely to how far the laser
  frequency is detuned from the atomic transition. It acts as an energy bias that favours one of
  the two atomic states over the other.

The initial state At time zero every atom is prepared in the same quantum state: The atoms are not correlated with each other, and on average each atom is neither up nor down. The initial state is the same for all trajectories.

The dynamics: At time zero the laser is switched on with a given Omega and Delta, and the atoms
evolve according to the laws of quantum mechanics. Interactions, driving and detuning compete, and depending on their balance the system can oscillate regularly, relax quickly towards an
equilibrium-like state, or stay almost frozen.

What is measured: At each sampled time two kinds of quantities are recorded:

- Magnetizations, one per atom: the average value of that atom's up/down state. It ranges from
  −1 (certainly down) to +1 (certainly up); 0 means equally likely.
- Connected correlations: one per pair of atoms: how statistically correlated the two atoms
  are, beyond what their individual magnetizations already explain. Zero means independent; they from -1 to +1.

##################################################################################
##################################################################################
####################### DATA STRUCTURE ###########################################
##################################################################################
##################################################################################

The file trajectories.csv contains the dynamics of magnetizations m_j and connected correlations c_ij of the initial state |−−...−⟩ under the quantum dynamics of a system composed of N qubits, with parameters Ω and Δ related to the amplitude and the frequency of the laser field applied to the qubits.

We consider N_traj trajectories that correspond to N_traj different pairs (Ω, Δ), drawn uniformly at random with Ω in [−2, 2] and Δ in [0, 2]. For each trajectory we simulate the dynamics of the initial state and compute at each time N magnetizations (m_1, m_2, ..., m_N) and N(N-1)/2 connected correlations (c_12, c_13, ..., c_1N, c_23,..., c_2N,...,c_{N-2,N-1},c_{N-2,N},c_{N-1,N}). The dynamics is sampled at the time values 0,dt,2*dt,...,(N_points-1)*dt, where (N_points-1)*dt=T_fin is the final sampled time.

For each trajectory, the first column contains the time values (0,dt,2*dt,...,T_fin), then the other columns the N magnetizations (m_1, m_2, ..., m_N) at each time and then the N(N-1)/2 connected correlations at each time (c_12, c_13, ..., c_1N, c_23,..., c_2N,...,c_{N-2,N-1},c_{N-2,N},c_{N-1,N}). Thus, we have total 1 + N + N(N-1)/2 = 1 + N(N+1)/2 columns. The number of rows for each trajectory is N_points.

At any 0 on the first row (times), a new trajectory starts, so the total number of rows in the file is N_points x N_traj.

The file parameters.csv contains one row per trajectory, in the same order as in trajectories.csv, with columns traj_id, Omega, Delta; the other two columns max_bond_dim, hit_maxdim are not important for the project.

We consider N=10, T_fin=5, dt = 0.01, N_points = 501, N_traj=500; thus the number of columns is 56 and the number of rows is 250500.

##################################################################################
##################################################################################
####################### MACHINE LEARNING TASK ####################################
##################################################################################
##################################################################################

Train ML models to predict the quantum dynamics, i.e. the N magnetizations and the N(N-1)/2 correlations, for given laser parameters (Ω, Δ). The data are split into training and test sets along two directions:

Trajectories: the N_traj trajectories are divided into N_traj_train training trajectories and N_traj_test = N_traj − N_traj_train test trajectories, so that the model is tested on pairs (Ω, Δ) not seen during training.
Times: the N_points times of each trajectory are divided into the first N_points_train times (0, dt, ..., (N_points_train − 1)*dt), used for training, and the remaining N_points_test = N_points − N_points_train later times, used for testing, so that the model is tested on its ability to predict the dynamics at later times.

Possible questions to address:

-How the performance of the model depends on (Ω, Δ): compute the test error for each test trajectory and plot it as a function of (Ω, Δ), to identify which dynamical regimes are easy or hard to learn.
-How the performance changes with the number of training times N_points_train, at fixed number of training trajectories.
-How the performance changes with the number of training trajectories N_traj_train (e.g. 25, 50, 100, 200, 400), at fixed number of training times and on the same test trajectories.


