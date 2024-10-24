# **Simplified Tutorial: Using Multiple Friction Coefficients in PolyFEM Simulations**

## **Introduction**

Friction is essential for realistic simulations of physical interactions. By separately defining static and kinetic friction coefficients, you can achieve more accurate and controlled behavior of objects in your simulations. This guide will help you configure multiple friction coefficients in PolyFEM using JSON files.

---

## **Table of Contents**

1. [Understanding Friction](#understanding-friction)
   - [Friction on Multiple Scales](#friction-on-multiple-scales)
   - [Plowing Model](#plowing-model)
   - [Adhesion in the Plowing Model](#adhesion-in-the-plowing-model)
   - [Static Friction (`static_friction_coefficient`)](#static-friction-coefficient)
   - [Kinetic Friction (`kinetic-friction-coefficient`)](#kinetic-friction-coefficient)
2. [Benefits of Separate Friction Coefficients](#benefits)
3. [Configuring Friction in JSON](#configuring-friction)
   - [Basic Configuration Structure](#basic-structure)
   - [Key Parameters](#key-parameters)
4. [Best Practices](#best-practices)
5. [Example Configurations](#examples)
   - [Basic Setup](#example-1)
   - [Pairwise Friction Settings](#example-2)
   - [Friction Without Static Friction](#example-3)
6. [Reference: Material Friction Coefficients](#reference)
7. [Conclusion](#conclusion)

---

## <a name="understanding-friction"></a>Understanding Friction

Friction is a complex phenomenon that arises from interactions at various scales. Understanding these interactions is crucial for accurate simulation modeling.

### <a name="friction-on-multiple-scales"></a>Friction on Multiple Scales

Friction operates across different scales, each contributing to the overall behavior observed in simulations:

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
    \mu = \frac{2}{\pi} \tan{\theta} \quad \text{(Equation 333)}
    \]
  
  This formulation allows the plowing model to generate an isotropic limit surface based on the orientation of meso-facets. However, to capture anisotropic friction behavior, especially when the direction of sliding affects the resistance, the model can be further refined.

  - **Anisotropic Extension**:
    
    Consider the sliding direction \( \hat{d} \) in the tangent plane of the contact surface. The friction coefficient can be modified as:
    
    \[
    \mu_{\hat{d}} = \frac{\hat{m}' \cdot \hat{d}}{2\pi \theta} \quad \text{(Equation 334)}
    \]
    
    Here, \( \hat{m}' \) is the facet orientation \( \hat{m} \) projected onto the tangent space of the contact plane. This modification introduces anisotropy, as \( \mu \) now depends on the sliding direction.

  - **Asymmetric Behavior**:
    
    To account for scenarios where the friction should only resist motion in one direction (akin to a plow that only pushes soil in one direction), the model incorporates a max function:
    
    \[
    \mu_{\hat{d}} = \max\left(0, \frac{\hat{m}' \cdot \hat{d}}{2\pi \theta}\right) \quad \text{(Equation 335)}
    \]
    
    This ensures that friction is only generated when the sliding direction aligns with the plowing direction.

  - **Combining Surface Interactions**:
    
    For two colliding surfaces A and B, directional friction coefficients for both can be computed and combined:
    
    \[
    \mu_{\hat{d}} = \max\left(0, \frac{\hat{m}'_A \cdot \hat{d}}{2\pi \theta_A}\right) + \max\left(0, \frac{\hat{m}'_B \cdot \hat{d}}{2\pi \theta_B}\right) \quad \text{(Equation 336)}
    \]
    
    Where \( \theta_A, \hat{m}'_A \) and \( \theta_B, \hat{m}'_B \) are the corresponding angles and facet orientations from surfaces A and B, respectively. This version of the model is both anisotropic and asymmetric, capturing complex frictional behaviors based on facet orientations and sliding directions.

- **Implications for Simulation**:
  - **Realism**: Incorporating the plowing model helps in simulating realistic frictional behavior, especially for materials with significant surface roughness.
  - **Control**: By adjusting parameters related to deformation and surface interactions, you can fine-tune the frictional response in your simulations.

The plowing model provides a more nuanced understanding of friction, bridging the gap between abstract coefficients and tangible physical interactions.

### <a name="adhesion-in-the-plowing-model"></a>Adhesion in the Plowing Model

In addition to friction, **adhesion** plays a critical role in the interaction between sliding objects, especially for materials that exhibit sticky behaviors. Incorporating adhesion into the plowing model enhances the simulation's realism by accounting for the additional resistance caused by molecular or surface bonding.

- **Conceptual Integration**:
  - **Adhesive Forces**: While the plowing model accounts for frictional resistance due to surface asperity deformation, adhesion introduces an attractive force that resists separation of contact surfaces.
  - **Sticky Interactions**: Imagine two sticky surfaces being pulled apart; adhesion resists this separation beyond the frictional forces from plowing.

- **Mathematical Representation**:
  
  Extending the plowing model to include adhesion involves modifying the friction coefficient to account for the directionality and intensity of adhesive forces. Following the **Augmented Incremental Potential Contact (IPC)** framework:

  - **Directional Adhesion**:
    
    The adhesion force can be modeled similarly to friction but incorporates the direction of sliding and the adhesion intensity. Let:
    
    - \( \beta \) represent the adhesion intensity, ranging from 0 (no adhesion) to 1 (full adhesion).
    - The modified friction coefficient with adhesion becomes:
      
      \[
      \mu_{\hat{d}} = \max\left(0, \beta_A \frac{\hat{m}'_A \cdot \hat{d}}{2\pi \theta_A}\right) + \max\left(0, \beta_B \frac{\hat{m}'_B \cdot \hat{d}}{2\pi \theta_B}\right) \quad \text{(Equation 336 with Adhesion)}
      \]
    
    Here, \( \beta_A \) and \( \beta_B \) modulate the friction based on the adhesion intensity of surfaces A and B, respectively.

  - **Adhesion Force-Distance Relationship**:
    
    Adhesion introduces a dependence of the friction coefficient on the displacement and the time evolution of adhesion intensity. For instance, the **Raous-Cangemi-Cocou (RCC)** adhesion model defines how adhesion intensity \( \beta \) evolves with displacement:
    
    \[
    \beta(t) = \min\left(1, \max\left(0, \beta_0 + \eta \cdot f(d)\right)\right)
    \]
    
    Where \( \eta \) is a viscosity parameter and \( f(d) \) represents the adhesion force as a function of displacement \( d \).

- **Implications for Simulation**:
  - **Enhanced Realism**: Incorporating adhesion allows the simulation to capture sticky interactions, such as tape peeling or adhesive bonding between materials.
  - **Parameter Control**: By adjusting adhesion parameters (\( \beta \), \( \eta \), etc.), you can simulate a wide range of adhesive behaviors, from weak stickiness to strong bonding.
  - **Complex Interactions**: Adhesion combined with the plowing model enables the simulation of intricate behaviors like debonding and direction-dependent resistance.

## **Small Mention of Advanced Models**

For those interested in more advanced friction and adhesion models, the **Augmented Incremental Potential Contact (IPC)** framework introduced by Fang et al. (2023) provides robust simulations of sticky interactions between elastoplastic solids. This method extends the traditional IPC solver by integrating adhesion models, enabling the simulation of complex behaviors such as debonding and anisotropic friction. Incorporating such advanced models can further enhance the realism of your simulations, especially in scenarios involving adhesive materials.

**Reference:**

Fang, Yu et al. "Augmented Incremental Potential Contact for Sticky Interactions." *IEEE Transactions on Visualization and Computer Graphics*, 2023.

---

### <a name="static-friction-coefficient"></a>Static Friction (`static_friction_coefficient`)

- **Definition**: Friction between two objects at rest relative to each other.
- **Role**: Determines the maximum force needed to start moving an object.
- **Impact**: Higher values require more force to initiate movement.

### <a name="kinetic-friction-coefficient"></a>Kinetic Friction (`kinetic-friction-coefficient`)

- **Definition**: Friction between two objects in motion relative to each other.
- **Role**: Dictates the ongoing frictional force during movement.
- **Impact**: Typically lower than static friction, making it easier to maintain motion.

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
    "dhat_percentage": 0.03,
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
```

### <a name="key-parameters"></a>Key Parameters

#### **Static Friction Coefficient (`static_friction_coefficient`)**

- **`global`**: Universal static friction value for all material pairs.
- **`pairwise`**: Specific static friction values for defined material pairs.
  - **`ids`**: Identifiers for the material pair (use `"all"` to apply to one material and all others).
  - **`value`**: Static friction coefficient for the pair.

#### **Kinetic Friction Coefficient (`kinetic_friction_coefficient`)**

- **`global`**: Universal kinetic friction value for all material pairs.
- **`pairwise`**: Specific kinetic friction values for defined material pairs.
  - **`ids`**: Identifiers for the material pair.
  - **`value`**: Kinetic friction coefficient for the pair.

#### **Deprecated Parameter: Friction Coefficient (`mu`)**

- **Description**: Previously used for friction. Now deprecated in favor of separate static and kinetic coefficients. If both are defined, `mu` is ignored.
- **Usage**: While it's recommended to use `static_friction_coefficient` and `kinetic_friction_coefficient`, the `mu` parameter remains for backward compatibility. However, relying on `mu` alone limits the ability to model different behaviors for starting and sustaining motion.

#### **Additional Contact Parameters**

- **`epsv`**: Smoothing parameter for friction transitions.
- **`friction_iterations`**: Number of iterations for friction calculations.
- **`friction_convergence_tol`**: Convergence tolerance for friction calculations.
- **`use_convergent_formulation`**: Determines whether to use a convergent (area-weighted) formulation for friction.

#### **Friction Coefficient Without Static Friction (`mu_kinetic`)**

In scenarios where only kinetic friction is required—such as objects that are always in motion relative to each other—you can configure friction coefficients without defining static friction. This simplifies the configuration and focuses solely on the resistance experienced during motion.

**Example Configuration:**

```json
"contact": {
    "enabled": true,
    "dhat_percentage": 0.03,
    "mu_kinetic": {
        "global": 0.4,
        "pairwise": [
            {
                "ids": [1, 2],
                "value": 0.35
            },
            {
                "ids": ["all", 3],
                "value": 0.45
            }
        ]
    },
    "epsv": 1e-3,
    "friction_iterations": 1,
    "friction_convergence_tol": 1e-2,
    "use_convergent_formulation": false
}
```

**Note**: Ensure that your simulation scenario logically aligns with the absence of static friction to maintain physical realism.

---

## <a name="best-practices"></a>Best Practices

1. **Hierarchy Matters**: Pairwise settings override global friction coefficients.
2. **Ensure Realism**: Set `static_friction_coefficient.global` ≥ `kinetic_friction_coefficient.global`.
3. **Consistent Units**: Both coefficients are dimensionless.
4. **Solver Compatibility**: Confirm that your PolyFEM version supports separate friction coefficients.
5. **Collision Mesh Configuration**: Properly set up collision meshes to enhance contact accuracy.

---

## <a name="examples"></a>Example Configurations

### <a name="example-1"></a>1. Basic Setup

**Configuration:**

```json
"contact": {
    "enabled": true,
    "dhat_percentage": 0.05,
    "static_friction_coefficient": {
        "global": 0.6
    },
    "kinetic_friction_coefficient": {
        "global": 0.4
    },
    "epsv": 1e-4,
    "friction_iterations": 5,
    "friction_convergence_tol": 1e-3,
    "use_convergent_formulation": false
}
```

**Explanation:**

- **Global Static Friction**: `0.6` – More force needed to start movement.
- **Global Kinetic Friction**: `0.4` – Easier to maintain movement once started.

### <a name="example-2"></a>2. Pairwise Friction Settings

**Configuration:**

```json
"contact": {
    "enabled": true,
    "dhat_percentage": 0.03,
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
    },
    "epsv": 1e-3
}
```

**Explanation:**

- **Pair (1, 2)**:
  - Static Friction: `0.5`
  - Kinetic Friction: `0.3`
- **Pair (2, 3)**:
  - Static Friction: `0.7`
  - Kinetic Friction: `0.4`
- **Global Friction**: Set to `0.0` to ensure pairwise settings are used.

### <a name="example-3"></a>3. Friction Without Static Friction

**Configuration:**

```json
"contact": {
    "enabled": true,
    "dhat_percentage": 0.02,
    "mu_kinetic": {
        "global": 0.35,
        "pairwise": [
            {
                "ids": [1, 3],
                "value": 0.25
            },
            {
                "ids": ["all", 4],
                "value": 0.45
            }
        ]
    },
    "epsv": 1e-3,
    "friction_iterations": 2,
    "friction_convergence_tol": 1e-4,
    "use_convergent_formulation": true
}
```

**Explanation:**

- **Global Kinetic Friction**: `0.35` – Standard resistance during motion.
- **Pair (1, 3)**:
  - Kinetic Friction: `0.25` – Reduced resistance for this specific material pair.
- **Pair ("all", 4)**:
  - Kinetic Friction: `0.45` – Increased resistance when interacting with material ID `4`.
- **No Static Friction**: Suitable for simulations where objects are perpetually in motion or static friction is negligible.

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

Datta, Sayantan. *A Numerical Study of Frictional Contact*. Master's thesis, McGill University, Montreal, 2019. [Read the Thesis](https://escholarship.mcgill.ca/concern/theses/z603r069h)


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
