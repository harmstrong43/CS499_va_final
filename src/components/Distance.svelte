<script>
    import { scaleLinear } from "d3-scale";

    export let lectures = [];  
    export let colorScale;
    export let pixelID;
    export let distancePixel;
    export let distLines = [];
    export let hovered = false;
    export let currentIndex = undefined;
    export let selectedLectureNumbers = [0,1,2,3];
    
    let distPixelIDXScale;
    let distancePixelScale;
	let distancePixelTicks = [];
	let distpixelIDTicks = [];

    export const type = "Distance";
    const instructors = [0,1,2,3];
    const chartWidth = 420;
	const chartHeight = 420;
	const yAxisPadding = 40;
	const xAxisPadding = 40;
	const dividedAxisPadding = 10;
	const yAxisLength = chartHeight - 2 * yAxisPadding;
	const xAxisLength = chartWidth - 2 * xAxisPadding;
	const distxAxisLength = ((xAxisLength) / 2) - dividedAxisPadding;
	const distyAxisLength = ((yAxisLength) / 2) - dividedAxisPadding;
    const numTicks = 11;

    //Returns true if value is two or three, used to position axes
    function isTwoOrThree(index) {
		if (index === 2 || index === 3) {
			return 1;
		}
		else {
			return 0;
		}
	}
    //Scale Axes
    distPixelIDXScale = scaleLinear()
    .domain([
        1,
        Math.max(...pixelID.map(x => Math.max(...x)))
    ])
    .range([0, distxAxisLength]);

    
	distpixelIDTicks = distPixelIDXScale.ticks(numTicks);

    distancePixelScale = scaleLinear()
    .domain([
        0,
        Math.max(...distancePixel.map(x => Math.max(...x.slice(1))))
    ])
    .range([0, distyAxisLength]);

    distancePixelTicks = distancePixelScale.ticks(11);

</script>

<svg id = "d2p" width = "{chartWidth}px" height = "{chartHeight}px">
    <g id = "axis-lines" transform = "translate({xAxisPadding}, {yAxisPadding})">

        <line class = "axes"
            x2 = "{(chartWidth - 2 * xAxisPadding) / 2}"
            y1 = "{chartHeight - 2 * yAxisPadding}" y2 = "{chartHeight - 2 * yAxisPadding}" 
        />

        <line class = "axes"
            x2 = "{(chartWidth - 2 * xAxisPadding) / 2}"
            y1 = "{(chartHeight - 2 * yAxisPadding) - (distyAxisLength + 2 * dividedAxisPadding)}" 
            y2 = "{(chartHeight - 2 * yAxisPadding) - (distyAxisLength + 2 * dividedAxisPadding)}" 
        />

        <line class = "axes"
            x1 = "{((chartWidth - 2 * xAxisPadding) / 2) + dividedAxisPadding}" 
            x2 = {(chartWidth - 2 * xAxisPadding)}
            y1 = "{chartHeight - 2 * yAxisPadding}" y2 = "{chartHeight - 2 * yAxisPadding}" 
        />

        <line class = "axes"
            x1 = "{((chartWidth - 2 * xAxisPadding) / 2) + dividedAxisPadding}" 
            x2 = {(chartWidth - 2 * xAxisPadding)}
            y1 = "{(chartHeight - 2 * yAxisPadding) - (distyAxisLength + 2 * dividedAxisPadding)}" 
            y2 = "{(chartHeight - 2 * yAxisPadding) - (distyAxisLength + 2 * dividedAxisPadding)}"
        />

        {#each distpixelIDTicks as tick}
            <g id = "x-tick-labels-first-column" transform = "translate({distPixelIDXScale(tick)}, {chartHeight - 2 * yAxisPadding})">
                <line class = "axes" y2 = "5" />
                <text class = "tick-labels smaller" x="-5" y="12">{tick.toFixed(0)}</text>
            </g> 
            <g id = "x-tick-labels-second-column" transform = "translate({distPixelIDXScale(tick) + (distxAxisLength + 2 * dividedAxisPadding)}, {chartHeight - 2 * yAxisPadding})">
                <line class = "axes" y2 = "5" />
                <text class = "tick-labels smaller" x="-5" y="12">{tick.toFixed(0)}</text>
            </g>
        {/each}
        {#each distancePixelTicks as tick}
            <g id = "y-tick-labels-first-row" transform = "translate(0, {(chartHeight - 2 * yAxisPadding) - distancePixelScale(tick)})">
                <text class = "tick-labels smaller" x="-12" y = "1">{tick.toFixed(0)}</text>
            </g> 
            <g id = "y-tick-labels-second-row" transform = "translate(0, {(chartHeight - 2 * yAxisPadding) - (distyAxisLength + 2 * dividedAxisPadding) - distancePixelScale(tick)})">
                <text class = "tick-labels smaller" x="-12" y = "1">{tick.toFixed(0)}</text>
            </g> 
        {/each}
        {#each instructors as index}
            <g id = "line-chart" transform = "translate({(index % 2) * (distxAxisLength + 2 * dividedAxisPadding)}, {(chartHeight - 2 * yAxisPadding) - (isTwoOrThree(index) * (distyAxisLength + 2 * dividedAxisPadding))})">
                <polyline class = "chart-line"
                    bind:this={distLines[index]} 
                    style = "{selectedLectureNumbers.includes(index) ? "stroke: " + colorScale(index) : "display: none"}"
                    points = "{lectures[index].data.slice(1).map(point => distPixelIDXScale(point['PID']) + "," + (-1 * distancePixelScale(point['D2P [pixel]']))).join(" ")}" 
                    on:mouseenter={() => {
                        hovered = true;
                        currentIndex = index;
                    }}
                    on:mouseleave={() => {
                        hovered = false;
                        currentIndex = index;
                    }} 
                />
                <line class = "faint-lines"
                    x2 = {distxAxisLength}
                    y1 = {-1 * distancePixelScale(300)} y2 = {-1 * distancePixelScale(300)}
                    style = "stroke: gray; opacity: 0.2;"
                />
                <line class = "faint-lines"
                    x2 = {distxAxisLength}
                    y1 = {-1 * distancePixelScale(600)} y2 = {-1 * distancePixelScale(600)}
                    style = "stroke: gray; opacity: 0.2;"
                />
            </g>
        {/each}
    </g>
    <text 
        x = "{(chartWidth - 2 * yAxisPadding) / 2}" 
        y = "{chartHeight - (yAxisPadding / 3) + 7}">
        Pixel ID
    </text>
    <text 
        transform="translate({xAxisPadding / 2}, {(chartHeight - yAxisPadding) /2 }) rotate(270)" 
        text-anchor = "middle">
        D2P[Pixel]
    </text>
</svg>


<style>

    polyline.chart-line {
        stroke-width: 1;
        fill: none;
        opacity: 0.6;
    }
    line.axes {
        stroke: gray;
        opacity: 1;
    }
    text.smaller {
		font-size: 6px;
        fill: gray;
	}
    svg {
		margin: 5px;
	}

</style>