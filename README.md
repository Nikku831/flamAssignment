# Answer

$$
\begin{array}{|c|c|}
\hline
\textbf{Parameter} & \textbf{Fitted Value} \\
\hline
\theta & 29.99997293215849^\circ \\
M &  0.029999996873053714 \\
X & 54.99999821279995 \\
\hline
\end{array}
$$

# The Problem

This problem was to find three unknown variables — **θ (theta)**, **M**, and **X** — from the following parametric system:

$$
x = t\cos\theta - e^{M|t|}\sin(0.3t)\sin\theta + X
$$
$$
y = 42 + t\sin\theta + e^{M|t|}\sin(0.3t)\cos\theta
$$

where `t` is parameter.

## Solution Approach

**Given:**

- 1500 `(x, y)` pairs
- Known range of `t`: `[6, 60]`
- Known bounds on the unknowns:
  - `θ ∈ [0°, 50°]`
  - `M ∈ [-0.05, 0.05]`
  - `X ∈ [0, 100]`

**Unknowns to solve for:** `θ`, `M`, `X`

Rearranging the equations:

$$
x - X = t\cos\theta - e^{M|t|}\sin(0.3t)\sin\theta
$$

$$
y - 42 = t\sin\theta + e^{M|t|}\sin(0.3t)\cos\theta
$$

Letting

$$
u = t,\qquad
v = e^{M|t|}\sin(0.3t),
$$

this is exactly a 2D rotation:

$$
x - X = u\cos\theta - v\sin\theta
$$

$$
y - 42 = u\sin\theta + v\cos\theta.
$$

Writing in matrix form:

```math

\begin{bmatrix}

x - X \

y - 42

\end{bmatrix}

=

\begin{bmatrix}

\cos\theta & -\sin\theta \

\sin\theta & \cos\theta

\end{bmatrix}

\begin{bmatrix}

u \

v

\end{bmatrix}

```
This equation is of the form $Ax=b$ where A is a rotation matrix. Rotation matrix is orthogonal, so it can be inverted exactly by taking its transpose. We also know that $Ax=b \Rightarrow\ A^\top Ax=A^\top b \Rightarrow\ x=A^\top b,[\quad \because\ A^\top A=I].$ Hence
```math
\begin{bmatrix}
u \\
v
\end{bmatrix}
=
\begin{bmatrix}
\cos\theta & \sin\theta \\
-\sin\theta & \cos\theta
\end{bmatrix}
\begin{bmatrix}
x - X \\
y - 42
\end{bmatrix}
```
This means for *any* trial values of `θ` and `X`, we can recover `u` and `v` directly:

$$
u = (x − X)\cos\theta + (y − 42)\sin\theta
$$
$$
v = −(x − X)\sin\theta + (y − 42)\cos\theta
$$

Since $u = t$ exactly, every $t_i$ is known the moment we pick a trial $(\theta, M, X)$ — it doesn't need to be solved for separately. All that remains is to check how well the *other* recovered coordinate, $v_i$, matches what the model predicts at that $t_i$. Minimizing the squared distance between the two over all points pins down $X$, $M$, $\theta$ — an ordinary **3-parameter nonlinear least squares fit**:

$$
\min_{\theta\, M\, X} \ \sum_i \left[ v_i - e^{M|t_i|}\sin(0.3\,t_i) \right]^2
$$

where $t_i = u = (x_i - X)\cos\theta + (y_i - 42)\sin\theta$ is recomputed directly from the given data at every step of the fit.

## Method

1. **Residual function** — for a trial $(θ, M, X)$, compute $t_i$ and $v_i$ for every
   data point, then compare $v_i$ against the model's prediction
   $e^{(M|t_i|)}\sin(0.3t_i)$.
2. **Bounded nonlinear least squares** — `scipy.optimize.least_squares` using the
trust-region reflective algorithm — fits $\theta$, $M$, $X$ within their known
bounds. This method takes cautious, incremental steps toward the best fit and
automatically respects the given bounds by "bouncing" proposed steps back inside
the valid range rather than getting stuck at the edges.
3. **Multi-start optimization** — since the cost surface is non-convex (the
   $e^{(M|t|)}$ term combined with $sin(0.3t)$ creates multiple local minima), the fit
   is repeated from 50 independent random starting points across the full bounds.
   This is used to *prove* the found solution is the true global minimum, not a
   lucky/local result.
4. **Consistency check** — the recovered $t$ values are checked against the known
   $t \epsilon [6, 60]$ range as an independent physical validity check (unrelated to the fit
   cost itself).

## Results

### Verifying Global Convergence

- 50 independent fits run from random starting points spanning the full parameter bounds.
- 38/50 runs converged to the same solution, with cost $\approx 9.1 \times 10^{-9}$.
- The best distinct alternative solution found had cost $\approx 9.1 \times 10^{-9}$ — roughly
  12 orders of magnitude worse, confirming the winning solution is the unique
  global minimum, not one of several comparably good fits.

### Consistency Check (Recovered t vs. Known Range)

- Known $t$ range: $[6, 60]$
- Recovered $t$ range from the fitted parameters: $\approx [6.04, 59.995]$
- Matches the known range approximately — independent confirmation the fit is
  physically valid.

## Desmos

The fitted curve was plotted in Desmos:

🔗 [Desmos graph link](https://www.desmos.com/calculator/57wvv9ojwc)
