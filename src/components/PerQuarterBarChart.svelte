<script>
    import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";

    export let lectures;
    export let selectedData;
    export let selectedLectureNumbers = [];

	const sliceBarChartHeight = 300
	const sliceBarWidth = 40

    let sliceYScale
	let sliceYScaleTicks = []

    onMount(async () => {
        sliceYScale = scaleLinear()
			.domain([1,0])
			.range([0,sliceBarChartHeight])

		sliceYScaleTicks = sliceYScale.ticks(10)
    });

</script>

<div>
    <h4>Breakdown Per Quarter of Lecture</h4>           
    <svg style="height: {sliceBarChartHeight + 60}; padding: 20px">
        <g id="per-quarter-bar-chart" transform="translate(0,10)">
           <g class="chart-axes">
                <line y1="0" y2="{sliceBarChartHeight}" stroke="black" stroke-width="1"/>

                {#each sliceYScaleTicks as tick}
                    <g transform="translate(0, {sliceYScale(tick)})">
                        <line x1="0" x2="5" stroke="black" stroke-width="0.5" />
                        <text x="10" y="6">{tick}</text>
                    </g>
                
                {/each}
            </g>

            <g id="per-quarter-bar-chart-bars" transform="translate(40,0)">
                {#each [0,1,2,3] as idx}
                    {#if lectures && selectedData && selectedData.length !== 0 }
                        {#each selectedLectureNumbers as selectedLectureNum}
					        {#if lectures[selectedLectureNum] !== undefined}
					
                                <rect 
                                    x="{(sliceBarWidth + 20) * idx}"
                                    y="{sliceYScale(selectedData.filter((e) => {
                                        return ((e["Nr"] > idx * lectures[selectedLectureNum].data.length / 4) && (e["Nr"] <= (idx+1) * lectures[selectedLectureNum].data.length / 4))
                                    }).length / selectedData.length)}"
                                    height="{sliceBarChartHeight - sliceYScale(selectedData.filter((e) => {
                                        return ((e["Nr"] > idx * lectures[selectedLectureNum].data.length / 4) && (e["Nr"] <= (idx+1) * lectures[selectedLectureNum].data.length / 4))
                                    }).length / selectedData.length)}"
                                    width="{sliceBarWidth}"
                                    style="fill:black;"
                                />	
                            {/if}
                        {/each}
                    {/if}
                    <text x="{(sliceBarWidth + 20) * idx}" y="{sliceBarChartHeight + 20}">Quar. {idx + 1}</text>
                
                {/each}
            </g>
        </g>
    </svg>
</div>

<style>
	.chart-axes {
		font-size: 0.8em;
		color: gray;
	}

	svg {
		margin: 5px;
	}

    rect {
        transition: y 1.5s, height 1.5s;
    }

</style>