# TaskBasedProduction

[![Build Status](https://github.com/lucalore98/TaskBasedProduction.jl/actions/workflows/CI.yml/badge.svg?branch=main)](https://github.com/lucalore98/TaskBasedProduction.jl/actions/workflows/CI.yml?query=branch%3Amain)

TaskBasedProduction is a Julia package that provides functions for calculating unit labor demands, marginal products of labor, production functions, assignment thresholds, elasticities of substitution, and complementarities among worker types in a task-based production model. The package includes utilities for handling incomplete gamma functions and power series representations to facilitate these calculations. This package was developed by Daniel Haanwinckel and Luca Lorenzini based on the paper *Supply, Demand, Institutions, and Firms: A Theory of Labor Market Sorting and the Wage Distribution* by Daniel Haanwinckel ([NBER Working Paper No. 31318](https://www.nber.org/papers/w31318)).

## Installation

To install TaskBasedProduction, you can clone the repository and add it to your Julia environment:

```julia
using Pkg
Pkg.add(url="https://github.com/haanwinckel/TaskBasedProduction.git")


using TaskBasedProduction 
using SpecialFunctions


θ = 1.0
κ = 0.5
z = 1.2
αVec = [0.1, 0.2, 0.3]
labor_input = [0.5; 0.04; 0.19]

initial_guess = find_initial_guess(θ, κ, z, αVec; threshold=1e-2)
q, xT, fval = prod_fun(labor_input, θ, κ, z, αVec; initial_guess=initial_guess, x_tol=1e-10)

println("Quantity Produced: ", q)
println("Task Thresholds: ", xT)
println("Approximation error: ", fval)

# Call unitInputDemand and print the output
labor_input2 = q * unitInputDemand(xT, θ, κ, z, αVec)
println("Labor Demand: ", labor_input2)
println("Error: ", labor_input2 - labor_input)

# Call margProdLabor with labor demand and print the output
MPL = margProdLabor(labor_input, θ, κ, z, αVec)
println("Marginal Products of Labor (with labor demand): ", MPL)

MPL = margProdLabor(labor_input, θ, κ, z, αVec, xT)
println("Marginal Products of Labor (with labor demand): ", MPL)

# Call elasticity_sub_comp with labor demand, MPL, xT, and parameters of the gamma function
ϵ_sub, ϵ_compl = elasticity_sub_comp(labor_input, θ, κ, z, αVec, MPL, xT)
println("Allen partial elasticity of substitution: ", ϵ_sub)
println("Hicks partial elasticity of substitution: ", ϵ_compl)

## GENERAL PARAMETERIZATION OF FUNCTIONS

# Define the density function b_g(x)
b_g(x) = (x^(κ-1) * exp(-x/θ)) / (θ^κ * gamma(κ))
e_h1(x) = exp(0.1*x)
e_h2(x) = exp(0.2*x)
e_h3(x) = exp(0.3*x)
e_h = [e_h1, e_h2, e_h3]  # Example e_h functions

initial_guess_gen = find_initial_guess_gen(z, b_g, e_h; threshold=1e-2, verbose=false)
q_gen, xT_gen, fval = prod_fun_general(labor_input, z, b_g, e_h; initial_guess=initial_guess_gen)

labor_input_general = q_gen * unitInputDemand_general(xT_gen, z, b_g, e_h)
println("Labor Demand: ", labor_input_general)
println("Is approximation close? ", isapprox(labor_input, labor_input_general, atol=1e-6))

println("Quantity Produced: ", q_gen)
println("Task Thresholds: ", xT_gen)
MPL_gen = margProdLabor_general(labor_input_general, z, b_g, e_h, xT_gen, q_gen)
ϵ_sub_gen, ϵ_compl_gen = elasticity_sub_comp_general(labor_input_general, z, b_g, e_h, MPL_gen, xT_gen)

println("General case Allen partial elasticity of substitution: ", ϵ_sub_gen)
println("General case Hicks partial elasticity of substitution: ", ϵ_compl_gen)
```
## Functions and Features
# 1) **unitInputDemand**
Calculates unit labor demands given blueprint scale `θ`, blueprint shape `κ`, productivity `z`, an array of comparative advantage values `αVec` with H elements (one for each worker type), and an array `xT` of H-1 thresholds in task space.

# Arguments
- `xT`: An array of H-1 thresholds in task space.
- `θ`: Blueprint scale parameter.
- `κ`: Blueprint shape parameter.
- `z`: Productivity parameter.
- `αVec`: An array of comparative advantage values with H elements.
- `skipParamChecks`: A boolean indicating whether to skip parameter checks (default is false).

# Returns
- An array representing the labor demand for each labor type.
```julia
    unitInputDemand(xT::AbstractArray{<:Real}, θ::Real, κ::Real, z::Real, αVec::AbstractArray{<:Real}, skipParamChecks::Bool = false) -> AbstractArray{<:Real}
 ```
# 2) **margProdLabor**
Calculates the marginal productivity of labor for each worker type given the input parameters.

# Arguments
- `labor_input`: An array of labor demand values.
- `θ`: Blueprint scale parameter.
- `κ`: Blueprint shape parameter.
- `z`: Productivity parameter.
- `αVec`: An array of comparative advantage values.
- `xT`: (optional) An array representing the precomputed task thresholds. If not provided, it will be computed within the function.
- `q`: (optional) A scalar representing the precomputed quantity produced. If not provided, it will be computed within the function.

# Returns
- An array representing the marginal productivity of labor for each worker type.
```julia 
    margProdLabor(labor_input::AbstractArray{<:Real}, θ::Real, κ::Real, z::Real, αVec::AbstractArray{<:Real}; xT=nothing) -> AbstractArray{<:Real}
```
# 3) **prod_fun**
Calculates the quantity produced (q), and task thresholds (xT) given labor inputs (l), blueprint scale θ, blueprint shape κ, productivity z, and an array of 
comparative advantage values αVec with H elements (one for each worker type).

Inputs:
- `labor_input`: Array of labor inputs of different types.
- `θ`: Blueprint scale parameter.
- `κ`: Blueprint shape parameter.
- `z`: Productivity parameter.
- `αVec`: Array of comparative advantage values with H elements.
- `initial_guess`: (optional) Initial guess for optimization. If not provided, defaults to zeros array.
- `optim_options`: (optional) Optimization options. If not provided, defaults to high tolerance values.

Returns:
- `q`: Quantity produced.
- `xT`: Array of task thresholds.
- `fval`: Final value of the objective function.

``` julia
    prod_fun(labor_input::AbstractArray{<:Real}, θ::Real, κ::Real, z::Real, αVec::AbstractArray{<:Real}; initial_guess=nothing, optim_options=nothing)
```
# 4) **elasticity_sub_comp**

Calculates the elasticity of substitution and complementarity for a given set of parameters.

# Arguments
- `labor_input`: An array of labor inputs of different types with H elements.
- `θ`: Blueprint scale parameter.
- `κ`: Blueprint shape parameter.
- `z`: Productivity parameter.
- `αVec`: An array of comparative advantage values with H elements.
- `MPL`: (optional) An array representing the marginal productivity of labor. If not provided, it will be computed within the function.
- `xT`: (optional) An array representing precomputed task thresholds. If not provided, it will be computed within the function.

# Returns
- `ϵ_h_sub`: Matrix of elasticity of substitution values for each worker type h (rows) relative to worker type h_prime (columns).
- `ϵ_h_compl`: Matrix of elasticity of complementarity values for each worker type h (rows) relative to worker type h_prime (columns).

``` julia
    elasticity_sub_comp(labor_input::AbstractArray{<:Real}, θ::Real, κ::Real, z::Real, αVec::AbstractArray{<:Real}; MPL=nothing, xT=nothing) -> (AbstractArray{<:Real}, AbstractArray{<:Real})
```
# 5) **unitInputDemand_general**

Calculates unit labor demands given an array `xT` of H-1 thresholds in task space, a productivity value `z`, 
a density function `b_g` for the task distribution, and an array `e_h` of H functions
representing the cost of each labor type as a function of task complexity.

The function first verifies that `b_g` is a valid density function. Then it computes
the labor demand for each labor type by numerically integrating the ratio `b_g(x) / (z * e_h[h](x))`
over the intervals defined by the thresholds in `xT`.

# Arguments
- `xT`: A vector of H-1 thresholds in task space.
- `z`: Productivity value.
- `b_g`: A density function for the task distribution.
- `e_h`: A vector of H functions representing the cost of each labor type as a function of task complexity.

# Returns
- A vector representing the labor demand for each labor type.


``` julia 
unitInputDemand_general(xT::Vector{Float64}, z::Real, b_g::Function, e_h::Vector{Function}) -> Vector{Float64}
```
# 6) **margProdLabor_general**

Calculates the marginal productivity of labor for each worker type given the input parameters.

# Arguments
- `labor_input`: An array of labor inputs of different types with H elements.
- `z`: A productivity scalar.
- `b_g`: A task density function.
- `e_h`: A vector of comparative advantage functions.
- `xT`: (optional) An array representing the precomputed task thresholds. If not provided, it will be computed within the function.
- `q`: (optional) A scalar representing the precomputed quantity produced. If not provided, it will be computed within the function.

# Returns
- An array representing the marginal productivity of labor for each worker type.

``` julia
margProdLabor_general(labor_input::AbstractArray{<:Real}, z::Real, b_g::Function, e_h::Vector{Function}; xT=nothing) -> AbstractArray{<:Real}
```
# 7) **prod_fun_general**

Calculates the quantity produced (q), and task thresholds (xT) given labor inputs (labor_input), productivity z, general blueprint density function (b_g), and a vector of efficiency functions (e_h), one for each labor type.

Inputs:
- `labor_input`: Array of labor inputs of different types.
- `z`: Productivity parameter.
- `b_g`: Blueprint density function.
- `e_h`: Vector of efficiency functions, one for each type.
- `initial_guess`: (optional) Initial guess for optimization. If not provided, defaults to zeros array.
- `x_tol`: (optional) Tolerance for the solution vector. Default is 1e-12.
- `f_tol`: (optional) Tolerance for the function value. Default is 1e-12.
- `g_tol`: (optional) Tolerance for the gradient. Default is 1e-12.
- `iterations`: (optional) Maximum number of iterations for the optimization. Default is 1000.
- `max_retries`: (optional) Maximum number of retries if the optimization fails. Default is 5.

Returns:
- `q`: Quantity produced.
- `xT`: Array of task thresholds.
- `fval`: Final value of the objective function.

``` julia
    prod_fun_general(labor_input::AbstractArray{<:Real}, z::Real, b_g:: Function, e_h::Vector{Function}; initial_guess=nothing, x_tol=1e-12, f_tol=1e-12, g_tol=1e-12, iterations=1000, max_retries=5)
```
# 8) **elasticity_sub_comp_general**

Calculates the elasticity of substitution and complementarity for a given set of parameters.

# Arguments
- `labor_input`: An array of labor inputs of different types with H elements.
- `z`: Productivity parameter.
- `b_g`: General task density function.
- `e_h`: Vector of comparative advantage functions.
- `MPL`: (optional) An array representing the marginal productivity of labor. If not provided, it will be computed within the function.
- `xT`: (optional) An array representing precomputed task thresholds. If not provided, it will be computed within the function.

# Returns
- `ϵ_h_sub`: Matrix of elasticity of substitution values for each worker type h (rows) relative to worker type h_prime (columns).
- `ϵ_h_compl`: Matrix of elasticity of complementarity values for each worker type h (rows) relative to worker type h_prime (columns).

``` julia
    elasticity_sub_comp_general(labor_input::AbstractArray{<:Real}, z::Real, b_g::Function, e_h::Vector{Function}; MPL=nothing, xT=nothing) -> (AbstractArray{<:Real}, AbstractArray{<:Real})
```

# 9) **find_initial_guess**

Generate an initial guess for the optimization problem in `prod_fun` such that the implied labor demand is non-trivial.

# Arguments
- `labor_input::AbstractArray{<:Real}`: The observed labor input for each task.
- `θ::Real`: The scale parameter of the gamma distribution.
- `κ::Real`: The shape parameter of the gamma distribution.
- `z::Real`: A scaling factor for the labor input.
- `αVec::AbstractArray{<:Real}`: An array of task-specific parameters.
- `threshold::Real`: The minimum acceptable labor demand for each task.

# Returns
- `initial_guess::Array{<:Real}`: A vector containing the initial guess for the optimization, including the log of the initial production quantity `q` and the initial task thresholds `xT`.



``` julia
    find_initial_guess(labor_input::AbstractArray{<:Real}, θ::Real, κ::Real, z::Real, αVec::AbstractArray{<:Real}; threshold::Real=1e-2)
```

# 10) **find_initial_guess_gen**

  
Generate an initial guess for the optimization problem using a general density function such that the implied labor demand is non-trivial.

# Arguments
- `labor_input::AbstractArray{<:Real}`: The observed labor input for each task.
- `z::Real`: A scaling factor for the labor input.
- `αVec::AbstractArray{<:Real}`: An array of task-specific parameters.
- `pdf::Function`: The general density function.
- `threshold::Real`: The minimum acceptable labor demand for each task.
- `verbose::Bool`: Optional boolean flag to enable or disable verbose output for debugging.

# Returns
- `initial_guess::Array{<:Real}`: A vector containing the initial guess for the optimization, including the log of the initial production quantity `q` and the initial task thresholds `xT`.

``` julia
  find_initial_guess_gen(labor_input::AbstractArray{<:Real}, z::Real, αVec::AbstractArray{<:Real}, pdf::Function; threshold::Real=1e-2, verbose::Bool=false)
``` 

## Contributing
Contributions are welcome! Please feel free to submit a pull request or open an issue if you have any suggestions or find any bugs.

## License
This project is licensed under the MIT License - see the LICENSE file for details.