# Drift_Orbits_3bp

I. Requirements:
	The CAPD 5.1 (Computer Assisted Proofs in Dynamics) library.
		
	Unzip the file "capd-capdDynSys-5.1.2.zip" into your home folder (the home folder is suggested since then
	no changes to the makefile for the proof should be necessary (see II.4 below)).

	From the folder into which the source code was unzipped, in terminal, type:

	./configure
	make

	This compiles the CAPD package into the "capd-capdDynSys-5.1.2" folder. To remove it, simply delete the folder. 
	The installation does not “pollute” the system in any way.
	
	Further information, documentation and latest version of the CAPD library is available at:

	http://capd.ii.uj.edu.pl
	

II. Compiling and executing the proof for the paper.

	1. Unzip the file "DriftOrbits3bp.zip" in any folder location of your choice.

	2. To compile (from the folder into which it was unzipped), in terminal, type:

	make

	3. To execute the proof, from terminal, type:
	
	./DriftOrbits3bp

	4. Makefile changes:

	If the CAPD-DynSys library has been compiled in the home folder, then no changes
	should be necessary in the makefile.

	If the CAPD-DynSys library has been compiled outside of the home folder, 
	then the line

	CAPDBINDIR = ~/capd-capdDynSys-5.1.2/bin/

	in the makefile should be modified to the location containing the CAPD-DynSys library. 


III. In what order to read the files for the proof. 
(Following this order helps to trace how the argument is conducted.)


	1. LyapOrb.h/cpp
	2. DLyapParallelShooting.h/cpp
	3. ILyapParallelShooting.h/cpp
	4. wuEnclosure.h/cpp
	5. wsEnclosure.h/cpp
	6. persistenceOfNHIM.h/cpp
	7. IHomParallelShooting.h/cpp
	8. MelnikovComputation.h/cpp
	9. 3bp.cpp

	A description of the purposes of the routines and the contents of the files:

1. LyapOrb.h/cpp 
	These files contain two functions: 
	LyapPoint(x) which computes a point (x,0,0,py) on a Lyapunov orbit, for a given value of x.
	findTime(q) which computes the time to section {Y=0} when integrating from the vector q.
	Both these functions are non-rigorous. They are used to provide an initial guess, that is 
	later validated by using rigorous numerics.

2. DLyapParallelShooting.h/cpp
	The main function of interest is LyapParallelShooting(). This function computes (non-rigorously) 
	a sequence of points along a Lyapunov orbit. 
	This provides an initial guess for the interval arithmetic 	validation in ILyapParallelShooting files.
	One additional purpose of this function is a computation of a matrix C, which is then used in 
	ILyapParallelShooting for the Krawczyk method. 

	Note: Parallel shooting requires appropriate setup of the derivative of the `shooting operator'. 
	Such derivative requires inserting various fragments into a `big matrix'. 
	These tasks are handled by technical functions from OperationsForShooting.h/cpp.

3. ILyapParallelShooting.h/cpp
	The main function of interest is pointsOnLyapOrb() which computes a sequence of points along a Lyapunov orbit. 
	This is a rigorously validated enclosure, by means of parallel shooting and Krawczyk method. 
	Apart from the points along the orbit, the function also computes and validates a bound on the time of 
	integration between the points. This is the interval t, passed by reference.

4. wuEnclosure.h/cpp
	The main routine here is validateWuEnclosure(). This function computes the bounds on the unstable 
	fibres, based on points along the Lyapunov orbit. The bounds are in local coordinates given by linear 
	changes A[I]. The fibres are contained in cones, which are computed by this routine.
	An additional function of interest here is validateFiberInExtendedPhaseSpace(). 
	This validates the bound on the fiber, based at the first point along the Lyapunov orbit. 
	The objective is to obtain the bound in the extended phase space. Such bound is then used for 
	the computation of the scattering map along the angle coordinate.

5. wsEnclosure.h/cpp
	Here we obtain mirror bounds (for the stable fibres) to the ones obtained in 4 by using 
	the R-symmetry of the PCR3BP.

6. persistenceOfNHIM.h/cpp
	Here we check the twist conditions needed for KAM. The main function of interest is 
	validatePersistenceOfNHIM(). 
	The twist coefficients are computed from the parallel shooting method described in the paper; 
	this is done by the function twistCoefficients(). Then such coefficients are checked that they are 
	nonzero in validatePersistenceOfNHIM(), and written out by the function.

7. IHomParallelShooting.h/cpp
	This is used to compute and validate bounds on the homoclinic orbit. This is done by using 
	the Krawczyk method. An initial guess of the homoclinic is computed by non-rigorous parallel 
	shooting method from DHomParallelShooting.h/cpp (where the function HomoclinicParallelShooting() 
	computes an initial guess as well as the matrix C needed for the Krawczyk method). 
	The functions of main interest in IHomParallelShooting are:
		- pointsOnHomoclinic()
	which computes half of the points along the homoclinic by validating their bounds with the Krawczyk method,
		- pointsOnFullHomoclinic()
	which completes the half number of points to the full homoclinic from symmetry,
		- establishTransversality()
	which establishes transversality of the intersection at section {Y=0}.

8. MelnikovComputation.h/cpp
	This file contains functions, which perform the computation of the scattering map of the 
	unperturbed system along the angle coordinate. (On actions the scattering map is constant, 
	so there is not need to compute bounds.) This is done by the function scatteringMap() which 
	is based directly on the estimates from the paper.

	The function find_m_iterates() computes the number of iterates m(z) from the paper.
	The function checkStripTransition() establishes that the inner dynamics starting from 
	angle theta will enter after some number of iterates in the given strip.

	Another function of interest is LgBound(). This function computes the bound on the coefficient L_g 
	needed for the application of the main shadowing theorem. This computation is based on the method 
	described in the appendix of the paper.

	The next function of interest is 
		gI(p,&theta,t,&Phi,&P,&H).
	The "p" is a bound on the points along a homoclinic orbit. The time "t" is the bound on the time 
	from point to point along the flow. We take the homoclinic in extended phase space with initial 
	angle equal to "theta". This means that the homoclinic in extended phase space is
		x[i] = (p[i],theta+i*t).
	This gI() function computes:
		\Sum_{i=0}^{4} \pi_I g(0,p[i]).
	Since we take I(x)=H(x), we use the Hamiltonian of the PCR3BP as coordinate I. Above we write five 
	coefficients in the sum, since this sum corresponds to the five iterates of the Poincare map from {Y=0} to {Y=0}. 
	In the code, instead of the Poincare map we consider time t maps along the flow as our local maps, and then the 
	map to section {Y=0}. The effect is the same: We can represent f^5 (where f is the map to section {Y=0}) 
	as f o Phi_t^(n-1); in other words
		f^5(x) = f o Phi_t^(n-1) (x).
	What the function gI() computes is the bound on
		\pi_I (f o Phi_t^{\eps}^(n-1) (x) - x)  \in \eps*Bound.
	The interval returned by gI() is an interval Bound from above, which we write as dH in the function. Since 
		f_{\eps}^5(x) = f o Phi_t^{\eps}^(n-1) (x)
	this has the same effect as computing a bound on
		\pi_I (f^5_{\eps}(x) - x).

	The final function is 
		gI(p,A,Cone,&Domain,m,&theta,t,&Phi,&P,&H).
	This performs the same tasks as the previous gI(), but instead of computing a bound on 
		\pi_I (f^5_{\eps}(x) - x)
	for a point x along a homoclinic, it computes a bound on 
		\pi_I (f_{\eps}(x) - x) 
	for a point from the stable manifold of the Lyapunov orbit, in its close neighbourhood; 
	which we call "Domain" in the code. In this function we make use of the knowledge that after each 
	iterate of the map, the point approaches closer to the Lyapunov orbit, so the "Domain" shrinks. 
	This means that the next time we compute gI() we can do so along the smaller domain, which improves 
	the estimates, since integration is performed on smaller and smaller sets as we consider points 
	closer to the Lyapunov orbit. This turned out to be important, since the m(z) in the paper is a large number, 
	and we need to use this function for m(z)-5 times. With each iterate Domain shrinks, and the accuracy is improved.

9. The actual proof is performed in the 3bp.cpp file, where we bring all of the above components together.
	There we perform the following steps:
   	1.  Validate bounds on points around a family of Lyapunov orbits.
    	2.  Compute bounds (based on cones) on the position of the unstable manifold of Lyapunov 
		orbits intersected with {Y=0}.
    	3.  Compute bounds on the fibres from 2., but including the extended phase space. 
		(We compute the constant M on fibres in extended phase space from the paper.)
    	4.  We automatically obtain mirror bounds for the stable fibers from the symmetry of the PCR3BP.
    	5.  We validate the twist conditions for persistence of NHIM.
    	6.  We compute bounds on homoclinic orbits for our Lyapunov orbits. We also establish that such 
		homoclinics lie along transversal intersections.
    	7.  Computation of the Lg bound.
    	8.  Validation of energy changes from Sd to Sd.
    	9.  Validation of energy changes from Su to Su.

If the code reaches the end of the main() function, this means that the proof has been conducted successfully. 
The message "The proof ended successfully." will be displayed.
