# SDPSolver
A native Julia semidefinite program (SDP) solver.
- Motivated by the eft/modular bootstrap programs, SDPSolver is a parallelized, arbitrary precision SDP solver based on the primal-dual interior-point method. 
- SDPSolver is largely inspired by SDPA and SDPB, with slightly different parallelization architecture.
- The solver is still in a development stage, which is far from fully optimized and might contain bugs. Corrections and suggestions are welcome and will get serious attention : )

## Problem statement
The function
```julia
sdp(prec, c, A, C, B, b, β, Ωp, Ωd, ϵ_gap, ϵ_primal, ϵ_dual, iterMax)
```

solves the following SDP:

### (Primal)
$$
    \begin{aligned}
        \text{Minimize } \quad & c^T x \\
        \text{subject to } \quad & X^{(l)} = \sum_i x_i A_i^{(l)} - C^{(l)} \geq 0, \quad l = 1, 2, ..., L \\
        & B^T x = b 
    \end{aligned}
$$

### (Dual)
$$
    \begin{aligned}
        \text{Maximize } \quad & \sum_l tr(C^{(l)} Y^{(l)}) + b^T y \\
        \text{subject to } \quad & \sum_l tr(A_{\star}^{(l)} Y^{(l)}) + B y + c = 0 \\
        & Y^{(l)} \geq 0, \quad l = 1, 2, ..., L
    \end{aligned}
$$

### (Domain)
$$
    \begin{aligned}
        x & \in \mathbb{R}^m \\
        B & \in \mathbb{R}^{m \times n} \\
        b, y & \in \mathbb{R}^n \\
        A_i^{(l)}, C^{(l)}, X^{(l)}, Y^{(l)} & \in \mathbb{S}^{k^{(l)}} \\
    \end{aligned}
$$
    
## Interior point method
In each iteration, the program solves the following deformed KKT conditions to determine the Newton step:
- Primal feasibility

$$ X^{(l)} = \sum_i x_i A_i^{(l)} - C^{(l)} $$

- Dual feasibility

$$ \sum_l tr(A_{\star}^{(l)} Y^{(l)}) + B y + c = 0 $$

- Complementarity

$$ X^{(l)} Y^{(l)} = \mu^{(l)} I $$

Mehrotra's predictor-corrector method is used to accelerate convergence. 

After a search direction is obtained, the step size is determined by requiring the positivity of $X$ and $Y$.

## Arguments
`prec`: arithemetic precision in base-10, which is equivalent to
```julia
setprecision(prec, base = 10)
```
The default value of the global variable `T` is set to `BigFloat`, which supports arbitrary precision arithmetic. 
If accuracy is not a concern, the user can manually set `T` to other arithmetic types for improved performance, `T = Float64` for example.

`β`: factor of reduction in μ in each step

`Ωp` and `Ωd` are initial values for the matrices X and Y: $X = Ω_p I, Y = Ω_d I$

The iteration terminates if
- the duality gap, primal infeasibility, and dual infeasibility are below `ϵ_gap`, `ϵ_primal`, and `ϵ_dual`, respectively, or
- the number of iteration exceeds `iterMax`.

## Output
The function `sdp()` returns a dictionary with the following keys:
- "x": value of the variable `x`
- "X": value of the variable `X`
- "y": value of the variable `y`
- "Y": value of the variable `Y`
- "p-Obj": value of the primal objective function
- "d-Obj": value of the dual objective fucntion
- "status": reports the status of optimization. Can be either "Optimal" or "Cannot reach optimality within `iterMax` iterations."
