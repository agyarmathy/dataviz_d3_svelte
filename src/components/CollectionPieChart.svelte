<script>
    import { onMount } from "svelte";
    import { select, pie, arc, scaleOrdinal, schemeCategory10 } from "d3";
    import dataWebImage from "$lib/Images/dataWebImage.jpg";

    let data = [
        { name: "A", value: 10 },
        { name: "B", value: 20 },
        { name: "C", value: 30 },
        { name: "D", value: 40 },
        { name: "E", value: 11 },
        // ...more data...
    ];

    let svg;
    let width = 600;
    let height = 400;
    let radius = Math.min(width, height) / 2.5; // Adjust this value

    let color = scaleOrdinal(schemeCategory10);

    let pieGenerator = pie().value((d) => d.value);
    let arcGenerator = arc().innerRadius(0).outerRadius(radius);
    let selectedSection; // Add this line

    onMount(() => {
        let g = select(svg)
            .append("g")
            .attr("transform", `translate(${width / 2}, ${height / 2})`)
            .attr("class", "glass");

            

            let total = data.reduce((sum, d) => sum + d.value, 0);

let arcs = g.selectAll("path")
    .data(pieGenerator(data))
    .join("path")

        // Add this block
        let filter = g.append("defs").append("filter").attr("id", "grayscale");
        filter
            .append("feColorMatrix")
            .attr("type", "matrix")
            .attr(
                "values",
                "0.3333 0.3333 0.3333 0 0 0.3333 0.3333 0.3333 0 0 0.3333 0.3333 0.3333 0 0 0 0 0 1 0",
            );

        g.selectAll("path")
            .data(pieGenerator(data))
            .join("path")
            .attr("d", arcGenerator)
            .attr("fill", (d) => color(d.data.name))
            .each(function (d) {
                this._current = d;
            }) // Store the initial angles
            .on("mouseover", function (event, d) {
                // Change this block
                if (selectedSection) {
                    select(selectedSection)
                        .transition()
                        .attr("transform", "translate(0, 0)");
                }
                select(this)
                    .transition()
                    .attr(
                        "transform",
                        `translate(${
                            Math.cos(
                                (d.startAngle + d.endAngle) / 2 - Math.PI / 2,
                            ) * 10
                        }, ${
                            Math.sin(
                                (d.startAngle + d.endAngle) / 2 - Math.PI / 2,
                            ) * 10
                        })`,
                    );
                selectedSection = this;
                g.selectAll("path")
                    .filter(":not(:hover)")
                    .style("filter", "url(#grayscale)");
            })
            .on("mouseout", function (event, d) {
                // Add this block
                if (selectedSection) {
                    select(selectedSection)
                        .transition()
                        .attr("transform", "translate(0, 0)");
                    selectedSection = null;
                }
                g.selectAll("path").style("filter", null);
            });
            
            let legend = select(svg)
            .append("g")
            .attr("transform", `translate(0, ${height + 20})`) // Move the entire legend group down
            .selectAll(".legend")
            .data(data)
            .enter()
            .append("g")
            .attr("class", "legend")
            .attr("transform", (d, i) => `translate(0, ${i * 20})`); // Position each legend item

        legend.append("rect")
            .attr("x", width - 18)
            .attr("width", 18)
            .attr("height", 18)
            .style("fill", (d) => color(d.name));

        legend.append("text")
            .attr("x", width - 24)
            .attr("y", 9)
            .attr("dy", ".35em")
            .style("text-anchor", "end")
            .text((d) => d.name);
            
            
            arcs.append("text")
            .attr("transform", (d) => `translate(${arcGenerator.centroid(d)})`)
            .attr("dy", "0.35em")
            .text((d) => `${Math.round(100 * d.data.value / total)}%`);
            g.selectAll(".arc")
            .data(pieGenerator(data))
            .join("g")
            .attr("class", "arc")
            .append("text")
            .attr("transform", (d) => `translate(${arcGenerator.centroid(d)})`)
            .attr("dy", "0.35em")
            .text((d) => `${Math.round(100 * d.data.value / total)}%`);
    });
</script>

<div class="glass" style="background-image: url({dataWebImage});">
  <div class="svg-container">
    <svg bind:this={svg} {width} {height}></svg>
  </div>
</div>

<style>
  .glass {
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(5px);
    border-radius: 10px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    background-size: cover;
    background-position: center;
  }

  .svg-container {
    width: 600px;
    height: 400px;
    background: rgba(255, 255, 255, 0.1);
    backdrop-filter: blur(5px);
    border-radius: 10px;
    border: 1px solid rgba(255, 255, 255, 0.2);
  }
</style>