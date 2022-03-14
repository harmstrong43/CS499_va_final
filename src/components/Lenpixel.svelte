<script>
    import { scaleLinear } from "d3-scale";

    export let lectures = [];  
    export let colorScale;
    export let pixelID;
    export let lenPixel;
    export let lines = [];
    export let currentIndex = undefined;
    export let hovered = false;
    export let selectedLectureNumbers = [0,1,2,3];
    
	let lenPixelTicks = [];
    let pixelIDXScale;
    let lenPixelScale;
    let pixelIDTicks = [];

    export const type = "Length";
    const chartWidth = 420;
    const instructors = [0,1,2,3];
	const chartHeight = 420;
	const yAxisPadding = 40;
	const xAxisPadding = 40;
	const yAxisLength = chartHeight - 2 * yAxisPadding;
	const xAxisLength = chartWidth - 2 * xAxisPadding;
    const numTicks = 11;

    pixelIDXScale = scaleLinear()
    .domain([
        1,
        Math.max(...pixelID.map(x => Math.max(...x)))
    ])
    .range([0, xAxisLength]);


    pixelIDTicks = pixelIDXScale.ticks(numTicks);

    lenPixelScale = scaleLinear()
    .domain([
        0,
        Math.max(...lenPixel.map(x => Math.max(...x)))
    ])
    .range([0, yAxisLength]);

    lenPixelTicks = lenPixelScale.ticks(numTicks);

</script>

{#if lectures !== undefined}
    <svg id = 'len-pixel-chart' width = "{chartWidth}px" height = "{chartHeight}px">	
        <g id = "axes" transform = "translate({xAxisPadding}, {yAxisPadding})">
            <line class = "axes" 
            y2 = "{chartHeight - 2 * yAxisPadding}" />
            <line class = "axes"
            x2 = "{chartWidth - 2 * xAxisPadding}"
            y1 = "{chartHeight - 2 * yAxisPadding}" y2 = "{chartHeight - 2 * yAxisPadding}" />
            {#each pixelIDTicks as tick}
                <g id = "x-tick-labels" transform = "translate({pixelIDXScale(tick)}, {chartHeight - 2 * yAxisPadding})">
                    <line class = "axes" y2 = "-5" />
                    <text class = "tick-labels" x="-9" y="-8">{tick.toFixed(0)}</text>
                </g> 
            {/each}
            {#each lenPixelTicks.slice(1) as tick}
                <g id = "y-tick-labels" transform = "translate(0, {(chartHeight - 2 * yAxisPadding) - lenPixelScale(tick)})">
                    <line class = "axes" x2 = "5" />
                    <text class = "tick-labels" x="9">{tick.toFixed(0)}</text>
                </g> 
            {/each}
            {#each instructors as index}
                <g id = "line-chart" transform = "translate(0, {(chartHeight - 2 * yAxisPadding)})">
                    <polyline class = "chart-line" 
                    bind:this={lines[index]}
                    style = "{selectedLectureNumbers.includes(index) ? "stroke: " + colorScale(index) : "display: none"}"
                    points = "{lectures[index].data.map(point => pixelIDXScale(point['PID']) + "," + (-1 * lenPixelScale(point['Len [pixel]']))).join(" ")}"
                    on:mouseenter={() => {
                        currentIndex = index;
                        hovered = true;
                    }}
                    on:mouseleave={() => {
                        currentIndex = index;
                        hovered = false;
                    }} />
                </g>
            {/each}
        </g>
        <text x = "{(chartWidth - 2 * yAxisPadding) / 2}" y = "{chartHeight - (yAxisPadding / 3)}">Pixel ID</text>
        <text transform="translate({xAxisPadding / 2}, {(chartHeight - yAxisPadding) /2 }) rotate(270)" text-anchor = "middle">Len[Pixel]</text>
    </svg>
{/if}

<style>
    line.axes {
		stroke: gray;
		stroke-width: 2;
	}
    polyline.chart-line {
		stroke-width: 2;
		fill: none;
		opacity: 0.8;
	}
    text.tick-labels {
		font-size: 10px;
        fill: gray;
	}
    svg {
		margin: 5px;
	}

</style>