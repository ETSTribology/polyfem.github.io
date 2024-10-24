# JSON Files Structure
--------------------

`PolyFEM_bin` expects as input a JSON file containing the setup and parameters of the problem you wish to solve. A basic example of such a file is as follows:

```json
{
    "common": "", // path to another JSON file containing default arguments on which to patch these arguments

    "geometry": [{
        "mesh": "" // mesh path (absolute or relative to JSON file)
    }],

    "time": {                         // time-dependent problem
        "tend": 1,                    // end time
        "dt": 0.1,                    // time step size
        "time_steps": 10,             // (alternativly) number of time steps
        "integrator": "ImplicitEuler" // time integration method
    },

    "contact": {
        "enabled": true // enable contact handling
    },

    "solver": {
        "linear": {
            "solver": "Eigen::PardisoLDLT"
        },
        "nonlinear": {
            "line_search": {
                "method": "backtracking"
            },
            "solver": "newton"
        }
    },

    // Material parameter
    "materials": {
        "type": "NeoHookean", // material model
        "E": 1.5, // Young's modulus
        "nu": 0.3, // Poisson ratio
        "rho": 1 // density
    },

    "output": {
        "json": "sim.json",           // output statistics
        "paraview": {                 // output geometry as paraview VTU files
            "file_name": "sim.pvd",
            "options": {
                "material": true,     // save material properties
                "body_ids": true      // save body ids
            },
            "vismesh_rel_area": 1e-05 // relative area for upsampling the solution
        }
    }
}
```

## Overview

This JSON configuration file is divided into several sections, each responsible for different aspects of the simulation setup:

1. **Common Settings**
2. **Geometry**
3. **Time Parameters**
4. **Contact Handling**
5. **Solver Configuration**
6. **Materials**
7. **Output Settings**
8. **Boundary Conditions**
9. **Initial Conditions**

Each section is detailed below with explanations, options, and examples to guide you in creating comprehensive simulation configurations.

---

## 1. Common Settings (`/common`)

- **Type:** File
- **Extensions:** `.json`
- **Purpose:** Allows patching common settings from an external JSON file. This is useful for reusing configurations across multiple simulation files.

**Example:**

```json
"common": "defaults/common_settings.json"
```

---

## 2. Geometry (`/geometry`)

- **Type:** List of Geometry Objects
- **Minimum Items:** 1
- **Purpose:** Defines the geometric entities involved in the simulation, such as meshes, planes, ground, etc.

### Geometry Object Structure

```json
{
    "mesh": "meshes/cube.obj",
    "enabled": true,
    "transformation": {
        "scale": [1, 1, 1],
        "rotation_mode": "xyz",
        "rotation": [0, 0, 0],
        "translation": [0, 0, 0]
    },
    "surface_selection": [],
    "volume_selection": [],
    "is_obstacle": false
}
```

### Fields Explained

- **`mesh`:**  
  **Type:** String  
  **Description:** Path to the mesh file (absolute or relative to the JSON file).

- **`enabled`:**  
  **Type:** Boolean  
  **Default:** `true`  
  **Description:** Determines whether the geometry is included in the simulation. Set to `false` to exclude.

- **`transformation`:**  
  **Type:** Object  
  **Description:** Applies geometric transformations to the mesh upon loading.

  - **`scale`:**  
    **Type:** List of floats  
    **Description:** Scaling factors along each axis. Can be a single float for uniform scaling or an array for non-uniform scaling.

  - **`rotation_mode`:**  
    **Type:** String  
    **Options:** `"xyz"`, `"axis_angle"`, `"quaternion"`, `"rotation_vector"`  
    **Default:** `"xyz"`  
    **Description:** Specifies the method used to interpret the `rotation` field.

  - **`rotation`:**  
    **Type:** List of floats  
    **Description:** Rotation parameters based on the `rotation_mode`.

  - **`translation`:**  
    **Type:** List of floats  
    **Description:** Translation offsets along each axis.

- **`surface_selection` & `volume_selection`:**  
  **Type:** List or Selection Object  
  **Description:** Defines subsets of the geometry for applying specific conditions or properties.

- **`is_obstacle`:**  
  **Type:** Boolean  
  **Default:** `false`  
  **Description:** If `true`, the geometry acts as a non-deforming obstacle in collision computations.

### Transformation Order

1. **Scale**
2. **Rotate**
3. **Translate**

### Selection Types

- **Volume Selection:** Assigns IDs to volume elements of the mesh.
- **Surface Selection:** Assigns IDs to surface elements of the mesh.
- **Curve and Point Selections:** (Advanced) Assign IDs to curves or points within the mesh for specialized conditions.

### Example with Transformations and Selections

```json
"geometry": [{
    "mesh": "meshes/cube.obj",
    "enabled": true,
    "transformation": {
        "scale": [2, 1, 1],
        "rotation_mode": "axis_angle",
        "rotation": [45, 0, 1, 0], // 45 degrees around Y-axis
        "translation": [1, 0, 0]
    },
    "surface_selection": [
        {
            "type": "box",
            "box": [[0, 0, 0], [1, 0.1, 1]],
            "relative": true,
            "id": 1
        }
    ],
    "volume_selection": [
        {
            "type": "sphere",
            "center": [0, 0, 0],
            "radius": 0.5,
            "relative": false,
            "id": 2
        }
    ],
    "is_obstacle": false
}]
```

---

## 3. Time Parameters (`/time`)

- **Type:** Object
- **Purpose:** Defines the temporal aspects of the simulation, including start and end times, time step size, and the integration method.

### Time Configuration Options

```json
"time": {
    "t0": 0.0,                         // Start time
    "tend": 1.0,                       // End time
    "dt": 0.01,                        // Time step size
    "time_steps": 100,                 // Number of time steps (alternative to `tend`)
    "integrator": "ImplicitEuler",     // Time integration method
    "quasistatic": false               // Ignore inertia for quasi-static simulations
}
```

### Fields Explained

- **`t0`:**  
  **Type:** Float  
  **Default:** `0.0`  
  **Description:** Start time of the simulation.

- **`tend`:**  
  **Type:** Float  
  **Description:** End time of the simulation.

- **`dt`:**  
  **Type:** Float  
  **Description:** Time step size.

- **`time_steps`:**  
  **Type:** Integer  
  **Description:** Number of time steps. Alternatively, you can specify `tend` and `dt`.

- **`integrator`:**  
  **Type:** String  
  **Options:** `"ImplicitEuler"`, `"ImplicitNewmark"`, `"BDF"`  
  **Description:** Time integration method to use.

- **`quasistatic`:**  
  **Type:** Boolean  
  **Default:** `false`  
  **Description:** If `true`, ignores inertia, useful for incremental load simulations.

### Integrator Parameters

#### Implicit Euler

```json
"time": {
    "integrator": "ImplicitEuler"
}
```

#### Implicit Newmark

```json
"time": {
    "integrator": "ImplicitNewmark",
    "gamma": 0.5, // Default
    "beta": 0.25  // Default
}
```

#### Backward Differentiation Formula (BDF)

```json
"time": {
    "integrator": "BDF",
    "steps": 2 // Order of BDF (1 to 6)
}
```

### Example Time Configuration

```json
"time": {
    "t0": 0.0,
    "tend": 2.0,
    "dt": 0.02,
    "integrator": "ImplicitNewmark",
    "gamma": 0.6,
    "beta": 0.3,
    "quasistatic": false
}
```

---

## 4. Contact Handling (`/contact`)

- **Type:** Object
- **Purpose:** Manages contact handling parameters, including collision detection and friction settings.

### Contact Configuration Options

```json
"contact": {
    "enabled": true,                     // Enable contact handling
    "dhat": 0.03,                        // Barrier activation distance
    "mu": 0.0,                           // Coefficient of friction
    "epsv": 1e-3,                        // Smoothing parameter for friction
    "friction_iterations": 1,            // Number of friction lagging iterations
    "friction_convergence_tol": 1e-2,    // Tolerance for friction convergence
    "barrier_stiffness": "adaptive"      // Barrier stiffness setting
}
```

### Fields Explained

- **`enabled`:**  
  **Type:** Boolean  
  **Default:** `false`  
  **Description:** Enables or disables contact handling.

- **`dhat`:**  
  **Type:** Float  
  **Default:** `0.001`  
  **Description:** Contact barrier activation distance. Refer to the IPC (Incremental Potential Contact) paper for details.

- **`mu`:**  
  **Type:** Float  
  **Default:** `0.0`  
  **Description:** Coefficient of friction. Setting to `0` disables friction.

- **`epsv`:**  
  **Type:** Float  
  **Default:** `0.001`  
  **Description:** Smoothing parameter for the transition between static and dynamic friction.

- **`friction_iterations`:**  
  **Type:** Integer  
  **Default:** `1`  
  **Description:** Number of friction lagging iterations. Setting to `0` disables friction; negative values indicate unlimited iterations.

- **`friction_convergence_tol`:**  
  **Type:** Float  
  **Default:** `0.01`  
  **Description:** Tolerance for friction convergence during iterations.

- **`barrier_stiffness`:**  
  **Type:** String or Float  
  **Default:** `"adaptive"`  
  **Options:** `"adaptive"`, Any positive float value  
  **Description:** Determines how the coefficient of the clamped log-barrier function for contact is updated. `"adaptive"` adjusts based on simulation progress, while a float sets a fixed stiffness value.

### Solver-Specific Contact Settings

```json
"solver": {
    "contact": {
        "broad_phase_method": "hash_grid", // Collision detection algorithm
        "ccd_tolerance": 1e-6,              // Continuous Collision Detection tolerance
        "ccd_max_iterations": 1000000       // Maximum CCD iterations
    }
}
```

### Fields Explained

- **`broad_phase_method`:**  
  **Type:** String  
  **Options:**  
    - `"hash_grid"` / `"HG"`  
    - `"brute_force"` / `"BF"`  
    - `"spatial_hash"` / `"SH"`  
    - `"bvh"` / `"BVH"`  
    - `"sweep_and_prune"` / `"SAP"`  
    - `"sweep_and_tiniest_queue"` / `"STQ"`  
  **Default:** `"hash_grid"`  
  **Description:** Broad phase collision-detection algorithm to use.

- **`ccd_tolerance`:**  
  **Type:** Float  
  **Default:** `1e-6`  
  **Description:** Tolerance for Continuous Collision Detection (CCD).

- **`ccd_max_iterations`:**  
  **Type:** Integer  
  **Default:** `1000000`  
  **Description:** Maximum number of iterations for CCD.

### Example Contact Configuration

```json
"contact": {
    "enabled": true,
    "dhat": 0.05,
    "mu": 0.2,
    "epsv": 1e-4,
    "friction_iterations": 5,
    "friction_convergence_tol": 1e-3,
    "barrier_stiffness": 1000.0
},
"solver": {
    "contact": {
        "broad_phase_method": "bvh",
        "ccd_tolerance": 1e-5,
        "ccd_max_iterations": 500000
    }
}
```

---

## 5. Solver Configuration (`/solver`)

- **Type:** Object
- **Purpose:** Configures the numerical solver, including threading, linear and nonlinear solvers, damping, and advanced settings.

### Solver Configuration Options

```json
"solver": {
    "max_threads": 8,                     // Maximum number of threads; 0 for unlimited
    "linear": {
        "solver": "Eigen::PardisoLDLT"     // Linear solver choice
    },
    "nonlinear": {
        "solver": "newton",                // Nonlinear solver choice
        "line_search": {
            "method": "backtracking"        // Line search method
        }
    },
    "augmented_lagrangian": {
        "initial_weight": 1e6,
        "scaling": 2.0,
        "max_weight": 1e8,
        "eta": 0.99,
        "nonlinear": {
            "type": "Newton",
            "tolerance": 1e-6,
            "max_iterations": 50
        }
    },
    "rayleigh_damping": [
        {
            "form": "elasticity",
            "stiffness_ratio": 0.05,
            "lagging_iterations": 2
        }
    ],
    "advanced": {
        "cache_size": 900000,
        "lump_mass_matrix": false,
        "lagged_regularization_weight": 0.0,
        "lagged_regularization_iterations": 1
    }
}
```

### Fields Explained

- **`max_threads`:**  
  **Type:** Integer  
  **Default:** `0`  
  **Description:** Maximum number of threads to use. `0` means unlimited.

- **`linear`:**  
  **Type:** Object  
  **Description:** Configuration for the linear solver.

  - **`solver`:**  
    **Type:** String  
    **Options:**  
      - `"AMGCL"`, `"Eigen::BiCGSTAB"`, `"Eigen::CholmodSupernodalLLT"`,  
      - `"Eigen::ConjugateGradient"`, `"Eigen::DGMRES"`, `"Eigen::GMRES"`,  
      - `"Eigen::LeastSquaresConjugateGradient"`, `"Eigen::MINRES"`,  
      - `"Eigen::PardisoLDLT"`, `"Eigen::PardisoLU"`,  
      - `"Eigen::SimplicialLDLT"`, `"Eigen::SparseLU"`,  
      - `"Eigen::UmfPackLU"`, `"Hypre"`, `"Pardiso"`  
    **Description:** Chooses the linear solver library and method.

- **`nonlinear`:**  
  **Type:** Object  
  **Description:** Configuration for the nonlinear solver.

  - **`solver`:**  
    **Type:** String  
    **Options:** `"newton"`, `"lbfgs"`  
    **Description:** Chooses the nonlinear solver algorithm.

  - **`line_search`:**  
    **Type:** Object  
    **Description:** Configuration for the line search method used in the nonlinear solver.

    - **`method`:**  
      **Type:** String  
      **Options:** `"backtracking"`, `"armijo"`, `"armijo_alt"`, `"more_thuente"`  
      **Description:** Method used to perform line search for step size determination.

- **`augmented_lagrangian`:**  
  **Type:** Object  
  **Description:** Parameters for the Augmented Lagrangian method used to impose constraints like Dirichlet boundary conditions.

  - **`initial_weight`:**  
    **Type:** Float  
    **Default:** `1e6`  
    **Description:** Initial weight for the Augmented Lagrangian.

  - **`scaling`:**  
    **Type:** Float  
    **Default:** `2.0`  
    **Description:** Multiplication factor to increase the weight if constraints are not met.

  - **`max_weight`:**  
    **Type:** Float  
    **Default:** `1e8`  
    **Description:** Maximum allowable weight for the Augmented Lagrangian.

  - **`eta`:**  
    **Type:** Float  
    **Default:** `0.99`  
    **Description:** Tolerance for updating the weight or Lagrangian.

  - **`nonlinear`:**  
    **Type:** Object  
    **Description:** Configuration for the nonlinear solver within the Augmented Lagrangian.

    - **`type`:**  
      **Type:** String  
      **Options:** `"Newton"`, `"LBFGS"`  
      **Description:** Nonlinear solver type.

    - **`tolerance`:**  
      **Type:** Float  
      **Default:** `1e-6`  
      **Description:** Convergence tolerance for the nonlinear solver.

    - **`max_iterations`:**  
      **Type:** Integer  
      **Default:** `50`  
      **Description:** Maximum iterations for the nonlinear solver.

- **`rayleigh_damping`:**  
  **Type:** List of Objects  
  **Description:** Applies Rayleigh damping to specified forms.

  - **Fields for Each Damping Entry:**
    - **`form`:**  
      **Type:** String  
      **Options:** `"elasticity"`, `"contact"`, `"friction"`  
      **Description:** Specifies which part of the simulation to damp.

    - **`stiffness_ratio`:**  
      **Type:** Float  
      **Description:** Ratio used to compute the stiffness for damping.

    - **`stiffness`:**  
      **Type:** Float  
      **Description:** Direct stiffness value if not using ratio.

    - **`lagging_iterations`:**  
      **Type:** Integer  
      **Default:** `1`  
      **Description:** Number of update iterations for lagging.

- **`advanced`:**  
  **Type:** Object  
  **Description:** Advanced solver settings for fine-tuning performance and accuracy.

  - **`cache_size`:**  
    **Type:** Integer  
    **Default:** `900000`  
    **Description:** Maximum number of elements when caching assembly values.

  - **`lump_mass_matrix`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** If `true`, uses a diagonal mass matrix.

  - **`lagged_regularization_weight`:**  
    **Type:** Float  
    **Default:** `0.0`  
    **Description:** Weight used to regularize singular static problems.

  - **`lagged_regularization_iterations`:**  
    **Type:** Integer  
    **Default:** `1`  
    **Description:** Number of iterations for regularizing singular static problems.

### Example Solver Configuration

```json
"solver": {
    "max_threads": 16,
    "linear": {
        "solver": "Eigen::GMRES"
    },
    "nonlinear": {
        "solver": "newton",
        "line_search": {
            "method": "armijo"
        }
    },
    "augmented_lagrangian": {
        "initial_weight": 1e5,
        "scaling": 1.5,
        "max_weight": 1e7,
        "eta": 0.95,
        "nonlinear": {
            "type": "Newton",
            "tolerance": 1e-5,
            "max_iterations": 30
        }
    },
    "rayleigh_damping": [
        {
            "form": "elasticity",
            "stiffness_ratio": 0.02,
            "lagging_iterations": 3
        }
    ],
    "advanced": {
        "cache_size": 500000,
        "lump_mass_matrix": true,
        "lagged_regularization_weight": 1e-3,
        "lagged_regularization_iterations": 2
    }
}
```

---

## 6. Materials (`/materials`)

- **Type:** List of Material Objects
- **Purpose:** Specifies material properties such as elasticity, viscosity, density, etc.

### Material Configuration Options

PolyFEM supports various material formulations, each requiring specific parameters. The supported formulations and their required parameters are as follows:

#### **Scalar Formulations**

- **`Helmholtz`**
- **`Laplacian`**
- **`Bilaplacian`**

#### **Tensor Formulations**

- **`HookeLinearElasticity`**
- **`LinearElasticity`**
- **`NeoHookean`**
- **`SaintVenant`**
- **`IncompressibleLinearElasticity`**
- **`Stokes`**
- **`NavierStokes`**
- **`OperatorSplitting`**

### General Material Object Structure

```json
{
    "type": "NeoHookean",   // Material model type
    "id": 1,                // (Optional) Volume selection ID
    "E": 210e9,             // Young's modulus
    "nu": 0.3,              // Poisson's ratio
    "rho": 7850             // Density
}
```

### Fields Explained

- **`type`:**  
  **Type:** String  
  **Options:**  
    - **Scalar Formulations:** `"Helmholtz"`, `"Laplacian"`, `"Bilaplacian"`
    - **Tensor Formulations:** `"HookeLinearElasticity"`, `"LinearElasticity"`, `"NeoHookean"`, `"SaintVenant"`, `"IncompressibleLinearElasticity"`, `"Stokes"`, `"NavierStokes"`, `"OperatorSplitting"`  
  **Description:** Specifies the material formulation to use.

- **`id`:**  
  **Type:** Integer  
  **Default:** `0`  
  **Description:** Volume selection ID to which the material is applied. Multiple IDs can be assigned via a list.

- **Material-Specific Parameters:**
  
  - **`E` (Young's Modulus):**  
    **Type:** Float  
    **Description:** Measures the stiffness of the material.

  - **`nu` (Poisson's Ratio):**  
    **Type:** Float  
    **Description:** Describes the ratio of transverse contraction strain to longitudinal extension strain.

  - **`lambda` & `mu` (Lamé Parameters):**  
    **Type:** Float  
    **Description:** Material constants used in linear elasticity.

  - **`c1`, `c2`, `c3`, `d1` (Mooney-Rivlin Parameters):**  
    **Type:** Float  
    **Description:** Constants defining the Mooney-Rivlin material model.

  - **`alphas`, `mus`, `Ds` (Ogden Parameters):**  
    **Type:** Lists of Floats  
    **Description:** Parameters defining the Ogden hyperelastic material model.

  - **`viscosity` (for Viscoplastic Materials):**  
    **Type:** Float  
    **Description:** Viscosity coefficient.

  - **`k` (Bulk Modulus):**  
    **Type:** Float  
    **Description:** Defines the material's resistance to uniform compression.

### Supported Material Types and Their Parameters

#### **1. NeoHookean**

```json
{
    "type": "NeoHookean",
    "id": 1,
    "E": 1.5,
    "nu": 0.3,
    "rho": 1
}
```

**Alternative Representation Using Lamé Parameters:**

```json
{
    "type": "NeoHookean",
    "id": 1,
    "lambda": 1e9,
    "mu": 0.5e9,
    "rho": 1
}
```

#### **2. Mooney-Rivlin**

```json
{
    "type": "MooneyRivlin",
    "id": 2,
    "c1": 0.2,
    "c2": 0.1,
    "k": 1.0,
    "rho": 1
}
```

#### **3. Ogden**

```json
{
    "type": "UnconstrainedOgden",
    "id": 3,
    "alphas": [1.5, -1.0],
    "mus": [2.0, 1.0],
    "Ds": [1.0, 0.5],
    "rho": 1
}
```

#### **4. Linear Elasticity**

```json
{
    "type": "LinearElasticity",
    "id": 4,
    "E": 200e9,
    "nu": 0.3,
    "rho": 7800
}
```

**Alternative Representation Using Lamé Parameters:**

```json
{
    "type": "LinearElasticity",
    "id": 4,
    "lambda": 1.0e9,
    "mu": 0.5e9,
    "rho": 7800
}
```

#### **5. Saint Venant**

```json
{
    "type": "SaintVenant",
    "id": 5,
    "E": 210e9,
    "nu": 0.29,
    "rho": 7850
}
```

#### **6. Stokes**

```json
{
    "type": "Stokes",
    "id": 6,
    "viscosity": 1.0,
    "rho": 1
}
```

#### **7. Navier-Stokes**

```json
{
    "type": "NavierStokes",
    "id": 7,
    "viscosity": 0.1,
    "rho": 1.2
}
```

#### **8. Operator Splitting**

```json
{
    "type": "OperatorSplitting",
    "id": 8,
    "viscosity": 0.05,
    "rho": 1.0
}
```

### Example Materials Configuration

```json
"materials": [
    {
        "type": "NeoHookean",
        "id": 1,
        "E": 210e9,
        "nu": 0.3,
        "rho": 7800
    },
    {
        "type": "MooneyRivlin",
        "id": 2,
        "c1": 0.15,
        "c2": 0.05,
        "k": 1.2,
        "rho": 8000
    },
    {
        "type": "LinearElasticity",
        "id": [3, 4],
        "lambda": 1.0e9,
        "mu": 0.4e9,
        "rho": 7850
    }
]
```

---

## 7. Output Settings (`/output`)

- **Type:** Object
- **Purpose:** Configures how and where simulation results are outputted, including formats and frequency.

### Output Configuration Options

```json
"output": {
    "directory": "results/",           // Directory for output files
    "log": "log.json",                 // Path to output log file
    "json": "sim_stats.json",          // Path for JSON output statistics
    "restart_json": "restart_sim.json",// Path for JSON output to restart the simulation
    "paraview": {                      // ParaView output settings
        "file_name": "simulation_output.pvd",
        "vismesh_rel_area": 1e-05,     // Relative area for upsampling the solution mesh
        "skip_frame": 1,               // Export every 'skip_frame'-th frame
        "high_order_mesh": true,       // Enable high-order mesh output
        "volume": true,                // Export volumetric mesh
        "surface": false,              // Export surface mesh
        "wireframe": false,            // Export wireframe of the mesh
        "points": false,               // Export Dirichlet points
        "options": {                   
            "use_hdf5": true,          // Export data in HDF5 format
            "material": true,          // Save material properties
            "body_ids": true,          // Save body IDs
            "contact_forces": true,    // Save contact forces
            "friction_forces": false,  // Save friction forces
            "velocity": true,          // Save velocities
            "acceleration": false,     // Save accelerations
            "scalar_values": true,     // Save scalar values
            "tensor_values": true,     // Save tensor values
            "discretization_order": true,// Save FE discretization order
            "nodes": true,             // Save node ordering
            "forces": false            // Save variational forces
        }
    },
    "data": {                           // Direct data output settings
        "solution": "solution.dat",     // Path to save main variable solution
        "full_mat": "full_matrix.mtx",  // Path to save system matrix without BCs
        "stiffness_mat": "stiff_matrix.mtx",// Path to save system matrix with BCs
        "stress_mat": "stress.dat",     // Path to save stress data
        "state": "state.hdf5",          // Path to save complete simulation state
        "rest_mesh": "rest_mesh.msh",   // Path to save rest mesh
        "mises": "mises.dat",           // Path to save Von Mises stress
        "nodes": "fem_nodes.dat"        // Path to save FEM nodes
    },
    "advanced": {                       // Advanced output options
        "timestep_prefix": "step_",     // Prefix for timestep-based output files
        "sol_on_grid": -1,               // Grid spacing for solution sampling (-1 to disable)
        "compute_error": true,           // Enable error computation against reference
        "sol_at_node": -1,               // Specific node ID to save solution values
        "vis_boundary_only": false,      // Save only boundary-touching elements
        "curved_mesh_size": false,       // Upsample curved edges to compute mesh size
        "save_solve_sequence_debug": false,// Save Augmented Lagrangian internal steps
        "save_ccd_debug_meshes": false,  // Save CCD debug meshes
        "save_time_sequence": true,      // Save timesteps
        "save_nl_solve_sequence": false, // Save nonlinear solve sequence for debugging
        "spectrum": false                // Export matrix spectrum (requires POLYSOLVE_WITH_SPECTRA)
    },
    "reference": {                      // Reference solutions for error computation
        "solution": [
            "sin(x) * cos(y)",            // Reference solution expressions
            "exp(z)"
        ],
        "gradient": [
            "cos(x) * cos(y)",
            "-sin(z)"
        ]
    }
}
```

### Fields Explained

- **`directory`:**  
  **Type:** String  
  **Default:** `""`  
  **Description:** Directory where all output files will be saved.

- **`log`:**  
  **Type:** String  
  **Description:** Path to the log file containing simulation progress and messages.

- **`json`:**  
  **Type:** String  
  **Description:** Path to save JSON-formatted simulation statistics, such as time steps and error metrics.

- **`restart_json`:**  
  **Type:** String  
  **Description:** Path to save JSON files that can be used to restart simulations.

- **`paraview`:**  
  **Type:** Object  
  **Description:** Settings for exporting simulation data in ParaView-compatible formats (`.vtu`, `.pvd`).

  - **`file_name`:**  
    **Type:** String  
    **Description:** Base filename for ParaView output.

  - **`vismesh_rel_area`:**  
    **Type:** Float  
    **Description:** Relative area threshold for upsampling the visualization mesh.

  - **`skip_frame`:**  
    **Type:** Integer  
    **Default:** `1`  
    **Description:** Export every `skip_frame`-th frame for time-dependent simulations.

  - **`high_order_mesh`:**  
    **Type:** Boolean  
    **Default:** `true`  
    **Description:** Enables high-order mesh output, supporting higher polynomial degrees.

  - **`volume`:**  
    **Type:** Boolean  
    **Default:** `true`  
    **Description:** Export volumetric mesh data.

  - **`surface`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** Export surface mesh data (useful for 2D polygon simulations).

  - **`wireframe`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** Export the wireframe representation of the mesh.

  - **`points`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** Export Dirichlet boundary points.

  - **`options`:**  
    **Type:** Object  
    **Description:** Additional options to customize the exported data.

    - **`use_hdf5`:**  
      **Type:** Boolean  
      **Default:** `false`  
      **Description:** If `true`, exports data in HDF5 format, compatible with ParaView versions >5.11.

    - **`material`:**  
      **Type:** Boolean  
      **Default:** `false`  
      **Description:** Save material properties sampled on mesh vertices.

    - **`body_ids`:**  
      **Type:** Boolean  
      **Default:** `false`  
      **Description:** Export volume IDs assigned to different bodies.

    - **`contact_forces`:**  
      **Type:** Boolean  
      **Default:** `false`  
      **Description:** Export contact forces applied on surfaces.

    - **`friction_forces`:**  
      **Type:** Boolean  
      **Default:** `false`  
      **Description:** Export friction forces applied on surfaces.

    - **`velocity`:**  
      **Type:** Boolean  
      **Default:** `false`  
      **Description:** Export velocity data of the simulation.

    - **`acceleration`:**  
      **Type:** Boolean  
      **Default:** `false`  
      **Description:** Export acceleration data of the simulation.

    - **`scalar_values`:**  
      **Type:** Boolean  
      **Default:** `true`  
      **Description:** Export scalar fields (e.g., temperature).

    - **`tensor_values`:**  
      **Type:** Boolean  
      **Default:** `true`  
      **Description:** Export tensor fields (e.g., stress).

    - **`discretization_order`:**  
      **Type:** Boolean  
      **Default:** `true`  
      **Description:** Export the discretization order of finite elements.

    - **`nodes`:**  
      **Type:** Boolean  
      **Default:** `true`  
      **Description:** Export node ordering information.

    - **`forces`:**  
      **Type:** Boolean  
      **Default:** `false`  
      **Description:** Export all variational forces on the finite element mesh.

- **`data`:**  
  **Type:** Object  
  **Description:** Direct paths to save various simulation data outputs.

  - **`solution`:**  
    **Type:** String  
    **Description:** Path to save the main variable solution in unrolled format.

  - **`full_mat`:**  
    **Type:** String  
    **Description:** Path to save the full system matrix without boundary conditions.

  - **`stiffness_mat`:**  
    **Type:** String  
    **Description:** Path to save the stiffness matrix with boundary conditions applied.

  - **`stress_mat`:**  
    **Type:** String  
    **Description:** Path to save stress data.

  - **`state`:**  
    **Type:** String  
    **Description:** Path to save the complete simulation state in PolyFEM HDF5 format, useful for restarting simulations.

  - **`rest_mesh`:**  
    **Type:** String  
    **Description:** Path to save the rest (undeformed) mesh in MSH format, useful for restarting simulations.

  - **`mises`:**  
    **Type:** String  
    **Description:** Path to save per-node Von Mises stress values.

  - **`nodes`:**  
    **Type:** String  
    **Description:** Path to save finite element mesh nodes.

- **`advanced`:**  
  **Type:** Object  
  **Description:** Advanced output options for more granular control over output behaviors.

  - **`timestep_prefix`:**  
    **Type:** String  
    **Default:** `"step_"`  
    **Description:** Prefix for output file names corresponding to each timestep (e.g., `step_1.vtu`).

  - **`sol_on_grid`:**  
    **Type:** Float  
    **Default:** `-1`  
    **Description:** Grid spacing for exporting the solution sampled on a regular grid. Set to `-1` to disable.

  - **`compute_error`:**  
    **Type:** Boolean  
    **Default:** `true`  
    **Description:** Enables computation of the error against a reference solution. If no reference is provided, computes solution norms.

  - **`sol_at_node`:**  
    **Type:** Integer  
    **Default:** `-1`  
    **Description:** Specific node ID at which to save solution values. Set to `-1` to disable.

  - **`vis_boundary_only`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** If `true`, saves only elements that touch the boundaries.

  - **`curved_mesh_size`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** Upsamples curved edges to compute mesh size accurately.

  - **`save_solve_sequence_debug`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** Saves internal steps of the Augmented Lagrangian method for debugging purposes.

  - **`save_ccd_debug_meshes`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** Saves collision detection debug meshes.

  - **`save_time_sequence`:**  
    **Type:** Boolean  
    **Default:** `true`  
    **Description:** Saves data for each timestep.

  - **`save_nl_solve_sequence`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** Saves data after every nonlinear iteration for debugging.

  - **`spectrum`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** Exports the spectrum of the system matrix. Requires compilation with `POLYSOLVE_WITH_SPECTRA`.

- **`reference`:**  
  **Type:** Object  
  **Description:** Defines reference solutions and gradients for error computation.

  - **`solution`:**  
    **Type:** List of Strings  
    **Description:** Mathematical expressions representing the reference solution.

  - **`gradient`:**  
    **Type:** List of Strings  
    **Description:** Mathematical expressions representing the gradient of the reference solution.

### Example Output Configuration

```json
"output": {
    "directory": "results/",
    "log": "logs/simulation_log.json",
    "json": "stats/simulation_stats.json",
    "restart_json": "restart/state_restart.json",
    "paraview": {
        "file_name": "visualization/sim_output.pvd",
        "vismesh_rel_area": 1e-04,
        "skip_frame": 2,
        "high_order_mesh": true,
        "volume": true,
        "surface": true,
        "wireframe": false,
        "points": true,
        "options": {
            "use_hdf5": true,
            "material": true,
            "body_ids": true,
            "contact_forces": true,
            "friction_forces": true,
            "velocity": true,
            "acceleration": false,
            "scalar_values": true,
            "tensor_values": true,
            "discretization_order": true,
            "nodes": true,
            "forces": true
        }
    },
    "data": {
        "solution": "outputs/solution.dat",
        "full_mat": "outputs/full_matrix.mtx",
        "stiffness_mat": "outputs/stiffness_matrix.mtx",
        "stress_mat": "outputs/stress_data.dat",
        "state": "outputs/state.hdf5",
        "rest_mesh": "outputs/rest_mesh.msh",
        "mises": "outputs/von_mises.dat",
        "nodes": "outputs/fem_nodes.dat"
    },
    "advanced": {
        "timestep_prefix": "timestep_",
        "sol_on_grid": 0.1,
        "compute_error": true,
        "sol_at_node": 42,
        "vis_boundary_only": true,
        "curved_mesh_size": true,
        "save_solve_sequence_debug": true,
        "save_ccd_debug_meshes": false,
        "save_time_sequence": true,
        "save_nl_solve_sequence": false,
        "spectrum": true
    },
    "reference": {
        "solution": [
            "sin(x) * cos(y)",
            "exp(z)"
        ],
        "gradient": [
            "cos(x) * cos(y)",
            "-sin(z)"
        ]
    }
}
```

---

## 8. Boundary Conditions (`/boundary_conditions`)

- **Type:** Object
- **Purpose:** Defines various boundary conditions applied to the simulation domain.

### Boundary Conditions Configuration Options

```json
"boundary_conditions": {
    "rhs": 0.0,                                     // Right-hand side value for PDEs
    "dirichlet_boundary": [                        // Dirichlet boundary conditions
        {
            "id": 1,
            "value": [0, 0, 0],
            "dimension": [true, true, true]
        }
    ],
    "neumann_boundary": [                          // Neumann boundary conditions
        {
            "id": 2,
            "value": [1000, 0, 0]
        }
    ],
    "normal_aligned_neumann_boundary": [            // Normal-aligned Neumann BCs
        {
            "id": 3,
            "value": [500],
            "interpolation": "linear"
        }
    ],
    "pressure_boundary": [                          // Pressure boundary conditions
        {
            "id": 4,
            "value": 101325,
            "time_reference": [0.0, 1.0, 2.0]
        }
    ],
    "pressure_cavity": [                            // Pressure cavity boundary conditions
        {
            "id": 5,
            "value": 50000
        }
    ],
    "obstacle_displacements": [                     // Obstacle displacements
        {
            "id": 6,
            "value": [0.1, 0.0, 0.0],
            "interpolation": "cubic"
        }
    ],
    "periodic_boundary": {                          // Periodic boundary conditions
        "enabled": true,
        "tolerance": 1e-5,
        "correspondence": [
            [1.0, 0.0, 0.0],
            [0.0, 1.0, 0.0]
        ],
        "linear_displacement_offset": [
            [0.0, 0.0, 0.0],
            [0.1, 0.0, 0.0]
        ],
        "fixed_macro_strain": [0, 1],
        "force_zero_mean": true
    }
}
```

### Fields Explained

- **`rhs`:**  
  **Type:** Float or List  
  **Default:** `[]`  
  **Description:** Right-hand side value(s) for the PDEs being solved. Can be a single float for scalar PDEs or a list for vector-valued PDEs.

- **`dirichlet_boundary`:**  
  **Type:** List of Objects or String  
  **Description:** Prescribed values for the primary variables on specified boundaries.

  - **Object Structure:**
    - **`id`:**  
      **Type:** Integer  
      **Description:** Boundary ID from volume or surface selections.

    - **`value`:**  
      **Type:** List of Floats  
      **Description:** Prescribed displacement or other primary variable values.

    - **`dimension`:**  
      **Type:** List of Booleans  
      **Default:** `[true, true, true]`  
      **Description:** Indicates which dimensions the Dirichlet condition applies to.

- **`neumann_boundary`:**  
  **Type:** List of Objects  
  **Description:** Prescribed fluxes or forces on specified boundaries.

  - **Object Structure:**
    - **`id`:**  
      **Type:** Integer  
      **Description:** Boundary ID.

    - **`value`:**  
      **Type:** List of Floats  
      **Description:** Prescribed flux or force values.

- **`normal_aligned_neumann_boundary`:**  
  **Type:** List of Objects  
  **Description:** Neumann boundary conditions aligned with the normal vector of surfaces.

  - **Object Structure:**
    - **`id`:**  
      **Type:** Integer  
      **Description:** Boundary ID.

    - **`value`:**  
      **Type:** List of Floats  
      **Description:** Pressure values applied.

    - **`interpolation`:**  
      **Type:** String  
      **Options:** `"linear"`, `"cubic"`  
      **Description:** Interpolation method for time-dependent values.

- **`pressure_boundary`:**  
  **Type:** List of Objects  
  **Description:** Specifies pressure values applied on boundaries, potentially varying over time.

  - **Object Structure:**
    - **`id`:**  
      **Type:** Integer  
      **Description:** Boundary ID.

    - **`value`:**  
      **Type:** Float  
      **Description:** Pressure value.

    - **`time_reference`:**  
      **Type:** List of Floats  
      **Description:** Times at which the pressure value is specified.

- **`pressure_cavity`:**  
  **Type:** List of Objects  
  **Description:** Similar to `pressure_boundary`, typically used for cavity simulations.

  - **Object Structure:**
    - **`id`:**  
      **Type:** Integer  
      **Description:** Boundary ID.

    - **`value`:**  
      **Type:** Float  
      **Description:** Pressure value.

- **`obstacle_displacements`:**  
  **Type:** List of Objects  
  **Description:** Prescribed displacements for obstacles within the simulation domain.

  - **Object Structure:**
    - **`id`:**  
      **Type:** Integer  
      **Description:** Obstacle ID.

    - **`value`:**  
      **Type:** List of Floats  
      **Description:** Displacement vector.

    - **`interpolation`:**  
      **Type:** String  
      **Options:** `"linear"`, `"cubic"`  
      **Description:** Interpolation method for time-dependent displacements.

- **`periodic_boundary`:**  
  **Type:** Object  
  **Description:** Defines periodic boundary conditions ensuring periodicity across specified boundaries.

  - **Fields:**
    - **`enabled`:**  
      **Type:** Boolean  
      **Default:** `false`  
      **Description:** Enables or disables periodic boundary conditions.

    - **`tolerance`:**  
      **Type:** Float  
      **Default:** `1e-5`  
      **Description:** Relative tolerance for determining periodic correspondence.

    - **`correspondence`:**  
      **Type:** List of Lists  
      **Description:** Defines periodic directions. Each sub-list represents a periodic direction vector.

    - **`linear_displacement_offset`:**  
      **Type:** List of Lists  
      **Description:** Defines linear displacement offsets for periodic boundaries.

    - **`fixed_macro_strain`:**  
      **Type:** List of Integers  
      **Description:** Specifies fixed macro strains along certain directions.

    - **`force_zero_mean`:**  
      **Type:** Boolean  
      **Default:** `false`  
      **Description:** If `true`, enforces a zero mean on the periodic solution to ensure uniqueness.

### Example Boundary Conditions Configuration

```json
"boundary_conditions": {
    "rhs": [100.0, 0.0, 0.0],
    "dirichlet_boundary": [
        {
            "id": 1,
            "value": [0.0, 0.0, 0.0],
            "dimension": [true, true, true]
        },
        {
            "id": 2,
            "value": [0.1, 0.0, 0.0],
            "dimension": [true, false, false]
        }
    ],
    "neumann_boundary": [
        {
            "id": 3,
            "value": [500.0, 0.0, 0.0]
        }
    ],
    "pressure_boundary": [
        {
            "id": 4,
            "value": 101325,
            "time_reference": [0.0, 1.0, 2.0]
        }
    ],
    "obstacle_displacements": [
        {
            "id": 5,
            "value": [0.05, 0.0, 0.0],
            "interpolation": "linear"
        }
    ],
    "periodic_boundary": {
        "enabled": true,
        "tolerance": 1e-4,
        "correspondence": [
            [1.0, 0.0, 0.0],
            [0.0, 1.0, 0.0]
        ],
        "linear_displacement_offset": [
            [0.0, 0.0, 0.0],
            [0.1, 0.0, 0.0]
        ],
        "fixed_macro_strain": [0, 1],
        "force_zero_mean": true
    }
}
```

---

## 9. Initial Conditions (`/initial_conditions`)

- **Type:** Object
- **Purpose:** Sets the initial state of the simulation, including initial displacement, velocity, and acceleration.

### Initial Conditions Configuration Options

```json
"initial_conditions": {
    "solution": [                                // Initial solution
        {
            "id": 1,
            "value": [0.0, 0.0, 0.0]
        },
        {
            "id": 2,
            "value": [0.1, 0.0, 0.0]
        }
    ],
    "velocity": [                                // Initial velocity
        {
            "id": 1,
            "value": [0.0, 0.0, 0.0]
        },
        {
            "id": 2,
            "value": [1.0, 0.0, 0.0]
        }
    ],
    "acceleration": [                            // Initial acceleration
        {
            "id": 1,
            "value": [0.0, 0.0, 0.0]
        },
        {
            "id": 2,
            "value": [0.0, 9.81, 0.0]
        }
    ]
}
```

### Fields Explained

- **`solution`:**  
  **Type:** List of Objects  
  **Description:** Defines the initial displacement or primary variable values.

  - **`id`:**  
    **Type:** Integer  
    **Description:** Volume or surface selection ID.

  - **`value`:**  
    **Type:** List of Floats  
    **Description:** Initial displacement vector.

- **`velocity`:**  
  **Type:** List of Objects  
  **Description:** Defines the initial velocity of the simulation.

  - **`id`:**  
    **Type:** Integer  
    **Description:** Volume or surface selection ID.

  - **`value`:**  
    **Type:** List of Floats  
    **Description:** Initial velocity vector.

- **`acceleration`:**  
  **Type:** List of Objects  
  **Description:** Defines the initial acceleration of the simulation.

  - **`id`:**  
    **Type:** Integer  
    **Description:** Volume or surface selection ID.

  - **`value`:**  
    **Type:** List of Floats  
    **Description:** Initial acceleration vector.

### Example Initial Conditions Configuration

```json
"initial_conditions": {
    "solution": [
        {
            "id": 1,
            "value": [0.0, 0.0, 0.0]
        },
        {
            "id": 2,
            "value": [0.05, 0.0, 0.0]
        }
    ],
    "velocity": [
        {
            "id": 1,
            "value": [0.0, 0.0, 0.0]
        },
        {
            "id": 2,
            "value": [1.0, 0.0, 0.0]
        }
    ],
    "acceleration": [
        {
            "id": 1,
            "value": [0.0, 0.0, 0.0]
        },
        {
            "id": 2,
            "value": [0.0, 9.81, 0.0]
        }
    ]
}
```

---

## 10. Input for Restarting Simulations (`/input`)

- **Type:** Object
- **Purpose:** Provides mechanisms to restart simulations or input external data.

### Input Configuration Options

```json
"input": {
    "data": {
        "state": "restart/state.hdf5",   // Path to HDF5 file containing simulation state
        "reorder": false                 // Whether to reorder input data
    }
}
```

### Fields Explained

- **`data`:**  
  **Type:** Object  
  **Description:** Contains input data for restarting simulations.

  - **`state`:**  
    **Type:** String  
    **Description:** Path to an HDF5 file containing the simulation state (`u`, `v`, `a`).

  - **`reorder`:**  
    **Type:** Boolean  
    **Default:** `false`  
    **Description:** If `true`, reorders input data to match the current mesh's node ordering.

### Example Input Configuration for Restarting

```json
"input": {
    "data": {
        "state": "restart/state_restart.hdf5",
        "reorder": true
    }
}
```

### Important Note

When restarting the simulation, ensure that the `"time": {"t0": <start_time>}` parameter is correctly set to the starting time of the resumed simulation. Omitting this will default the starting time to `0`.

---
