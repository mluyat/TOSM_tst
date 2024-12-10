
# How to combine ML and ecodesign to find the best structure/material for sustainable aviation?

The aerospace industry is rapidly evolving to reduce its environmental impact by focusing on lightweight structures. Topology optimization and material selection are key design variables for achieving reductions in both mass and environmental footprint, resulting in a highly coupled optimization process.

Currently, there is a lack of a comprehensive benchmark database that considers both the mechanical performance and the environmental impact of aerospace structures.

With this in mind, this project aims to develop a database capturing the environmental impact (energy consumption, CO2 footprint, and water usage) during the production of various strategic materials used in the aerospace domain. The ultimate goal is to predict and quantify the optimal balance between mechanical performance and environmental sustainability and quantify the environmental impact.

## SimJEB

The Simulated Jet Engine Bracket Dataset (SimJEB)[1] is a publicly available shape collection of 381 brackets designed for the same engineering task. These brackets were designed for the GE Jet Engine Bracket Challenge held in 2013. Each model in the dataset has the same four bolts and interface point ensuring compatibility for the same task. 

The goal of the challenge was to design the lightest possible bracket while ensuring that the maximum stress remained below the yield stress of Ti-6Al-4V across 4 load cases: vertical, diagonal, horizontal and torsional. 

## Methodology 

SimJEB database only provides the mechanical behaviour of the brackets for ti-6Al-4V. This project, however, aims on the one hand to compute the environmental impact of these brackets and on the other incorporate additional materials used in the aerospace domain






SimJEB only offers the results for Ti-6Al-4V. However, this project aimed at increasing the number of materials used. Using [Yepes Llorente and Morlier](https://github.com/mid2SUPAERO/HybML-EvoMatDesEco/) material database material properties and the environmantal impact of the structures can be obtained. The materials chosen are BeCu-C17000, Al2024, Ti-6Al-4V and AISI 304. All environmental properties are given by unit of mass, so the total footptrint is given by the total mass of the structure times the energy or water usage. For the CO2, however not only the CO2 during production but also the emissions during the life of the aircraft. Duriez and Morlier gives an approximate value of this last term given by the equation below.


```math
CO_{2,T} = CO_{2,prod} +CO_{2,life} = CO_{2,prod} +98.8 \frac{ TC O2}{kg_{struct}} \cdot M
```

The next step is to do a FEA to obtain the maximum stresses and displacements. SimJEB offers .fem files which can be read and used by Optistruct software from Altair. Using a Python code the .fem is modified to introduce new materials and by calling the execute the Python code is able to run the simulations for each material and structure. The results are then kept in an Excel file. 

The aim of the project is to find the structures with the lowest environmental impact. However, the fact that we are in a multi-objective optimization problem there is not a unique solution but a set of solutions, also know as the Pareto Front. The multi-objective optimization problem is defined below. The aim is to minimise stress, H20, CO2 and Energy and is subjected to the constraint that the maximum stress shouldn't be greater than the yield stress of the material. It is important to mention that the deformation is considered as a flexible constraint as there is no indication on the maximum displacement the structure can be subjected to.Using [Pymoo](https://pymoo.org/index.html)/) library a NSGA-2 algorithm can be used to obtain the results. 

\begin{aligned}
    & \min \sigma, H_2O, CO_2, E \\
    & \text{s.t. } \sigma -\sigma_{yield} \leq 0  \\
\end{aligned}






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
