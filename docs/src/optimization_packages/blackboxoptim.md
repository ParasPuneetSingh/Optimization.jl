# BlackBoxOptim.jl

[`BlackBoxOptim`](https://github.com/robertfeldt/BlackBoxOptim.jl) is a Julia package implementing **(Meta-)heuristic/stochastic algorithms** that do not require differentiability.

## Installation: OptimizationBBO.jl

To use this package, install the OptimizationBBO package:

```julia
import Pkg;
Pkg.add("OptimizationBBO");
```

## Global Optimizers

### Without Constraint Equations

The algorithms in [`BlackBoxOptim`](https://github.com/robertfeldt/BlackBoxOptim.jl) are performing global optimization on problems without
constraint equations. However, lower and upper constraints set by `lb` and `ub` in the `OptimizationProblem` are required.

A `BlackBoxOptim` algorithm is called by `BBO_` prefix followed by the algorithm name:

  - Natural Evolution Strategies:
    
      + Separable NES: `BBO_separable_nes()`
      + Exponential NES: `BBO_xnes()`
      + Distance-weighted Exponential NES: `BBO_dxnes()`

  - Differential Evolution optimizers, 5 different:
    
      + Adaptive DE/rand/1/bin: `BBO_adaptive_de_rand_1_bin()`
      + Adaptive DE/rand/1/bin with radius limited sampling: `BBO_adaptive_de_rand_1_bin_radiuslimited()`
      + DE/rand/1/bin: `BBO_de_rand_1_bin()`
      + DE/rand/1/bin with radius limited sampling (a type of trivial geography): `BBO_de_rand_1_bin_radiuslimited()`
      + DE/rand/2/bin: `de_rand_2_bin()`
      + DE/rand/2/bin with radius limited sampling (a type of trivial geography): `BBO_de_rand_2_bin_radiuslimited()`
  - Direct search:
    
      + Generating set search:
        
          * Compass/coordinate search: `BBO_generating_set_search()`
          * Direct search through probabilistic descent: `BBO_probabilistic_descent()`
  - Resampling Memetic Searchers:
    
      + Resampling Memetic Search (RS): `BBO_resampling_memetic_search()`
      + Resampling Inheritance Memetic Search (RIS): `BBO_resampling_inheritance_memetic_search()`
  - Stochastic Approximation:
    
      + Simultaneous Perturbation Stochastic Approximation (SPSA): `BBO_simultaneous_perturbation_stochastic_approximation()`
  - RandomSearch (to compare to): `BBO_random_search()`

The recommended optimizer is `BBO_adaptive_de_rand_1_bin_radiuslimited()`

The currently available algorithms are listed [here](https://github.com/robertfeldt/BlackBoxOptim.jl#state-of-the-library)

## Example

The Rosenbrock function can be optimized using the `BBO_adaptive_de_rand_1_bin_radiuslimited()` as follows:

```@example BBO
using Optimization, OptimizationBBO
rosenbrock(x, p) = (p[1] - x[1])^2 + p[2] * (x[2] - x[1]^2)^2
x0 = zeros(2)
p = [1.0, 100.0]
f = OptimizationFunction(rosenbrock)
prob = Optimization.OptimizationProblem(f, x0, p, lb = [-1.0, -1.0], ub = [1.0, 1.0])
sol = solve(prob, BBO_adaptive_de_rand_1_bin_radiuslimited(), maxiters = 100000,
    maxtime = 1000.0)
```

## Multi-objective optimization
The optimizer for Multi-Objective Optimization is `BBO_borg_moea()`. Your objective function should return a vector of the objective values and you should indicate the fitness scheme to be (typically) Pareto fitness and specify the number of objectives. Otherwise, the use is similar, here is an example:

```@example MOO-BBO
using OptimizationBBO, Optimization, BlackBoxOptim
using SciMLBase: MultiObjectiveOptimizationFunction
u0 = [0.25, 0.25]
opt = OptimizationBBO.BBO_borg_moea()
function multi_obj_func(x, p)
    f1 = (1.0 - x[1])^2 + 100.0 * (x[2] - x[1]^2)^2  # Rosenbrock function
    f2 = -20.0 * exp(-0.2 * sqrt(0.5 * (x[1]^2 + x[2]^2))) - exp(0.5 * (cos(2π * x[1]) + cos(2π * x[2]))) + exp(1) + 20.0  # Ackley function
    return (f1, f2)
end
mof = MultiObjectiveOptimizationFunction(multi_obj_func)
prob = Optimization.OptimizationProblem(mof, u0; lb = [0.0, 0.0], ub = [2.0, 2.0])
sol = solve(prob, opt, NumDimensions=2, FitnessScheme=ParetoFitnessScheme{2}(is_minimizing=true))
```
