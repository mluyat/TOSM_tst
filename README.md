<script type="text/javascript" async
  src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

# How to combine ML and ecodesign to find the best structure/material for sustainable aviation?

The aerospace industry is evolving to reduce its environmental impact throught lightweight structures. Topology and material selection are key design variables for reducing mass and environmental footprint, creating a coupled optimization process.
Currently, there is a lack of a comprehensive benchmark database that focuses on both mechanical performance and environmental impact of aerospace structures.

With this in mind, this project aims at developing a database on environmental impact (energy consumption, CO2 footprint and water consumption) during production of different strategic materials used in the aerospace domain and then predict the optimal mechanical and environmental performance.


## SimJEB

The Simulated Jet Engine Bracket Dataset (SimJEB)[^1] is a public shape collection consisting of 381 brackets that are used for the same engineering task. Each bracket took part in the GE Jet Engine Bracket Challenge and every single model has the same four bolts and interface point so they can use for the same engineering task. The aim was to design the lightest bracket with the maximum stress below the yiled stress of Ti-6Al-4V across 4 load cases (vertical, diagonal, horizontal and torsional). 

## Methodology 

SimJEB only offers the results for Ti-6Al-4V. However, this project aimed at increasing the number of materials used. Using [Yepes Llorente and Morlier](https://github.com/mid2SUPAERO/HybML-EvoMatDesEco/) material database material properties and the environmantal impact of the structures can be obtained. The materials chosen are BeCu-C17000, Al2024, Ti-6Al-4V and AISI 304. All environmental properties are given by unit of mass, so the total footptrint is given by the total mass of the structure times the energy or water usage. For the CO2, however not only the CO2 during production but also the emissions during the life of the aircraft. Duriez and Morlier [^2] gives an approximate value of this last term given by the equation below.
$ CO_{2,T} = CO_{2,prod} +CO_{2,life} = CO_{2,prod} +98.8 \frac{ tonnes of CO2}{kg structure} \cdot M $ 




```math
\left( \sum_{k=1}^n a_k b_k \right)^2 \leq \left( \sum_{k=1}^n a_k^2 \right) \left( \sum_{k=1}^n b_k^2 \right)
```










## References 

- Whalen, E., Beyene, A. and Mueller, C. (2021), SimJEB: Simulated Jet Engine Bracket Dataset. Computer Graphics Forum, 40: 9-17.

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
