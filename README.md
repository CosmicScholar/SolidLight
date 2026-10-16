# SolidLight
A simple HTML file where i test shaders. One of my many not so private coding projects.

# AresCore: Solid-Light Manifold — Technical Explanation & Real-World Physics Mapping

## Overview
**AresCore: Solid-Light Manifold** is a GPU-accelerated parallel simulation engine built on WebGPU. It manages and renders over **8.38 million independent particle nodes ($2^{23}$)** entirely on the graphics hardware. By avoiding CPU bottlenecks, the simulation performs parallel force integrations, vector field projections, and camera transformations in real time on every frame.

---

## Part 1: Mathematical & Computational Architecture

### 1. The GPU Compute & Double-Buffer Pipeline
The engine operates on a continuous feedback loop between compute shaders and render pipelines:
* **Storage Buffers (`b1` and `b2`):** Twin GPU memory buffers storing 8.38 million `vec4<f32>` records (3D position $x, y, z$ + Layer ID $w$).
* **Compute Pass (`@compute`):** A parallel grid of 256 threads per workgroup computes the physical state updates for every particle simultaneously.
* **State Synchronization (`copyBufferToBuffer`):** Once calculated in buffer `b2`, the data is instantly copied back to `b1` via a hardware-accelerated memory copy before the render pass.

### 2. Core Physics & Force Equations
The compute shader models a non-linear vector manifold driven by pseudo-magnetic cross products and damping forces:

* **Distance Field ($dist$):** Measures the Euclidean distance of a particle position $\mathbf{p} = (x, y, z)$ from the origin:
  $$dist = \Vert{}\mathbf{p}\Vert{} = \sqrt{x^2 + y^2 + z^2}$$

* **Dipole-Like Vector Field ($\mathbf{B}$):** Constructs a non-linear directional field using spatial coordinates:
  $$\mathbf{B} = \frac{1}{0.09 \cdot dist + 0.001} \begin{pmatrix} 3 \cdot x \cdot z \\ 3 \cdot y \cdot z \\ 3 \cdot z^2 - 0.09 \end{pmatrix}$$

* **Velocity & Stretch:** Particle velocity is derived from the cross product of position and field vector $\mathbf{B}$, scaled by `uncertainty`:
  $$\mathbf{vel} = (\mathbf{p} \times \mathbf{B}) \cdot \text{uncertainty}$$
  $$stretch = \Vert{}\mathbf{vel} \times \mathbf{B}\Vert{}$$

* **Sigmoidal Manifold Blending ($S$):** A smooth gating function controls how particles transition between fluid velocity and rigid manifold constraints:
  $$S = \frac{1}{1 + e^{-qc3\_scale \cdot (stretch - 1.2)}}$$
  $$\mathbf{manifoldForce} = \text{normalize}(\mathbf{p} \times (0, 1, 0)) \cdot 0.1$$
  $$\mathbf{update} = \text{mix}(\mathbf{vel}, \mathbf{manifoldForce}, S)$$

* **Strong Force Damping & Containment:** Integrates positions across delta time ($\Delta t$) with a radial attraction force toward an optimal shell ($0.3$), backed by a hard containment ceiling at $\Vert{}\mathbf{p}\Vert{} > 2.0$:
  $$\mathbf{p}_{new} = \mathbf{p} + \left( \mathbf{update} + \left( \mathbf{p} \cdot (0.3 - dist) \cdot \text{strongForce} \right) \right) \cdot \Delta t$$

---

## Part 2: Real-World Physics Parallels

While AresCore is an artistic and mathematical simulation rather than a strict engineering simulator, the forces and equations modeled map closely to several real-world physical systems:

### 1. Magnetohydrodynamics (MHD) & Plasma Confinement
* **The Sim Parallels:** The cross-product vector field $\mathbf{p} \times \mathbf{B}$ and distance-scaled denominators closely mirror **Lorentz forces** ($\mathbf{F} = q(\mathbf{E} + \mathbf{v} \times \mathbf{B})$) and magnetic dipole fields used in plasma physics.
* **Real-World Application:** This is structurally similar to how **Tokamak fusion reactors** and planetary magnetospheres use magnetic fields (like Earth's Van Allen radiation belts) to trap, bend, and confine high-velocity charged particles along specific geometric toroidal manifolds.

### 2. Fluid Dynamics & Navier-Stokes Vortex Shedding
* **The Sim Parameters:** Variables like `viscosity`, `strongForce`, and velocity cross-products simulate fluid-like shearing and tension.
* **Real-World Application:** The way particles stretch and cluster along vector boundaries mirrors **vortex dynamics** in aerodynamics and oceanography—specifically how fluids behave when flowing around obstacles or under rotational constraints (Coriolis-like effects).

### 3. Quantum Field & Topological Manifolds (`qc3_scale`)
* **The Sim Terminology:** The use of terms like `uncertainty`, `qc3_scale`, and `strongForce` references subatomic quantum mechanics.
* **Real-World Application:** While the math in the shader is classical vector calculus rather than quantum wavefunctions, the concept of a "manifold force" acting as a strict spatial boundary mimics **quantum confinement** (e.g., electrons trapped in quantum dots or energy wells) where particles are restricted to discrete geometric probability shells.
