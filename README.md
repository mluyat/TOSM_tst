
# How to combine ML and ecodesign to find the best structure/material for sustainable aviation?

The aerospace industry is rapidly evolving to reduce its environmental impact by focusing on lightweight structures. Topology optimization and material selection are key design variables for achieving reductions in both mass and environmental footprint, resulting in a highly coupled optimization process.

Currently, there is a lack of a comprehensive benchmark database that considers both the mechanical performance and the environmental impact of aerospace structures.

With this in mind, this project aims to develop a database capturing the environmental impact (energy consumption, CO2 footprint, and water usage) during the production of various strategic materials used in the aerospace domain. The ultimate goal is to predict and quantify the optimal balance between mechanical performance and environmental sustainability and quantify the environmental impact.

## SimJEB

The Simulated Jet Engine Bracket Dataset (SimJEB)[1] is a publicly available shape collection of 381 brackets designed for the same engineering task. These brackets were designed for the GE Jet Engine Bracket Challenge held in 2013. Each model in the dataset has the same four bolts and interface point ensuring compatibility for the same task. 

The goal of the challenge was to design the lightest possible bracket while ensuring that the maximum stress remained below the yield stress of Ti-6Al-4V across 4 load cases: vertical, diagonal, horizontal and torsional. 

## Methodology 

The SimJEB database only provides the mechanical behavior of brackets made from Ti-6Al-4V. However, this project aims to expand on this by, on the one hand, computing the environmental impact of these brackets and, on the other, incorporating additional materials commonly used in the aerospace industry. For this purpose, we have relied on the material dataset previously developed by [Yepes Llorente and Morlier](https://github.com/mid2SUPAERO/HybML-EvoMatDesEco/).

The materials considered include BeCu-C17000, Al2024 and AISI 304, apart from the Ti-6Al-4V. Environmental properties such as energy consumption, CO2 footprint and water usage are given per unit mass. Consequently, the total environmental footprint is computed as the product of the total mass of the structure (M) and the specific energy or water usage.

For CO2 emissions, however, the evaluation considers both the emissions during production (CO₂_prod) and the emissions generated throughout the the aircraft's lifetime (CO₂_life)  (Duriez et Morlier) .

```math
CO_{2,T} = CO_{2,prod} +CO_{2,life} = CO_{2,prod} +98.8 \frac{ TC O2}{kg_{struct}} \cdot M
```

The next step involves performing Finite Element Analysis (FEA) to determine the maximum stresses and displacements of the brackets. The SimJEB dataset provides .fem files, which are compatible with OptiStruct software from Altair.

Using a Python script, the .fem files are modified to introduce new materials into the simulations. The script then executes the simulations for each material and structure. The results, including the computed stresses and displacements, are stored in an Excel file for further analysis and comparison.

The aim of the project is to identify structures with the lowest environmental impact. However, because this is a multi-objective optimization problem, there is no single optimal solution but rather a Pareto Front—a set of solutions representing the trade-offs between conflicting objectives.

The multi-objective optimization problem is defined as follows: the aim is to minimize stress, water usage, CO2 emissions, and energy consumption, subject to the constraint that the maximum stress does not exceed the yield stress of the material. It is important to note that deformation is considered a flexible constraint, as there is no specified maximum allowable displacement for the structures.
```math
    & \min \sigma, H_2O, CO_2, E \\
    & \text{s.t. } \sigma -\sigma_{yield} \leq 0  \\
```


Using the [Pymoo](https://pymoo.org/index.html)/) library, the NSGA-II algorithm can be applied to solve this optimization problem and identify the Pareto Front solutions.






## References 

- [1] Whalen, E., Beyene, A. and Mueller, C. (2021), SimJEB: Simulated Jet Engine Bracket Dataset. Computer Graphics Forum, 40: 9-17.

- Yepes Llorente, L., Morlier, J., Sridhara, S. et al. A hybrid machine learning and evolutionary approach to material selection and design optimization for eco-friendly structures. Struct Multidisc Optim 67, 69 (2024). https://doi.org/10.1007/s00158-024-03777-z

-E. Duriez, J. Morlier, C. Azzaro-Pantel, M. Charlotte. Ecodesign with topology optimization, Procedia CIRP, Elsevier, 454-459, 2022.





The project uses the SimJEB database: https://simjeb.github.io/

OptistructAuto notebook: 
  - Inputs: directory with .fem files and an excel with materials properties (Materials.xlsx)
  - Output: Excel file with results of displacement, VM stresses and environmental impact (Results.xlsx)

Materials.xlsx:
  - Excel with the different materials that will be simulated and its properties

Results.xlsx: 
  - Results for non-optimized brackets
