# **Tutorial: Utilizing Multiple Friction Coefficients in Simulations**

## **Introduction**

Friction plays a pivotal role in accurately simulating real-world physical interactions. By distinguishing between static and kinetic friction coefficients, simulations can achieve higher realism and more precise control over object behaviors. This tutorial guides you through the process of configuring multiple friction coefficients in your simulation's JSON configuration, enhancing your ability to model complex material interactions.

---

## **Table of Contents**

1. [Understanding Friction Coefficients](#understanding-friction-coefficients)
   - [Static Friction Coefficient (`static_friction_coefficient`)](#static-friction-coefficient)
   - [Kinetic Friction Coefficient (`kinetic_friction_coefficient`)](#kinetic-friction-coefficient)
2. [Benefits of Using Separate Friction Coefficients](#benefits)
3. [Configuring Multiple Friction Coefficients in JSON](#configuring-json)
   - [Configuration Structure](#configuration-structure)
   - [Key Parameters Explained](#key-parameters)
4. [Best Practices and Important Considerations](#best-practices)
5. [Example Configurations](#example-configurations)
   - [Basic Setup with Separate Friction Coefficients](#example-1)
   - [Advanced Setup with Pairwise Friction Coefficients](#example-2)
   - [Combining Global and Pairwise Friction Settings](#example-3)
6. [Reference: Material Friction Coefficients](#reference)
7. [Conclusion](#conclusion)

---

## <a name="understanding-friction-coefficients"></a>Understanding Friction Coefficients

### <a name="static-friction-coefficient"></a>Static Friction Coefficient (`static_friction_coefficient`)

- **Definition**: Represents the friction between two objects that are in contact but not moving relative to each other.
- **Role**: Determines the maximum frictional force that must be overcome to initiate motion.
- **Impact**: A higher static friction coefficient means more force is required to start moving an object.

### <a name="kinetic-friction-coefficient"></a>Kinetic Friction Coefficient (`kinetic_friction_coefficient`)

- **Definition**: Represents the friction between two objects that are sliding past each other.
- **Role**: Dictates the frictional force experienced during motion.
- **Impact**: Typically lower than the static friction coefficient, allowing easier maintenance of movement once initiated.

---

## <a name="benefits"></a>Benefits of Using Separate Friction Coefficients

1. **Increased Realism**
   - Differentiates between the force needed to start movement and the force needed to sustain it.
   - Mimics real-world physics more closely, where initiating movement often requires more force than maintaining it.

2. **Enhanced Control**
   - Allows for precise tuning of frictional behavior for various material interactions.
   - Improves simulation accuracy by accommodating specific material properties.

3. **Flexibility in Simulation**
   - Enables detailed modeling of complex interactions between multiple materials.
   - Facilitates the creation of more sophisticated and nuanced simulations.

---

## <a name="configuring-json"></a>Configuring Multiple Friction Coefficients in JSON

### <a name="configuration-structure"></a>Configuration Structure

Both `static_friction_coefficient` and `kinetic_friction_coefficient` are defined within the `contact` section of your JSON configuration. They can either replace or work alongside the traditional `friction_coefficient` (`mu`) parameter to provide a more detailed friction model.

#### **Contact Configuration Example**

```json
"contact": {
    "enabled": true,
    "dhat_percentage": 0.03,
    "epsv": 1e-3,
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
}
```

### <a name="key-parameters"></a>Key Parameters Explained

#### **Static Friction Coefficient (`static_friction_coefficient`)**

- **Type**: Object
- **Components**:
  - **`global`**
    - **Type**: Float
    - **Default**: `0.0`
    - **Description**: Sets a universal static friction coefficient applicable to all material interactions unless overridden by pairwise settings.
  - **`pairwise`**
    - **Type**: Array of Objects
    - **Description**: Defines specific static friction coefficients for designated pairs of materials.
    - **Each Pair Object Contains**:
      - **`ids`**
        - **Type**: Array (Maximum 2 items)
        - **Content**: Surface IDs (Integers or `"all"`) specifying the materials involved in the pair.
        - **Usage**: Use `"all"` to apply the coefficient between a specific material and all others.
      - **`value`**
        - **Type**: Float
        - **Description**: The static friction coefficient for the specified material pair.

#### **Kinetic Friction Coefficient (`kinetic_friction_coefficient`)**

- **Type**: Object
- **Components**:
  - **`global`**
    - **Type**: Float
    - **Default**: `0.0`
    - **Description**: Sets a universal kinetic friction coefficient applicable to all material interactions unless overridden by pairwise settings.
  - **`pairwise`**
    - **Type**: Array of Objects
    - **Description**: Defines specific kinetic friction coefficients for designated pairs of materials.
    - **Each Pair Object Contains**:
      - **`ids`**
        - **Type**: Array (Maximum 2 items)
        - **Content**: Surface IDs (Integers or `"all"`) specifying the materials involved in the pair.
        - **Usage**: Use `"all"` to apply the coefficient between a specific material and all others.
      - **`value`**
        - **Type**: Float
        - **Description**: The kinetic friction coefficient for the specified material pair.

#### **Deprecated Parameter: Friction Coefficient (`mu`)**

- **Type**: Float
- **Default**: `0.0`
- **Description**: **Deprecated**. It is recommended to use `static_friction_coefficient` and `kinetic_friction_coefficient` instead. If both are defined, `mu` will be ignored.

## <a name="best-practices"></a>Best Practices and Important Considerations

1. **Friction Hierarchy**
   - **Priority Order**: Pairwise friction settings take precedence over global settings.
   - **Implication**: Specific material interactions can be finely tuned without altering global friction parameters.

2. **Physical Realism**
   - **Condition**: Ensure that `static_friction_coefficient.global` is **greater than or equal to** `kinetic_friction_coefficient.global`.
   - **Reason**: More force is generally required to initiate movement than to sustain it, aligning with real-world physics.

3. **Replacing vs. Complementing `mu`**
   - **Replacing `mu`**: Define both `static_friction_coefficient` and `kinetic_friction_coefficient` to exclude `mu`.
   - **Complementing `mu`**: Retain `mu` for general friction if necessary, but leverage separate coefficients for enhanced control.

4. **Units Consistency**
   - **Note**: Both `static_friction_coefficient` and `kinetic_friction_coefficient` are dimensionless, representing force ratios.

---

## <a name="example-configurations"></a>Example Configurations

### <a name="example-1"></a>1. Basic Setup with Separate Friction Coefficients

**Configuration:**

```json
"contact": {
    "enabled": true,
    "dhat_percentage": 0.05,
    "epsv": 1e-4,
    "static_friction_coefficient": {
        "global": 0.6
    },
    "kinetic_friction_coefficient": {
        "global": 0.4
    },
}
```

**Explanation:**

- **Global Static Friction (`static_friction_coefficient.global`)**: `0.6`
  - **Effect**: Higher force needed to initiate movement.
  
- **Global Kinetic Friction (`kinetic_friction_coefficient.global`)**: `0.4`
  - **Effect**: Easier to maintain movement once started.
  
- **Collision Mesh**: Enabled with specified parameters to enhance collision detection accuracy.

---

### <a name="example-2"></a>2. Advanced Setup with Pairwise Friction Coefficients

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
    "epsv": 1e-3,
}
```

**Explanation:**

- **Pairwise Static Friction (`static_friction_coefficient.pairwise`)**:
  - **Pair (1, 2)**: `0.5`
  - **Pair (2, 3)**: `0.7`
  
- **Pairwise Kinetic Friction (`kinetic_friction_coefficient.pairwise`)**:
  - **Pair (1, 2)**: `0.3`
  - **Pair (2, 3)**: `0.4`
  
- **Global Friction Coefficients**: Set to `0.0` to ensure pairwise settings take precedence.
  
- **Collision Mesh**: Enabled with a finer `max_edge_length` for detailed collision detection.

---

### <a name="example-3"></a>3. Combining Global and Pairwise Friction Settings

**Configuration:**

```json
"contact": {
    "enabled": true,
    "dhat_percentage": 0.04,
    "static_friction_coefficient": {
        "global": 0.7,
        "pairwise": [
            {
                "ids": [1, 2],
                "value": 0.8
            },
            {
                "ids": ["all", 3],
                "value": 0.9
            }
        ]
    },
    "kinetic_friction_coefficient": {
        "global": 0.5,
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
    "friction_iterations": 3,
    "friction_convergence_tol": 1e-2,
    "use_convergent_formulation": true,
    "collision_mesh": {
        "enabled": false
    }
},
"solver": {
    "contact": {
        "broad_phase_method": "hash_grid",
        "ccd_tolerance": 1e-6,
        "ccd_max_iterations": 1000000
    }
}
```

**Explanation:**

- **Global Static Friction (`static_friction_coefficient.global`)**: `0.7`
  - **Effect**: Applies to all material interactions unless overridden by pairwise settings.
  
- **Pairwise Static Friction**:
  - **Pair (1, 2)**: `0.8`
  - **Pair ("all", 3)**: `0.9`
  
- **Global Kinetic Friction (`kinetic_friction_coefficient.global`)**: `0.5`
  
- **Pairwise Kinetic Friction**:
  - **Pair (1, 2)**: `0.35`
  - **Pair ("all", 3)**: `0.45`
  
- **Use Convergent Formulation**: Enabled for a more accurate IPC (Incremental Potential Contact) formulation.
  
- **Collision Mesh**: Disabled, relying on default collision detection methods.

---

## <a name="reference"></a>Reference: Material Friction Coefficients

Below is a comprehensive table of static friction coefficients between various material pairs. This data is sourced from Sayantan Datta's 2019 thesis and can be instrumental in selecting appropriate friction values for your simulations.

| **Material 1** | **E₁ (GPa)** | **µ_static₁** | **ν₁** | **ρ₁ (kg/m³)** | **Material 2** | **E₂ (GPa)** | **ν₂** | **ρ₂ (kg/m³)** | **Friction Coefficient (µ)** |
|----------------|-------------|---------------|-------|-----------------|----------------|-------------|-------|-----------------|------------------------------|
| Steel          | 200         | 0.25          | 0.3   | 8000            | Steel          | 200         | 0.25  | 8000            | 0.78                         |
| Copper         | 133         | 0.35          | 0.3   | 8940            | Copper         | 133         | 0.35  | 8940            | 1.21                         |
| Aluminum       | 69          | 0.33          | 0.33  | 2700            | Aluminum       | 69          | 0.33  | 2700            | 1.10                         |
| Aluminum       | 69          | 0.33          | 0.33  | 2700            | Steel          | 200         | 0.25  | 8000            | 0.61                         |
| Brass          | 115         | 0.34          | 0.33  | 8730            | Cast Iron      | 170         | 0.26  | 7200            | 0.40                         |
| Cadmium        | 64          | 0.31          | 0.31  | 8650            | Cadmium        | 64          | 0.31  | 8650            | 0.50                         |
| Cadmium        | 64          | 0.31          | 0.31  | 8650            | Steel          | 200         | 0.25  | 8000            | 0.60                         |
| Cast Iron      | 170         | 0.26          | 0.26  | 7200            | Cast Iron      | 170         | 0.26  | 7200            | 1.10                         |
| Chromium       | 248         | 0.31          | 0.31  | 7190            | Chromium       | 248         | 0.31  | 7190            | 0.41                         |
| Copper         | 133         | 0.35          | 0.35  | 8940            | Cast Iron      | 170         | 0.26  | 7200            | 1.05                         |
| Copper         | 133         | 0.35          | 0.35  | 8940            | Copper         | 133         | 0.35  | 8940            | 1.00                         |
| Copper         | 133         | 0.35          | 0.35  | 8940            | Steel          | 200         | 0.25  | 8000            | 0.53                         |
| Glass          | 60          | 0.25          | 0.25  | 2400            | Glass          | 60          | 0.25  | 2400            | 0.95                         |
| Glass          | 60          | 0.25          | 0.25  | 2400            | Steel          | 200         | 0.25  | 8000            | 0.60                         |
| Glass          | 60          | 0.25          | 0.25  | 2400            | Nickel         | 170         | 0.31  | 8900            | 0.78                         |
| Graphite       | 20          | 0.20          | 0.20  | 2050            | Graphite       | 20          | 0.20  | 2050            | 0.10                         |
| Graphite       | 20          | 0.20          | 0.20  | 2050            | Steel          | 200         | 0.25  | 8000            | 0.10                         |
| Nickel         | 170         | 0.31          | 0.31  | 8900            | Nickel         | 170         | 0.31  | 8900            | 0.80                         |
| Nickel         | 170         | 0.31          | 0.31  | 8900            | Steel          | 200         | 0.25  | 8000            | 0.70                         |
| Nylon          | 4           | 0.39          | 0.39  | 1130            | Nylon          | 4           | 0.39  | 1130            | 0.20                         |
| Plexiglass     | 3.3         | 0.37          | 0.37  | 1190            | Plexiglass     | 3.3         | 0.37  | 1190            | 0.80                         |
| Plexiglass     | 3.3         | 0.37          | 0.37  | 1190            | Steel          | 200         | 0.25  | 8000            | 0.45                         |
| Polystyrene    | 2.5         | 0.40          | 0.40  | 1040            | Polystyrene    | 2.5         | 0.40  | 1040            | 0.50                         |
| Polystyrene    | 2.5         | 0.40          | 0.40  | 1040            | Steel          | 200         | 0.25  | 8000            | 0.30                         |
| Rubber         | 0.01        | 0.47          | 0.47  | 2300            | Asphalt        | 3           | 0.35  | 2500            | 0.60                         |
| Rubber         | 0.01        | 0.47          | 0.47  | 2300            | Concrete       | 17          | 0.17  | 2400            | 0.90                         |
| Brass          | 115         | 0.34          | 0.34  | 8730            | Steel          | 200         | 0.25  | 8000            | 0.35                         |
| Steel          | 200         | 0.25          | 0.25  | 8000            | Cast Iron      | 170         | 0.26  | 7200            | 0.40                         |
| Teflon         | 0.5         | 0.47          | 0.47  | 2200            | Steel          | 200         | 0.25  | 8000            | 0.04                         |
| Teflon         | 0.5         | 0.47          | 0.47  | 2200            | Teflon         | 0.5         | 0.47  | 2200            | 0.04                         |
| Wood           | 10          | 0.35          | 0.35  | 750             | Wood           | 10          | 0.35  | 750             | 0.30                         |
| Wood           | 10          | 0.35          | 0.35  | 750             | Copper         | 133         | 0.35  | 8940            | 0.40                         |
| Wood           | 10          | 0.35          | 0.35  | 750             | Steel          | 200         | 0.25  | 8000            | 0.45                         |
| Wood           | 10          | 0.35          | 0.35  | 750             | Concrete       | 17          | 0.17  | 2400            | 0.62                         |
| Zinc           | 82.7        | 0.25          | 0.25  | 7120            | Zinc           | 82.7        | 0.25  | 7120            | 0.60                         |
| Zinc           | 82.7        | 0.25          | 0.25  | 7120            | Cast Iron      | 170         | 0.26  | 7200            | 0.85                         |

---

## <a name="conclusion"></a>Conclusion

By leveraging separate static and kinetic friction coefficients, you can significantly enhance the realism and accuracy of your simulations. This tutorial provided a comprehensive guide to configuring multiple friction coefficients within your JSON configuration, highlighted best practices, and offered practical examples to get you started. Additionally, the reference table of material friction coefficients serves as a valuable resource for selecting appropriate values tailored to your specific simulation needs.

For further enhancements, consider exploring advanced friction models and integrating dynamic friction adjustments based on simulation parameters or environmental factors.

---

## **Source**

Datta, Sayantan. *A Numerical Study of Frictional Contact*. Master's thesis, McGill University, Montreal, 2019. [Link to Thesis](https://escholarship.mcgill.ca/concern/theses/z603r069h)

**BibTeX Entry:**

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
