<script>
    import { onMount } from "svelte";
    import {
        select,
        scaleBand,
        scaleLinear,
        max,
        axisBottom,
        axisLeft,
    } from "d3";

    let data = [
        { name: "A", value: 10 },
        { name: "B", value: 20 },
        { name: "C", value: 30 },
        { name: "D", value: 40 },
        
    ];

    let colorScale; 

    let svg;
    let width = 600;
    let height = 400;
    let margin = { top: 20, right: 20, bottom: 30, left: 40 };
    let sortByValueOrder = true; 
    let isSortByValueClicked = false; 

    let xScale, yScale, xAxis, yAxis;

    onMount(() => {
        xScale = scaleBand()
            .domain(data.map((d) => d.name))
            .range([margin.left, width - margin.right])
            .padding(0.1);

        yScale = scaleLinear()
            .domain([0, max(data, (d) => d.value)])
            .nice()
            .range([height - margin.bottom, margin.top]);

        xAxis = (g) =>
            g
                .attr("transform", `translate(0,${height - margin.bottom})`)
                .call(axisBottom(xScale).tickSizeOuter(0));

        yAxis = (g) =>
            g
                .attr("transform", `translate(${margin.left},0)`)
                .call(axisLeft(yScale))
                .call((g) => g.select(".domain").remove());

        let g = select(svg).append("g");

        colorScale = scaleLinear() 
    .domain([0, max(data, d => d.value)])
    .range(['#0077be', '#000080']); 

        g.selectAll("rect")
            .data(data)
            .join("rect")
            .attr("x", (d) => xScale(d.name))
            .attr("y", (d) => yScale(d.value))
            .attr("height", (d) => yScale(0) - yScale(d.value))
            .attr("width", xScale.bandwidth())
            .attr("fill", (d) => colorScale(d.value)); 

        g = select(svg).append("g");

        g.append("g")
            .attr("class", "x-axis") 
            .call(xAxis);

        g.append("g").call(yAxis);
    });

    function sortByValue() {
        isSortByValueClicked = true;
        if (!sortByValueOrder) {
            data.sort((a, b) => a.value - b.value);
        } else {
            data.sort((a, b) => b.value - a.value);
        }
        sortByValueOrder = !sortByValueOrder; 
        updateChart();
    }

    let sortByNameOrder = true; 
    let isSortByNameClicked = false; 

    function sortByName() {
        isSortByNameClicked = true;
        if (sortByNameOrder) {
            data.sort((a, b) => a.name.localeCompare(b.name));
        } else {
            data.sort((a, b) => b.name.localeCompare(a.name));
        }
        sortByNameOrder = !sortByNameOrder; 
        updateChart();
    }

    function updateChart() {
        xScale.domain(data.map((d) => d.name));

        colorScale.domain([0, max(data, d => d.value)]); 


        select(svg).selectAll('rect')
      .data(data)
      .join('rect')
      .transition()
      .duration(1000)
      .attr('x', d => xScale(d.name))
      .attr('y', d => yScale(d.value))
      .attr('height', d => yScale(0) - yScale(d.value))
      .attr('fill', d => colorScale(d.value)); 


        
        xAxis = (g) =>
            g
                .attr("transform", `translate(0,${height - margin.bottom})`)
                .call(axisBottom(xScale).tickSizeOuter(0));

        select(svg).select(".x-axis").transition().duration(1000).call(xAxis);
    }
</script>

<button on:click={sortByValue}>
    Sort by Value {isSortByValueClicked ? (sortByValueOrder ? "↑" : "↓") : ""}
    
</button>
<button on:click={sortByName}>
    Sort by Name {isSortByNameClicked ? (sortByNameOrder ? "↑" : "↓") : ""}
    
</button>
<svg bind:this={svg} {width} {height}></svg>
