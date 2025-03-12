<script>
    import Plotly from 'plotly.js-dist';
    import { onMount } from "svelte";
    import * as XLSX from "xlsx";

    let finalData = [];
    let paretoData = [];
    let xAxis = 'Mass'; // Default X-axis column
    let yAxis = 'VM';   // Default Y-axis column
    let selectedMaterials = new Set(); // Track selected materials
    let allMaterials = []; // List of all materials in the dataset
    let minDisplacement = 0;
    let maxDisplacement = 10; 
    let paretoOptimal = false;

    const columnMapping = {
        "Mass": "Mass [kg]",
        "CO2": "CO2 [t]",
        "VM": "Max VM [MPa]",
        "Energy": "Energy [MJ]",
        "Water": "Water [L]",
        "Disp": "Max Disp [mm]"
    };
    const materialLegendMapping = {
        "BeCu C17000": "BeCu C17000",
        "Al-2024": "Al-2024",
        "Ti-6Al-4V": "Ti-6Al-4V",
        "AISI 304": "AISI 304"
    };

    // Assign colors to materials
    const materialColors = {
        "BeCu C17000": "red",
        "Al-2024": "#C0C0C0",
        "Ti-6Al-4V": "#0000FF",
        "AISI 304": "#ba8e23"
    };

    async function loadExcelData(filePath, isPareto = false) {
        try {
            const response = await fetch(filePath);
            if (!response.ok) throw new Error(`Failed to load file: ${response.statusText}`);
            
            const arrayBuffer = await response.arrayBuffer();
            const workbook = XLSX.read(arrayBuffer, { type: 'array' });
            const sheetName = workbook.SheetNames[1];
            const sheet = workbook.Sheets[sheetName];
            
            // Parse the Excel sheet with headers
            const data = XLSX.utils.sheet_to_json(sheet, { defval: null });
            if (isPareto) {
                paretoData = data;
            } else {
                finalData = data;
                allMaterials = [...new Set(data.map(d => d["Material"]))];
                selectedMaterials = new Set(allMaterials); // Initially, select all materials
            }

            console.log("Excel data loaded:", data); 
            updatePlot();
        } catch (error) {
            console.error("Error loading Excel data:", error);
        }
    }

    function updatePlot() {
    console.log("Updating plot with minDisplacement:", minDisplacement, "maxDisplacement:", maxDisplacement, "paretoOptimal:", paretoOptimal);
    if (!finalData.length) return; // Ensure data is loaded before plotting
    const xColumn = columnMapping[xAxis];
    const yColumn = columnMapping[yAxis];

    // Extract data for the selected X and Y axes and filter by selected materials
    const filteredFinalData = finalData.filter(d => selectedMaterials.has(d["Material"]) &&
             d["Max Disp [mm]"] >= minDisplacement && d["Max Disp [mm]"] <= maxDisplacement
    );

    const filteredParetoData = paretoData.filter(d => selectedMaterials.has(d["Material"]) &&
             d["Max Disp [mm]"] >= minDisplacement && d["Max Disp [mm]"] <= maxDisplacement
    );

    console.log("Filtered final data:", filteredFinalData);
    console.log("Filtered pareto data:", filteredParetoData);

    const finalPlotData = allMaterials.map(material => {
        const materialData = filteredFinalData.filter(d => d["Material"] === material);
        return {
            x: materialData.map(d => parseFloat(d[xColumn]) || 0),
            y: materialData.map(d => parseFloat(d[yColumn]) || 0),
            mode: 'markers',
            type: 'scatter',
            name: materialLegendMapping[material] || material, // Use custom name or fallback to original
            marker: {
                size: 10,
                color: materialColors[material] || 'gray', // Assign the correct color per material
                opacity: paretoOptimal ? 0.2 : 1 // Set transparency to 80% if paretoOptimal is true, otherwise no transparency
            },
            text: materialData.map(d => d['Name'] || "") // Optional labels
        };
    });

    const paretoPlotData = {
        x: filteredParetoData.map(d => parseFloat(d[xColumn]) || 0),
        y: filteredParetoData.map(d => parseFloat(d[yColumn]) || 0),
        mode: 'markers',
        type: 'scatter',
        name: 'Pareto Optimal',
        marker: {
            size: 10,
            color: 'black' // Set color to black
        },
        text: filteredParetoData.map(d => d['Name'] || "") // Optional labels
    };

    const plotData = [...finalPlotData, paretoPlotData];

    const layout = {
        xaxis: {
            title: {
                text: columnMapping[xAxis],
                font: {
                    size: 25,
                    family: 'Helvetica' // Set font to Helvetica
                },
                standoff: 50,
            },
            tickfont: {
                size: 25,
                family: 'Helvetica' // Set font to Helvetica
            }
        },
        yaxis: {
            title: {
                text: columnMapping[yAxis],
                font: {
                    size: 25,
                    family: 'Helvetica' // Set font to Helvetica
                },
                standoff: 70,
            },
            tickfont: {
                size: 25,
                family: 'Helvetica' // Set font to Helvetica
            }
        },
        legend: {
            font: {
                size: 30, // Adjust the size as needed
                family: 'Helvetica' // Set font to Helvetica
            },
            x: 1,
            y: 1,
            xanchor: 'right',
            yanchor: 'top'
        },
        showlegend: false
    };

    Plotly.newPlot('plot', plotData, layout);
}

    function toggleMaterial(material) {
        if (selectedMaterials.has(material)) {
            selectedMaterials.delete(material);
        } else {
            selectedMaterials.add(material);
        }
        updatePlot();
    }

    function toggleAllMaterials() {
        if (selectedMaterials.size === allMaterials.length) {
            selectedMaterials.clear(); // Unselect all materials
        } else {
            selectedMaterials = new Set(allMaterials); // Select all materials
        }
        updatePlot();
    }

    onMount(() => {
        loadExcelData('/FinalResults.xlsx');
        loadExcelData('/ParetoFrontResults.xlsx', true);
    });
</script>

<main>
    <div class="dropdown-container">
        <h2 class= "title">Variables to plot</h2>
        <div class="dropdown-item">
            <label for="x-axis">X-axis:</label>
            <select id="x-axis" bind:value={xAxis} on:change={updatePlot}>
                <option value="Mass">Mass</option>
                <option value="Disp">Disp</option>
                <option value="VM">VM</option>
                <option value="Energy">Energy</option>
                <option value="Water">Water</option>
                <option value="CO2">CO2</option>
            </select>
        </div>
        <div class="dropdown-item">
            <label for="y-axis">Y-axis:</label>
            <select id="y-axis" bind:value={yAxis} on:change={updatePlot}>
                <option value="Mass">Mass</option>
                <option value="Disp">Disp</option>
                <option value="VM">VM</option>
                <option value="Energy">Energy</option>
                <option value="Water">Water</option>
                <option value="CO2">CO2</option>
            </select>
        </div>
    </div>

    <div class="material-selection">
        <h2 class= "title">Material Selection</h2>
        {#each allMaterials as material}
            <label class="toggle-switch">
                <input type="checkbox" checked={selectedMaterials.has(material)} on:change={() => toggleMaterial(material)}>
                <span class="slider" style="background-color: {selectedMaterials.has(material) ? materialColors[material] : 'gray'};"></span>
                {material}
            </label>
        {/each}
        <label class="toggle-switch">
            <input type="checkbox" checked={selectedMaterials.size === allMaterials.length} on:change={toggleAllMaterials}>
            <span class="slider" style="background-color: {selectedMaterials.size === allMaterials.length ? 'green' : 'red'};"></span>
            {selectedMaterials.size === allMaterials.length ? "Unselect/ Select All Materials" : "Select All Materials"}
        </label>
    </div>

    <div class="pareto-optimal-selection">
        <h2 class= "title">Pareto Optimal</h2>
        <label class="toggle-switch">
            <input type="checkbox" bind:checked={paretoOptimal} on:change={updatePlot}>
            <span class="slider" style="background-color: {paretoOptimal ? 'green' : 'red'};"></span>
        </label>
    </div>

    <div class="displacement-range">
        <label for="min-displacement">Min Displacement:</label>
        <input type="range" id="min-displacement" min="0" max="8" step="0.1" bind:value={minDisplacement} on:input={updatePlot} />
        <span>{minDisplacement}</span>

        <label for="max-displacement">Max Displacement:</label>
        <input type="range" id="max-displacement" min="0" max="10" step ="0.1" bind:value={maxDisplacement} on:input={updatePlot} />
        <span>{maxDisplacement}</span>
    </div>

    <div id="plot" style="width: 100%; height: 600px;"></div>
</main>

<style>
    main {
        font-family: Helvetica, sans-serif;
        padding: 2rem;
        font-size: 30;
    }
    .dropdown-container {
        display: flex;
        flex-direction: row; /* Align dropdowns horizontally */
        align-items: center; /* Align dropdowns vertically centered */
        margin-bottom: 1rem;
    }
    .dropdown-item {
        display: flex;
        align-items: center;
        margin-right: 1rem; /* Add space between dropdown items */
    }
    .dropdown-item label {
        margin-right: 1rem;
        font-size: 1.5;
        font-family: 'Helvetica', sans-serif; 
    }
    .material-selection {
        display: flex;
        flex-wrap: wrap;
        align-items: center;
        margin-bottom: 1rem;
        font-family: 'Hel';
    }
    .pareto-optimal-selection {
        display: flex;
        align-items: center;
        margin-bottom: 1rem;
    }
   
    .toggle-switch {
        display: flex;
        align-items: center;
        margin: 0.5rem;
    }
    .toggle-switch .slider {
        margin-right: 1rem; /* Add space between the toggle switch and the label */
    }
    .toggle-switch input {
        display: none;
    }
    .toggle-switch .slider {
        position: relative;
        width: 40px;
        height: 20px;
        background-color: gray;
        border-radius: 20px;
        cursor: pointer;
        transition: background-color 0.2s;
    }
    .toggle-switch .slider:before {
        content: "";
        position: absolute;
        width: 18px;
        height: 18px;
        border-radius: 50%;
        background-color: white;
        top: 1px;
        left: 1px;
        transition: transform 0.2s;
    }
    .toggle-switch input:checked + .slider {
        background-color: green;
    }
    .toggle-switch input:checked + .slider:before {
        transform: translateX(20px);
    }
    #plot {
        margin-top: 1rem;
    }
    .displacement-range {
        display: flex;
        align-items: center;
        margin-bottom: 1rem;
    }
    .displacement-range label {
        margin-right: 1rem;
        font-size: 1.5;
        font-family: 'Helvetica', sans-serif; 
    }
    .displacement-range input {
        margin-right: 1rem;
        font-size: 1.5;
        font-family: 'Helvetica', sans-serif; 
    }
    .displacement-range span{
        margin-right: 3rem;
    }
    .title {
        display: flex;
        flex-wrap: wrap;
        align-items: center;
        margin-right: 2rem;
        font-family: 'Hel';
    }
</style>