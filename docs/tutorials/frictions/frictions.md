# **Tutorial: Using Multiple Friction Coefficients in PolyFEM Simulations**

<div class="md-source-date"><small>Author: Antoine Boucher</small></div>

## **Introduction**

Friction is essential for realistic simulations of physical interactions. By separately defining static and kinetic friction coefficients, you can achieve more accurate and controlled behavior of objects in your simulations. This guide will help you configure multiple friction coefficients in PolyFEM using JSON files.

---

## **Table of Contents**

1. [Prerequisites](#prerequisites)
   - [Using Docker](#using-docker)
2. [Understanding Friction](#understanding-friction)
   - [Friction on Multiple Scales](#friction-on-multiple-scales)
   - [Plowing Model](#plowing-model)
   - [Static Friction (`static_friction_coefficient`)](#static-friction-coefficient)
   - [Kinetic Friction (`kinetic-friction-coefficient`)](#kinetic-friction-coefficient)
3. [Benefits of Separate Friction Coefficients](#benefits)
4. [Configuring Friction in JSON](#configuring-friction)
   - [Basic Configuration Structure](#basic-structure)
   - [Key Parameters](#key-parameters)
   - [Defining Materials](#defining-materials)
5. [Best Practices](#best-practices)
6. [Example Configurations](#examples)
   - [Basic Setup](#example-1)
   - [Pairwise Friction Settings](#example-2)
7. [Reference: Material Friction Coefficients](#reference)
8. [Conclusion](#conclusion)

---

### <a name="prerequisites"></a>Prerequisites

In this tutorial, we assume you have already installed PolyFEM on your machine. If not, please follow the instructions [here](https://github.com/polyfem/polyfem#installation). Additionally, ensure you have [Docker Desktop](https://www.docker.com/products/docker-desktop) installed if you plan to use Docker for running PolyFEM.

---

The first step is to clone and build PolyFEM. Follow the [official instructions](https://github.com/polyfem/polyfem#installation) to clone the repository and build the project. This will create the `PolyFEM_bin` executable that we will use to run the examples.

- [ ] [Build PolyFEM](../../../cxx_index)
- [ ] [Build fTetwild](https://github.com/wildmeshing/fTetWild)

To run an example, use the following command:

The instructions for building PolyFEM can be found [here](../../../cxx_index). After compilation, to use PolyFEM in any working directory machine, we suggest adding an alias
```shell
alias polyfem='path/to/PolyFEM/build/PolyFEM_bin'
```
to your `~/.bashrc` or `~/.zshrc` file.

Then you can use it by

```shell
polyfem --help
```

### <a name="using-docker"></a>Using Docker

Alternatively, you can use the pre-built Docker image for PolyFEM. This allows you to run PolyFEM without manually compiling it on your system.

1. **Pull the Docker Image**

   Use the following command to pull the PolyFEM Docker image:

   ```shell
   docker pull antoinebou12/polyfem
   ```

2. **Use**

   To easily run PolyFEM from any directory, set up:

   ```shell
   docker run --rm -v $(pwd):/data antoinebou12/polyfem PolyFEM_bin
   ```

   **Explanation:**
   
   - `docker run --rm`: Runs the Docker container and removes it after execution.
   - `-v $(pwd):/data`: Mounts the current working directory (`$(pwd)`) to the `/data` directory inside the container. This allows PolyFEM to access your local files.
   - `antoinebou12/polyfem`: Specifies the Docker image to use.
   - `PolyFEM_bin`: The executable inside the Docker container.

---

## <a name="understanding-friction"></a>Understanding Friction

Friction is a complex phenomenon that arises from interactions at various scales.

### <a name="friction-on-multiple-scales"></a>Friction on Multiple Scales

Friction operates across different scales, each contributing to the overall behavior observed in simulations:

![image](https://github.com/user-attachments/assets/6d5996a8-ec1d-4259-9b0f-71aba515919b)

1. **Macroscopic Scale**:
   - **What It Is**: The large-scale interaction between two objects, such as a block sliding on a table.
   - **Relevance**: Defined by friction coefficients like static (`µ_static`) and kinetic (`µ_kinetic`) friction, which dictate the ease or difficulty of initiating and maintaining motion.

2. **Mesoscopic Scale**:
   - **What It Is**: The intermediate scale involving surface asperities—tiny protrusions and indentations on material surfaces.
   - **Relevance**: Determines how much of the surfaces actually come into contact and how they deform under applied forces, influencing the macroscopic friction coefficients.

3. **Microscopic Scale**:
   - **What It Is**: The smallest scale, dealing with molecular or atomic interactions at contact points.
   - **Relevance**: Governs the fundamental forces that resist motion, such as Van der Waals forces, electrostatic interactions, and chemical bonding.

Understanding friction at these multiple scales allows for more accurate and realistic simulation models, capturing both the observable behavior and the underlying physical interactions.

### <a name="plowing-model"></a>Plowing Model

One way to conceptualize friction is through the **Plowing Model**, which draws an analogy between sliding objects and a plow moving through soil.

![image](https://github.com/user-attachments/assets/c914346e-8839-4432-a160-85607267e02a)

- **Concept**:
  - Imagine a plow moving through dirt. As it progresses, it deforms the soil, creating resistance.
  - Similarly, when one object slides over another, the surface asperities deform, leading to energy loss and resistance—this is friction.

- **Key Points**:
  - **Deformation**: Just as the plow distorts the soil, sliding objects deform each other's surfaces at contact points.
  - **Energy Dissipation**: The deformation process absorbs energy, contributing to the frictional force.
  - **Surface Roughness**: The effectiveness of the plowing model depends on the roughness of the surfaces, much like how a plow's design affects its efficiency in soil.

- **Mathematical Representation**:
  
  The friction coefficient due to plowing can be modeled based on the orientation of a meso-facet. Let:
  
  - \( \theta \) be the angle between the mean surface normal \( \hat{n} \) and the meso-facet normal \( \hat{m} \), i.e., \( \theta = \cos^{-1}(\hat{n} \cdot \hat{m}) \).
  
  - The friction coefficient \( \mu \) is then given by:
  
    \[
    \mu = \frac{2}{\pi} \tan{\theta} \quad \text{(Equation)}
    \]
  
  This formulation allows the plowing model to generate an isotropic limit surface based on the orientation of meso-facets. However, to capture anisotropic friction behavior, especially when the direction of sliding affects the resistance, the model can be further refined.

  - **Anisotropic Extension**:
    
    Consider the sliding direction \( \hat{d} \) in the tangent plane of the contact surface. The friction coefficient can be modified as:
    
    \[
    \mu_{\hat{d}} = \frac{\hat{m}' \cdot \hat{d}}{2\pi \theta} \quad \text{(Equation)}
    \]
    
    Here, \( \hat{m}' \) is the facet orientation \( \hat{m} \) projected onto the tangent space of the contact plane. This modification introduces anisotropy, as \( \mu \) now depends on the sliding direction.

  - **Asymmetric Behavior**:
    
    To account for scenarios where the friction should only resist motion in one direction (akin to a plow that only pushes soil in one direction), the model incorporates a max function:
    
    \[
    \mu_{\hat{d}} = \max\left(0, \frac{\hat{m}' \cdot \hat{d}}{2\pi \theta}\right) \quad \text{(Equation)}
    \]
    
    This ensures that friction is only generated when the sliding direction aligns with the plowing direction.

  - **Combining Surface Interactions**:
    
    For two colliding surfaces A and B, directional friction coefficients for both can be computed and combined:
    
    \[
    \mu_{\hat{d}} = \max\left(0, \frac{\hat{m}'_A \cdot \hat{d}}{2\pi \theta_A}\right) + \max\left(0, \frac{\hat{m}'_B \cdot \hat{d}}{2\pi \theta_B}\right) \quad \text{(Equation 336)}
    \]
    
    Where \( \theta_A, \hat{m}'_A \) and \( \theta_B, \hat{m}'_B \) are the corresponding angles and facet orientations from surfaces A and B, respectively. This version of the model is both anisotropic and asymmetric, capturing complex frictional behaviors based on facet orientations and sliding directions.

### <a name="static-friction-coefficient"></a>Static Friction (`static_friction_coefficient`)

- **Definition**: Friction between two objects at rest relative to each other.
- **Role**: Determines the maximum force needed to start moving an object.
- **Impact**: Higher values require more force to initiate movement.

### <a name="kinetic-friction-coefficient"></a>Kinetic Friction (`kinetic-friction-coefficient`)

- **Definition**: Friction between two objects in motion relative to each other.
- **Role**: Dictates the ongoing frictional force during movement.
- **Impact**: Typically lower than static friction, making it easier to maintain motion once started.

---

## <a name="benefits"></a>Benefits of Separate Friction Coefficients

1. **Realism**: Differentiates the force required to start movement from that needed to sustain it.
2. **Control**: Allows precise tuning for different material interactions.
3. **Flexibility**: Enables detailed modeling of complex scenarios involving multiple materials.

---

## <a name="configuring-friction"></a>Configuring Friction in JSON

### <a name="basic-structure"></a>Basic Configuration Structure

Both `static_friction_coefficient` and `kinetic_friction_coefficient` are defined within the `contact` section of your JSON configuration. They can replace or complement the traditional `friction_coefficient` (`mu`) for enhanced friction modeling.

**Example Structure:**

```json
"contact": {
    "enabled": true,
    "dhat": 0.01,
    "static_friction_coefficient": {
        "global": 0.5
    },
    "kinetic_friction_coefficient": {
        "global": 0.3
    },
    "epsv": 1e-3,
    "friction_iterations": 1,
    "friction_convergence_tol": 1e-2,
    "use_convergent_formulation": false
}
```

### <a name="key-parameters"></a>Key Parameters

#### **Static Friction Coefficient (`static_friction_coefficient`)**

- **`global`**: Universal static friction value for all material pairs.
- **`pairwise`**: Specific static friction values for defined material pairs.
  - **`ids`**: Identifiers for the material pair (use `"all"` to apply to one material and all others).
  - **`value`**: Static friction coefficient for the pair.

#### **Kinetic Friction Coefficient (`kinetic-friction-coefficient`)**

- **`global`**: Universal kinetic friction value for all material pairs.
- **`pairwise`**: Specific kinetic friction values for defined material pairs.
  - **`ids`**: Identifiers for the material pair.
  - **`value`**: Kinetic friction coefficient for the pair.

#### **Deprecated Parameter: Friction Coefficient (`mu`)**

- **Description**: Previously used for friction. Now deprecated in favor of separate static and kinetic coefficients. If both are defined, `friction_coefficient` is ignored.
- **Usage**: While it's recommended to use `static_friction_coefficient` and `kinetic_friction_coefficient`, the `mu` parameter remains for backward compatibility. However, relying on `friction_coefficient` alone limits the ability to model different behaviors for starting and sustaining motion.

### <a name="defining-materials"></a>Defining Materials

To effectively use multiple friction coefficients, you must define the materials involved in your simulation. Each material is assigned a unique `id` which corresponds to the identifiers used in the friction coefficient definitions.

**Material Definition Structure:**

```json
"materials": [
    {
        "id": 1, // Unique identifier for the material
        "type": "NeoHookean", // Material model
        "E": 1.5e6, // Young's modulus in Pascals
        "nu": 0.3, // Poisson's ratio
        "rho": 1 // Density in kg/m³
    },
]
```

**Key Points:**

- **`id`**: Each material must have a unique identifier. This `id` is used in the `static_friction_coefficient.pairwise` and `kinetic_friction_coefficient.pairwise` sections to specify friction values between material pairs.
  
  For example, in the `pairwise` configuration:
  
  ```json
  {
      "ids": [1, 2],
      "value": 0.6
  }
  ```
  
  This defines the friction coefficient between Material 1 and Material 2.

- **Material Properties**:
  - **`type`**: Specifies the material model (e.g., `NeoHookean`, `LinearElastic`, etc.).
  - **`E`**: Young's modulus, defining the material's stiffness (in Pascals).
  - **`nu`**: Poisson's ratio, describing the material's deformation characteristics.
  - **`rho`**: Density of the material (in kg/m³).

**Complete Configuration Example Including Materials:**

```json
{
    "materials": [
        {
            "id": 1,
            "type": "NeoHookean",
            "E": 1.5e6,
            "nu": 0.3,
            "rho": 1
        },
        {
            "id": 2,
            "type": "NeoHookean",
            "E": 2.0e6,
            "nu": 0.25,
            "rho": 1.2
        },
        {
            "id": 3,
            "type": "NeoHookean",
            "E": 1.8e6,
            "nu": 0.28,
            "rho": 1.1
        }
    ],
    "contact": {
        "enabled": true,
        "dhat": 0.01,
        "static_friction_coefficient": {
            "global": 0.5,
            "pairwise": [
                {
                    "ids": [1, 2],
                    "value": 0.6
                },
                {
                    "ids": ["all", 3],
                    "value": 0.7
                }
            ]
        },
        "kinetic_friction_coefficient": {
            "global": 0.3,
            "pairwise": [
                {
                    "ids": [1, 2],
                    "value": 0.4
                },
                {
                    "ids": ["all", 3],
                    "value": 0.5
                }
            ]
        },
        "epsv": 1e-3,
        "friction_iterations": 1,
        "friction_convergence_tol": 1e-2,
        "use_convergent_formulation": false
    }
}
```

**Explanation:**

- **Materials Section**: Defines three materials with unique `id`s (1, 2, 3) and their respective properties.
- **Contact Section**: Utilizes the material `id`s in the `pairwise` friction coefficient definitions to specify different friction values for each material pair.

---

## <a name="best-practices"></a>Best Practices

1. **Hierarchy Matters**: Pairwise settings override global friction coefficients.
2. **Ensure Realism**: Set `static_friction_coefficient.global` ≥ `kinetic_friction_coefficient.global`.
3. **Consistent Units**: Both coefficients are dimensionless.
4. **Solver Compatibility**: Confirm that your PolyFEM version supports separate friction coefficients.
5. **Collision Mesh Configuration**: Properly set up collision meshes to enhance contact accuracy.
6. **Define Materials Clearly**: Ensure each material has a unique `id` and that these IDs are consistently used across friction definitions.
7. **Validate Configuration**: Before running simulations, validate your JSON configuration to check for consistency and correctness in material and friction definitions.

---

## <a name="examples"></a>Example Configurations

### <a name="example-1"></a>1. Basic Setup

**Configuration:**

```json
{
    "materials": [
        {
            "id": 1,
            "type": "NeoHookean",
            "E": 1.5e6,
            "nu": 0.3,
            "rho": 1
        }
    ],
    "contact": {
        "enabled": true,
        "static_friction_coefficient": {
            "global": 0.6
        },
        "kinetic_friction_coefficient": {
            "global": 0.4
        }
    }
}
```

**Explanation:**

- **Materials**: Defines a single material with `id` 1.
- **Global Static Friction**: `0.6` – More force needed to start movement.
- **Global Kinetic Friction**: `0.4` – Easier to maintain movement once started.

### <a name="example-2"></a>2. Pairwise Friction Settings

**Configuration:**

```json
{
    "materials": [
        {
            "id": 1,
            "type": "NeoHookean",
            "E": 1.5e6,
            "nu": 0.3,
            "rho": 1
        },
        {
            "id": 2,
            "type": "NeoHookean",
            "E": 2.0e6,
            "nu": 0.25,
            "rho": 1.2
        },
        {
            "id": 3,
            "type": "NeoHookean",
            "E": 1.8e6,
            "nu": 0.28,
            "rho": 1.1
        }
    ],
    "contact": {
        "enabled": true,
        "static_friction_coefficient": {
            "global": 0.0,
            "pairwise": [
                {
                    "ids": [1, 2],
                    "value": 0.5
                },
                {
                    "ids": [2, 3],
                    "value": 0.7
                }
            ]
        },
        "kinetic_friction_coefficient": {
            "global": 0.0,
            "pairwise": [
                {
                    "ids": [1, 2],
                    "value": 0.3
                },
                {
                    "ids": [2, 3],
                    "value": 0.4
                }
            ]
        }
    }
}
```

**Explanation:**

- **Materials**: Defines three materials with unique `id`s (1, 2, 3).
- **Pair (1, 2)**:
  - Static Friction: `0.5`
  - Kinetic Friction: `0.3`
- **Pair (2, 3)**:
  - Static Friction: `0.7`
  - Kinetic Friction: `0.4`
- **Global Friction**: Set to `0.0` to ensure pairwise settings are used exclusively.

---

## <a name="reference"></a>Reference: Material Friction Coefficients

Below is a comprehensive table of static friction coefficients between various material pairs. This data is sourced from Sayantan Datta's 2019 thesis and can be instrumental in selecting appropriate friction values for your simulations.

| **Material 1** | **E₁ (GPa)** | **µ_static₁** | **ν₁** | **ρ₁ (kg/m³)** | **Material 2** | **E₂ (GPa)** | **µ_static₂** | **ν₂** | **ρ₂ (kg/m³)** | **Friction Coefficient (µ)** |
|----------------|-------------|---------------|-------|-----------------|----------------|-------------|---------------|-------|-----------------|------------------------------|
| Steel          | 200         | 0.25          | 0.3   | 8000            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.78                         |
| Copper         | 133         | 0.35          | 0.3   | 8940            | Copper         | 133         | 0.35          | 0.3   | 8940            | 1.21                         |
| Aluminum       | 69          | 0.33          | 0.33  | 2700            | Aluminum       | 69          | 0.33          | 0.33  | 2700            | 1.10                         |
| Aluminum       | 69          | 0.33          | 0.33  | 2700            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.61                         |
| Brass          | 115         | 0.34          | 0.33  | 8730            | Cast Iron      | 170         | 0.26          | 0.26  | 7200            | 0.40                         |
| Cadmium        | 64          | 0.31          | 0.31  | 8650            | Cadmium        | 64          | 0.31          | 0.31  | 8650            | 0.50                         |
| Cadmium        | 64          | 0.31          | 0.31  | 8650            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.60                         |
| Cast Iron      | 170         | 0.26          | 0.26  | 7200            | Cast Iron      | 170         | 0.26          | 0.26  | 7200            | 1.10                         |
| Chromium       | 248         | 0.31          | 0.31  | 7190            | Chromium       | 248         | 0.31          | 0.31  | 7190            | 0.41                         |
| Copper         | 133         | 0.35          | 0.35  | 8940            | Cast Iron      | 170         | 0.26          | 0.26  | 7200            | 1.05                         |
| Copper         | 133         | 0.35          | 0.35  | 8940            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.53                         |
| Glass          | 60          | 0.25          | 0.25  | 2400            | Glass          | 60          | 0.25          | 0.25  | 2400            | 0.95                         |
| Glass          | 60          | 0.25          | 0.25  | 2400            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.60                         |
| Glass          | 60          | 0.25          | 0.25  | 2400            | Nickel         | 170         | 0.31          | 0.31  | 8900            | 0.78                         |
| Graphite       | 20          | 0.20          | 0.20  | 2050            | Graphite       | 20          | 0.20          | 0.20  | 2050            | 0.10                         |
| Graphite       | 20          | 0.20          | 0.20  | 2050            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.10                         |
| Nickel         | 170         | 0.31          | 0.31  | 8900            | Nickel         | 170         | 0.31          | 0.31  | 8900            | 0.80                         |
| Nickel         | 170         | 0.31          | 0.31  | 8900            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.70                         |
| Nylon          | 4           | 0.39          | 0.39  | 1130            | Nylon          | 4           | 0.39          | 0.39  | 1130            | 0.20                         |
| Plexiglass     | 3.3         | 0.37          | 0.37  | 1190            | Plexiglass     | 3.3         | 0.37          | 0.37  | 1190            | 0.80                         |
| Plexiglass     | 3.3         | 0.37          | 0.37  | 1190            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.45                         |
| Polystyrene    | 2.5         | 0.40          | 0.40  | 1040            | Polystyrene    | 2.5         | 0.40          | 0.40  | 1040            | 0.50                         |
| Polystyrene    | 2.5         | 0.40          | 0.40  | 1040            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.30                         |
| Rubber         | 0.01        | 0.47          | 0.47  | 2300            | Asphalt        | 3           | 0.35          | 0.35  | 2500            | 0.60                         |
| Rubber         | 0.01        | 0.47          | 0.47  | 2300            | Concrete       | 17          | 0.17          | 0.17  | 2400            | 0.90                         |
| Brass          | 115         | 0.34          | 0.34  | 8730            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.35                         |
| Steel          | 200         | 0.25          | 0.25  | 8000            | Cast Iron      | 170         | 0.26          | 0.26  | 7200            | 0.40                         |
| Teflon         | 0.5         | 0.47          | 0.47  | 2200            | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.04                         |
| Teflon         | 0.5         | 0.47          | 0.47  | 2200            | Teflon         | 0.5         | 0.47          | 0.47  | 2200            | 0.04                         |
| Wood           | 10          | 0.35          | 0.35  | 750             | Wood           | 10          | 0.35          | 0.35  | 750             | 0.30                         |
| Wood           | 10          | 0.35          | 0.35  | 750             | Copper         | 133         | 0.35          | 0.35  | 8940            | 0.40                         |
| Wood           | 10          | 0.35          | 0.35  | 750             | Steel          | 200         | 0.25          | 0.3   | 8000            | 0.45                         |
| Wood           | 10          | 0.35          | 0.35  | 750             | Concrete       | 17          | 0.17          | 0.17  | 2400            | 0.62                         |
| Zinc           | 82.7        | 0.25          | 0.25  | 7120            | Zinc           | 82.7        | 0.25          | 0.25  | 7120            | 0.60                         |
| Zinc           | 82.7        | 0.25          | 0.25  | 7120            | Cast Iron      | 170         | 0.26          | 0.26  | 7200            | 0.85                         |

---

## <a name="conclusion"></a>Conclusion

Separating static and kinetic friction coefficients enhances the realism and accuracy of your PolyFEM simulations. By following this simplified guide, you can effectively configure multiple friction coefficients to model complex material interactions. Use the provided examples and reference table to get started, and adjust parameters as needed to suit your specific simulation requirements.

For more advanced configurations and troubleshooting, refer to the [PolyFEM Documentation](https://polyfem.github.io/) or visit the [PolyFEM GitHub Repository](https://github.com/polyfem/polyfem).

---

## **Source**

Datta, Sayantan. *A Numerical Study of Frictional Contact*. Master's thesis, McGill University, Montreal, 2019. [Read the Thesis](https://sayan1an.github.io/pdfs/mastersThesis.pdf)

```bibtex
@thesis{SayantanMasters2019,
   abstract = {Friction is a complex phenomenon resulting from elastic and plastic deformations coupled with molecular interaction along the contact boundary. When two surfaces touch, their roughness, and normal force determines the actual area under contact, governing the process of deformation and molecular interaction. A typical macroscopic interaction may involve millions of microscopic contacts and the aggregate of these forces give rise to the phenomenon of friction. In this thesis, our goal is to simulate the phenomenon of friction assuming unlubricated contact and elastic deformation at the contact asperities. We collect data by varying many parameters that affect friction between two surfaces and build a function approximator exploiting the correlation in data. Such an approximator is a computationally inexpensive, versatile and more accurate substitute for friction coefficient tables currently in use with various physically based simulators.},
   author = {Sayantan Datta},
   city = {Montreal},
   institution = {McGill University},
   pages = {1-91},
   title = {A Numerical Study of Frictional Contact},
   url = {https://escholarship.mcgill.ca/concern/theses/z603r069h},
   year = {2019},
}
```

---

For a deeper understanding of contact and friction simulation in computer graphics, refer to:

Andrews, Sheldon; Erleben, Kenny; Ferguson, Zachary. *Contact and Friction Simulation for Computer Graphics*. In: **ACM SIGGRAPH 2022 Courses**. Association for Computing Machinery, 2022. [Read the Paper](https://siggraphcontact.github.io/assets/files/SIGGRAPH22_friction_contact_notes.pdf)

```bibtex
@inproceedings{SiggraphContact22,
  author = {Andrews, Sheldon and Erleben, Kenny and Ferguson, Zachary},
  title = {Contact and Friction Simulation for Computer Graphics},
  year = {2022},
  publisher = {Association for Computing Machinery},
  address = {New York, NY, USA},
  url = {https://siggraphcontact.github.io/assets/files/SIGGRAPH22_friction_contact_notes.pdf},
  doi = {10.1145/xxxxxx},
  booktitle = {ACM SIGGRAPH 2022 Courses},
  articleno = {2},
  numpages = {124},
  location = {Hybrid Event, Vancouver, Canada},
  series = {SIGGRAPH '22}
}
```
