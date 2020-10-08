# General Model Setup:

pyCycle requires a "design" case to be run at some reference flight condition.
In general, the specific condition is semi-arbitrary, but for this model a
top of climb flight condition is choose (MN=0.8, alt=35000 ft). 
At the design case, you can specify propulsion design variables such as FPR, Fn_DES, or T4_MAX. 

To evaluate engine performance at any other flight condition, you run "off-design" points. 
At these points, in typical propulsion modeling you would specify the flight condition (MN, alt) and some kind of throttle parameter (e.g FAR, T4, or Power Code). 
Then the outputs are all the performance metrics of the engine (e.g. thrust, BPR, fuel flow, temperature, FPR, etc. )

You will always run ONE design point, and one or more off-design points. 

# Modifications for aeropropulsive coupling

## Design Case
The design case does not need to be directly coupled to CFD. 
You could do so if you wished, but its not necessary. 
This condition just serves as a reference, so it should be sufficient to fix the inlet flow conditions to something reasonable in order to get the reference areas. 

You will end up with optimization design variables that are the inputs to the design case. 
For example, as the model is currently set up the design case sizes the engine by the `Fn_DES` variable. 
If you wanted a bigger engine, you would increase that value (or vice versa). 
The impact of these design variables on the odd-design cases will be felt by the areas and map scalars that are passed from the design case to off-design cases. 

## Off-Design Cases
The off-design cases will get their flow-initialization via the CFDStart element --- 
see section III.A of [this paper](https://arc.aiaa.org/doi/pdf/10.2514/1.C035845). 
The CFDStart element requires the following inputs: static pressure (area averaged), velocity (mass averaged), flow area, mass flow rate. 

In a typical off-design case, engine mass flow is actually a computed output. 
Given a fixed geometry nozzle, the conservation of mass is managed by varying mass-flow until the computed nozzle area matches the specified area from the design case. 

In a coupled CFD-propulsion moodel we have two options for how to deal with this: 

### Use mass-flow as a coupling variable 
If you choose to follow the standard engine modeling convention, then mass-flow is a computed output from the engine model. 
The CFD would take an input (from the propulsion model) of mass-flow. 
It would then internally iterate to find a static pressure at the out-flow condition that matches that mass flow. 
Then the CFD can output (to the propulsion model) mass-averaged velocity, area, and the static pressure value that was iterated on to converge mass-flow. 
(Note related to the derivatives: this means that the implicit relationship in that boundary condition needs to be included as an additional state variable in the adjoint CFD adjoint)

### Use static pressure as a coupling variable
A standard subsonic outflow condition in CFD can output (to the propulsion model) mass-flow, area, and mass-averaged velocity. 
It requires a static pressure at the outflow condition as an input. 
So the goal would be to have static pressure be an output from the propulsion model. 

The CFDStart requires mass-flow, area, mass-averaged Velocity and area averaged static pressure as inputs. 
Three of those values are output by the CFD, but the static pressure is missing. 
We can modify the traditional conservation of mass balance in the propulsion model to account for this. 
Normally we would vary mass-flow till conservation of mass is satisfied. 
Instead, given a fixed mass-flow we can vary the static pressure in the CFDStart to satisfy conservation of mass. 


