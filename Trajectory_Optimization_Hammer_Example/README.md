Multiple Shooting Tutorial - Hammer Example

===============================================

This collection of matlab files can be used to demonstrate how to set up 
and solve a multiple shooting problem for a simple system with continuous
and discrete dynamics. In this case, it is finding the optimal torque 
for a periodic hammering motion.


1) Run MAIN.m
2) Edit settings (if desired) in Get_Parameters.m
3) The details of the multiple shooting algorithm are largely found in 
   Compute_Defects.m. 
4) Three plots will be generated, showing the progress of the optimization
   algorithm. They are created by Plotting_Script.

This code uses the matlab function fmincon to make it more portable. 
There is some evidence that SNOPT is better. There is a slight difference 
in how the algorithms are called, but they're pretty close.


Basic outline for setting up multiple shooting (follow along in the code):

1) Define all parameters for the problem.  Get_Parameters.m

2) Make an initial guess at the state. This can be done a number of ways, 
   but the general idea is guess a few points on the trajectory, and then
   interpolate between them. Make this as good as possible for complex 
   systems.  Initial_State.m

 ->NOTE: I've found that the best way to keep track of the decision 
         variables (sometimes I call them the State or X) is in a struct. 
         This makes the code easier to write and less prone to the nasty
         sort of bugs. The downside is that the optimization algorithms 
         cannot deal with structs. The solution is to write a fucntion that
         converts between a vector of doubles and a struct. In my code this
         is Convert_State.m. The basic idea is that a struct can be
         converted to a vector (storing the data) and a struct (storing the
         fieldnames and sizes) with no information lost.

3) Set the boundaries on the "State" (i.e. decision variables). This is  
   done in the file: Get_Bounds.m. This function just reads the boundaries 
   for each of the states (as defined in Get_Parameters.m) and then 
   multiplies them by the correct size matrix so that they line up with 
   each of the multiple shooting states.

4) Create an objective function: Objective_Function.m. In this case, I've
   set the objective to just be the integral of the actuator torque 
   squared. 

 ->NOTE: The objective function MUST be smooth for the optimization to 
         converge. One example is that you cannot use the function abs(). 
         Another is that you can use a simple saturation function on your 
         actuators. Doing something like cost = max(u) will also cause a
         slow convergence (or outright failure). I've included two extra
         functions in this directory to help with smoothing:

         Smooth_Saturation_Vector.m
         SmoothAbs.m

	 Although smoothing a discontinuous function will work, it is not
  	 the desired method. Rather, you should replace the discontinuous 
	 function with a set of constraints. For example, abs(x) should be:

	 min(x1+x2) subject to:

 	 x1 > 0;  x2 > 0; x = x1-x2;   
  
5) Define the nonlinear constraints: NonLinCon.m
   This is where all of the physics occurs. It is key that the dynamics
   functions take vectorized inputs. This makes the code easier to write 
   and makes the optimization run much faster. If you're using fmincon, 
   speed can be further increased by setting the "UseParallel" option. It
   is easist to see what is happening by reading the function:
   Compute_Defects.m.
   In this simple example, there are no complicated constraints besides the
   defect constraints. In other applications, this function might get long 
   and complicated. 

  ->NOTE: When using SNOPT, a single function call is used to get both the
          objective function value and constraints (all kinds). SNOPT reads 
          the first element of this vector as the objective function value
          and treats all subsequent elements as constraints.
 
 ->NOTE: When using SNOPT to solve a feasibility problem (satisfy 
         constraints, don't optimize anything), the first element of the
         vector that is returned by your objective function should be 0,
         and it should have bounds of zero. 
         (i.e. F(1)==0, Flow(1)==0, Fupp(1)==0)

 ->NOTE: When using SNOPT, DO NOT HAVE ANY CONSTANT TERMS IN YOUR OBEJCTIVE
         FUNCTION OF CONSTRAINTS. All constant terms must be subtracted off
         and stored in the bounds. This is because of how SNOPT internally
         approximates and tracks the constraint values.

 ->NOTE: The representation of the torque function is important. Here I use
         a piecewise linear torque. Note that the Runge-Kutta steps occurs 
         over a linear section of the torque curve. This means that any 
         given integration step does NOT integrate over the discontinuity 
         in the first derivative of the torque curve. This is important. 

-> NOTE: I use a fixed-step integration method. This is not by accident - 
         although a varaible step method might be more accurate, it is not
         as consistent as the fixed-step method. This consistency is 
         important for getting the optimization to converge quickly.

6) This problem has no linear constraints, so set them all as [] for now. 
   Note: when using SNOPT there is no need to seperate out the linear and 
   nonlinear constraints.

7) The last step is to set the options for the optimization. I've only set
   a few of them here, but there are many many more. To see all of the 
   available options, type 'optimset()' at the command line.

8) I choose to pass the inputs to the optimization as a Problem struct, 
   this makes book keeping easier.

9) Run the problem and check out the results!


 ->NOTE: I've glossed over the derivation of the dynamics because they are 
         simple for this problem. Check the functions:
         HammerDynamics.m and HammerImpact.m for more details. For 
         complicated problems it is typically a good idea to use computer
         algebra to solve the dynamics problem.


