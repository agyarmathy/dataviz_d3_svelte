<script>
    import { onMount } from "svelte";
    import { select } from "d3-selection";
    import { Delaunay } from "d3-delaunay";
    import { zoom as d3zoom } from "d3-zoom";

    

    let data = [
        [80, 200],
        [210, 230],
        [330, 100],
        [120, 300],
        [400, 350],
        [250, 150],
        [100, 50],
        [300, 200],
        [200, 100],
        [150, 250],
        [350, 300],
        [50, 200],
        [250, 350],
        [400, 250],
        [300, 50],
        [200, 300],
        [350, 150],
        [100, 400],
        [300, 350],
        [200, 50],
        [150, 300],
        [350, 200],
        [50, 250],
        [250, 400],
        [400, 300],
        [300, 100],
        [200, 350],
        [350, 100],
        [100, 250],
        [300, 400],
        [200, 200],
        [150, 350],
        [350, 250],
    ];

    for (let i = 0; i < 200; i++) {
        data.push([Math.random() * 500, Math.random() * 500]);
    }

    let svg;

    onMount(() => {
        if (!svg) {
            console.error("SVG element is not available");
            return;
        }
        let delaunay = Delaunay.from(data);

        let g = select(svg).append("g");

        let zoom = d3zoom()
            .scaleExtent([1, 10]) 
            .on("zoom", zoomed);

        
        select(svg).call(zoom);

        function zoomed(event) {
            g.attr("transform", event.transform); 
        }


        

        g.selectAll("path")
            .data(delaunay.trianglePolygons())
            .join("path")
            .attr("d", (d) => `M${d.join("L")}Z`)
            .style("stroke", "url(#gradient)")
            .style("stroke-width", 0.3)
            .style("stroke-opacity", 0.9)
            .style("fill", "#0000cc")
            .style("fill-opacity", 0.0779);

        let neighborsCount = data.map(
            (_, i) => Array.from(delaunay.neighbors(i)).length,
        );
        let maxNeighbors = Math.max(...neighborsCount);
        let minNeighbors = Math.min(...neighborsCount);

        let deltaNeighbors = maxNeighbors - minNeighbors;

        g.selectAll("circle")
            .data(data)
            .join("circle")
            .attr("cx", (d) => d[0])
            .attr("cy", (d) => d[1])
            .attr("r", 3.4)
            .style("fill", "#b3f0ff")
            .style("filter", "url(#glow)")
            .each(function (d, i) {
                let index = data.findIndex(
                    (point) => point[0] === d[0] && point[1] === d[1],
                );
                let opacity = (neighborsCount[index] * deltaNeighbors) / 644;
                this._clicked = false; 
                let tooltip = document.getElementById("tooltip");

                select(this)
                    .style("fill-opacity", opacity)
                    .style("filter", "url(#glow)")
                    .on("mouseover", function () {
                        if (this._clicked) {
                            
                            return;
                        }

                        select(this).style("fill-opacity", 0.5);

                        tooltip.style.visibility = "visible";
                        tooltip.textContent = `(${d[0]}, ${d[1]})`;
                    })
                    .on("mouseout", function () {
                        if (!this._clicked) {
                            
                            select(this).style("fill-opacity", opacity);
                        }
                        tooltip.style.visibility = "hidden";
                    })
                    .on("click", function () {
                        this._clicked = !this._clicked; 
                        if (this._clicked) {
                            select(this).style("fill-opacity", 0.5);
                            let legend = document.getElementById("legend");
                            let p = document.createElement("p");
                            p.textContent = `(${d[0]}, ${d[1]})`;
                            p.id = `point-${i}`; 
                            legend.appendChild(p);
                            tooltip.style.visibility = "hidden"; 
                        } else {
                            select(this).style("fill-opacity", opacity);
                            let p = document.getElementById(`point-${i}`); 
                            if (p) {
                                p.remove(); 
                            }
                        }
                    });
            });
    });
</script>

<svg
    bind:this={svg}
    width="500"
    height="500"
    viewBox="0 0 500 500"
    style="background-color: #000000;"
>
    <defs>
        <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="0%">
            <stop
                offset="0%"
                style="stop-color:rgb(0,153,153);stop-opacity:0.3"
            />
            <stop
                offset="100%"
                style="stop-color:rgb(0,122,156);stop-opacity:0.5"
            />
            <feGaussianBlur stdDeviation=".5" result="coloredBlur" />
            <feMerge>
                <feMergeNode in="coloredBlur" />
                <feMergeNode in="SourceGraphic" />
            </feMerge>
        </linearGradient>
        <filter id="glow">
            <feGaussianBlur stdDeviation="260" result="coloredBlur" />
            <feMerge>
                <feMergeNode in="coloredBlur" />
                <feMergeNode in="SourceGraphic" />
            </feMerge>
        </filter>
    </defs>
</svg>
<div id="legend"></div>
<div id="tooltip" style="position: absolute; visibility: hidden;"></div>

<style>
    circle:hover {
        fill-opacity: 1 !important;
    }
</style>
