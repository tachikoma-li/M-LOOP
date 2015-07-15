# MLOOP
Machine-learning online optimization package

<><><><><><><>
Installation
<><><><><><><>

Get anaconda.

Here we describe the required formats for the files:

<><><><><><><>
ExpOutput.mat
<><><><><><><>

This file must be generated by the experiment and is how the learner finds out about the latest run. It must contain the following three variable

cost (float)
uncer (float)
bad (boolean)

cost is the cost determined by the experiment, uncer is the uncertainty related to that cost (absolute uncertainty) and bad is a boolean indicating whether the run was fine or if it was a bad one.

If using NelderMead or Random, when a bad run occurs, cost and uncer must be provided, but, they can be safely set to NaN. However when using learners, you currently _must_ provide non NaN numbers every run, even a bad one. The learners will intelligently forget bad runs if a good run is later performed with the same parameters, but it can not handle NaN or inf values for these bad runs. (This behavior could be revisited in future versions)

The user is also free to add as many other variables as they want to file. They will all be recorded and appropriately saved during the run. 

The controller will continuously save a matlab file ExpTracking.mat which saves all the costs uncertainties and bads into arrays, plus any user defined variables.

Currently you will have be consistent with what you put in the matlab file for each run. If you add a new variable half way through the experiment, the program will halt. (This behavior could be revisited in future versions)

<><><><><><><>
ExpConfig.txt
<><><><><><><>

This file gives the bounds, type and initial conditions to the learner/optimizer. 

The file is parsed (reasonably) intelligently. Meaning extra whitespace, extra new lines and the order of the lines should not be an issue. Nevertheless, try and stay as close to the examples as you can.

Each parameter must now be input as

DescriptionOfParameter=(integer, string or array of whatever the Parameter is)

Here are a couple of examples of formatting for integers, arrays of floats and strings:

NumberOfRuns=1
MinimumBoundaries= [1.0 , 3.0, 4.0 ]
ControllerType = 'NelderMead'

In the above examples you can again see we can be reasonably flexible with white spacing. Just be sure to put either ' or " quotations around strings on the RHS of expressions.

The configuration file will be very different depending on what solver is used. Hence we will explicitly define the config file for each solver individually, even though they do have some common properties.

=================
NelderMead Solver 
=================
=================

The Neldermead solver has the following common settings:

ControllerType='NelderMead'
NumberOfRuns=(int)
NumberOfParameters=(int)  

Note controller type must be 'NelderMead'. Next we have the boundary conditions which can be one of the following:

BoundaryConditions ='Hard'
MinimumBoundaries=(arrays of floats of size NumberOfParameters)
MaximumBoundaries=(arrays of floats of size NumberOfParameters)

OR

BoundaryConditions ='None'

Next we provide the system with initial conditions. There are currently three options: Explicit, Scaled and Random described below.

First Explicit:

InitialConditionsSetup=  'Explicit'
InitialParameters=(arrays of floats of size NumberOfParameters)  
InitialSimplexDisplacements=(arrays of floats of size NumberOfParameters)  

Here the initial conditions will be a simplex formed with one corner being the InitialParameters, then each of the points of the simplex being the initial point displaced by one of the numbers from the InitialSimplexDisplacements array.

Next Scaled:

InitialConditionsSetup=  'Scaled'
InitialParameters=(arrays of floats of size NumberOfParameters)  
SimplexScaleSize=(float)  

This produces a simplex with one corner of being the InitialParameters, then each other corner being these InitialParameters plus the difference between the boundaries scaled by SimplexScaleSize. Explicitly the points of the simplex x[i] are given by:
x = initParameters 
x[i] = x[i] + ScaleOfSimplex*(MaximumBoundary - MinimumBoundary)[i]

If you have no boundaries or want it scaled according to different values you can instead write it as:

InitialConditionsSetup=  'Scaled'
InitialParameters=(arrays of floats of size NumberOfParameters)  
SimplexScaleSize=(float) 
MinimumInitialParameters=(arrays of floats of size NumberOfParameters)
MaximumInitialParameters=(arrays of floats of size NumberOfParameters)

MinimumInitialParameters and MaximumInitialParameters are now used instead of the boundaries. 

Lastly we have Random: 

InitialConditionsSetup=  'Random'  
SimplexScaleSize=(float)  

Random works in a very similar fashion to Scaled except it also picks the InitialParameters from a uniform distribution, hence you don't need to explicitly provide it. Again, if you want Random to sample from a different set of intervals you can instead write:

InitialConditionsSetup=  'Random'  
SimplexScaleSize=(float)  
MinimumInitialParameters=(arrays of floats of size NumberOfParameters)
MaximumInitialParameters=(arrays of floats of size NumberOfParameters)

Now we give a few examples of valid Nelder Mead Controller congif files

---

NumberOfRuns = 100
NumberOfParameters=3  

ControllerType='NelderMead'

BoundaryConditions ='Hard'
MinimumBoundaries= [1.0 , 3.0 , 4.0 ]
MaximumBoundaries= [10.0,10.0,10.0]

InitialConditionsSetup=  'Explicit'
InitialParameters  =[2.0,4.0,5.0]  
InitialSimplexDisplacements= [1.0,1.0,1.0]

---

NumberOfRuns = 19
NumberOfParameters=2  

ControllerType='NelderMead'

BoundaryConditions ='Hard'
MinimumBoundaries= [-1.0, -1.0]
MaximumBoundaries= [1.0,1.0]

InitialConditionsSetup=  'Random'
SimplexScaleSize= 0.1

---

InitialConditionsSetup=  'Random'
MinimumInitialParameters =[-1.0,-2.0]
MaximumInitialParameters  =[10,20]
SimplexScaleSize= 0.1

NumberOfRuns = 200

NumberOfParameters=2  

BoundaryConditions ='None'

ControllerType='NelderMead'

---

=======
Random
=======
=======

This optimizer/learner, isn't really either of these options. It's more of a randomized brute force approach. It just randomly tries a parameter given the boundaries. As a file it must be formatted as follows:

ControllerType='Random'
NumberOfRuns=(int)
NumberOfParameters=(int)  
MinimumBoundaries= (arrays of floats of size NumberOfParameters)
MaximumBoundaries= (arrays of floats of size NumberOfParameters)

An example of a correctly formatted random:

---

ControllerType='Random'
NumberOfRuns=100
NumberOfParameters=10  
MinimumBoundaries=[-10.0,-10.0,-10.0,-10.0,-10.0,-10.0,-10.0,-10.0,-10.0,-10.0]
MaximumBoundaries=[10.0,10.0,10.0,10.0,10.0,10.0,10.0,10.0,10.0,10.0]

---

=========
Learner
=========
=========

The learners work by creating a gaussian process to estimate the shape of the landscape, first based on a set of training data and then after every step it iteratively improves its estimate of the landscape.

At the moment there are is only one documented solver, the 'Global' version. In future there will be an 'Auto' version.

The GlobalLearner performs a set of sweeps where it goes from prioritizing new information to prioritizing finding the minima. More specifically, given a starting point it finds two points, one with the minimum cost and one with the maximum uncertainty. If they are the same point, it simply evaluates it. If there is a possible tradeoff between the two, it then attempts to find a new point with based on new cost function that balances between the cost and the uncertainty with regard to the current set balance ratio. The balance ratio is between 0 and 1 and it the value that is sweep repeatably through the run. A single run of the cycle is reminiscent of an annealing algorithm. As it initial prioritizes global searching and then moves towards refining the local optimal. The most important difference is that here the use will typically perform multiple sweep, and each old sweep informs the last. Indeed one may choose to have less points in sweep than the total number of sweeps performed, which is very different to the annealing algorithm. 

Internally the learners attempts to logically normalize the parameters and costs internally to hopefully improve robustness and consistency of performance. It uses the boundaries to normalize the parameters. In order to normalize the costs it the user must provide an expected minimum and maximum cost. The algorithm should still function even if the cost occasionally falls slightly out of these bounds. But if the cost falls significantly outside of the prescribed minimum cost the algorithm can become unstable. So always try to use a lower bound when providing the minimum cost. 

Common Learner Settings
~~~~~~~~~~~~~~~~~~~~~~~

All learners require the ControllerType currently it must be

ControllerType='GlobalLearner'

The learner must then be provided the following set of data:

NumberOfParameters=(int)
NumberOfRuns = (int)
MinimumBoundaries= (array of floats of the size NumberOfParameters)
MaximumBoundaries= (array of floats of the size NumberOfParameters)
MinimumCost = (float)
MaximumCost = (float)

Which are mostly self explanatory and/or the same as previous controllers. 

There are three options for the initial conditions: Random, Simplex or FromFile. 

Random
~~~~~~~

The random initial training will simple try uniformly distributed random points inside the boundaries. The required definitions are:

InitialTrainingSource='RandomSearch'
NumberOfTrainingRuns=(int)

The total number of runs the simulation will go for will now be NumberOfRuns + NumberOfTrainingRuns. You can optionally prescribe custom boundaries just for the random search:

MinimumInitialParameters= (array of floats of the size NumberOfParameters)
MaximumInitialParameters= (array of floats of the size NumberOfParameters)

Under the hood the controller will initiate a RandomSearch controller to get the training data.

Simplex
~~~~~~~~

Here the controller will create the initial points from a simplex. Under the hood the controller actually initiates a NelderMead controller, hence if you can also prescribe a certain number of runs and the training data will then be formed from an initial Nelder Mead run. The required definitions are:

InitialTrainingSource='Simplex'
InitialParameters=(array of floats of the size NumberOfParameters)
SimplexScaleSize=(float between 0.0 and 1.0)

By default the number of runs with be the NumberOfParameters + 1 + NumberOfRuns. But if you want the training data to be based on a NelderMead run you can add the additional optional definition:

NumberOfTrainingRuns=(int)

You can also provide custom boundaries for the NelderMead and simplex training run:

MinimumInitialParameters= (array of floats of the size NumberOfParameters)
MaximumInitialParameters= (array of floats of the size NumberOfParameters)

FromFile
~~~~~~~~~

Lastly you can train the learner from any previous run. This includes previous learning runs. Simply provide the _python_ archive of the previous run. By default this is output as 'ControllerArchive.pkl'. (In future version we may allow import from matlab files). The following is required to take from a file:

InitialTrainingSource='FromFile'

Optionally you can also provide the specific file name as:

FileName=(string)

Optional Common Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~

All learners use common code when searching for the next point to sample. The only difference between the algorithms is how much they prioritize new information of minimizing cost. The learner can spend a vast amount of resources when trying to best estimate the next point to sample, if you're finding the solver it taking longer to find a new point then the downtime of the experiment. You can make reduce the learners Accuracy, but vastly increase it's speed with the following settings.

The primary method to speed up your solver is by using the tag

NumberOfParticles = (int)

*DOCUMENT*

NumberOfThetaSearches = (int)

*DOCUMENT*

NumberOfParameterSearches = (int)

*DOCUMENT*

Learner specific settings
~~~~~~~~~~~~~~~~~~~~~~~~~

GlobalLearner
~~~~~~~~~~~~~

The global learner has a many unique setting related to how the sweep is performed. However, all of them have a default setting. The first quantity you can change is the number of runs in a sweep.

RunsInSweep = (int)

The default value for RunsInSweep = 5. This gives reasonable performance for all dimensions.

Custom Ramp
-----------

The default ramp is a linear sweep from SweepMinBias to SweepMaxBias. If this does not suit your needs you can actually completely customize the ramps function using:

SweepRampFunction = (lambda function which takes one integer and returns a float between 0.0 and 1.0)

Under the hood lambda functions are actually what are used to implement default ramp. Here we let the user directly make such a function. This is mostly included just to show off how flexible python is, and how clever the input method we are using is. If you use this trick make sure the lambda function behaves sanely. It will check it with a few random input integers, but that's it.

Putting the solver on a leash
-----------------------------

In high dimensional spaces the learners propensity to search the space, and particularly the boundaries might become highly inefficient. If instead you want the learner to only randomly walk thought the search space instead of completely freely probing it you can put it on a leash. In practice the leash puts bounds on the minimizer when searching the predicted landscape based on the initial starting point it is given. This means there is a maximum distance the learner can travel from any of the points it has previously visited. Using this approach ensures that the learner can still in principle search the space, however it has to do so in a restricted manner. 

The definition you need to add to turn the leash on is

LeashSize = (float between 0.0 and 1.0)

The leash size refers to the fraction of the difference between the minimum and maximum boundaries that the controller can maximally move in each direction. By default the Leash is set to LeashSize = 1.0, basically meaning it can go anywhere. But you can set it to be whatever length you want.

Much like the sweep function, if you instead want the leash to grow or shrink as a function of the run number you can instead specify a LeashFunction:

LeashFunction = (lambda function which takes one integer and returns a float between 0.0 and 1.0)

Again under the hood, all leashes are implemented with lambda functions. Here we are just giving the user direct control. Note that if LeashFunction is defined the program will ignore LeashSize if it is also specified.

Examples
--------

---

ControllerType='SimpleLearner'

NumberOfRuns = 10

NumberOfParameters=3
MinimumBoundaries= [-1.0,-2.0,-3.0]
MaximumBoundaries= [0.0,0.0,0.0]
MinimumCost = 0
MaximumCost = 1

InitialTrainingSource='Random'
NumberOfTrainingRuns = 100

---

ControllerType='GlobalLearner'

NumberOfRuns = 12

NumberOfParameters=2
MinimumBoundaries= [0.0,0.0]
MaximumBoundaries= [5.1,5.2]
MinimumCost = -2
MaximumCost = 3

InitialTrainingSource='Random'
NumberOfTrainingRuns = 100
MinimumInitialParameters= [1.9,2.9]
MaximumInitialParameters= [2.1,3.1]

---

ControllerType='GlobalLearner'

NumberOfRuns = 312
RunsInSweep = 50

NumberOfParameters=1
MinimumBoundaries= [-10.0]
MaximumBoundaries= [10.0]
MinimumCost = -1
MaximumCost = 0.5

InitialTrainingSource='Simplex'
InitialParameters = [-5]
SimplexScaleSize= 0.4
NumberOfTrainingRuns = 20

SweepMinBias = 0.1
SweepMaxBias = 0.9

---

ControllerType='GlobalLearner'

NumberOfRuns = 256
RunsInSweep = 8

NumberOfParameters=9
MinimumBoundaries= [-9,-8,-7,-6,-5,-4,-3,-2,-1]
MaximumBoundaries= [9,8,7,6,5,4,3,2,1]
MinimumCost = -1
MaximumCost = 10

InitialTrainingSource='FromFile'
FileName = 'ControllerArchiveBack.pkl'


---

ControllerType='GlobalLearner'

NumberOfRuns = 312

NumberOfParameters=3
MinimumBoundaries= [-9,-8,-7]
MaximumBoundaries= [-1,-1,-1]
MinimumCost = -9
MaximumCost = -8

InitialTrainingSource='Simplex'
InitialParameters = [0.0,0.0,0.0]
SimplexScaleSize= 0.4

SweepFunction = lambda x: (x%10+1)/10.0

---



