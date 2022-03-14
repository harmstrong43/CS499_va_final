<script>
    import { onMount, afterUpdate } from "svelte";
	import { scaleLinear } from "d3-scale";

    export let lectures;
    export let selectedData = [];
//    export let selectedLectureNumbers = [];

	const sliceBarChartHeight = 200
	const sliceBarWidth = 40

    const numQuarters = 4

    let sliceYScale
	let sliceYScaleTicks = []

    //the data in each quarter of lecture time
    let quarterBins = Array(numQuarters).fill(null)

    //the length each bar should be
    let quarter_bars = Array(numQuarters).fill({length:0})

    onMount(async () => {
        sliceYScale = scaleLinear()
			.domain([1,0])
			.range([0,sliceBarChartHeight])

		sliceYScaleTicks = sliceYScale.ticks(5)
        
    });

    afterUpdate(() => {
        quarter_bars = []

        /*

        Do we want to have all data selected by default?

        if (selectedData === undefined){
            selectedData = lectures;
        }

        */

       
        //put data into bins for each quarter of lecture 
        for (let i = 0; i < numQuarters; i++) {
            quarterBins[i] = {
            quarter_num: i,
            lectures: []
            }

            for (let j = 0; j < selectedData.length; j++){
                quarterBins[i].lectures.push({
                    lecture_num: selectedData[j].lecture_num,
                    data: selectedData[j].data.filter(e => {
                        return ((e["Nr"] > i * lectures[selectedData[j].lecture_num].data.length / numQuarters) && (e["Nr"] <= (i+1) * lectures[selectedData[j].lecture_num].data.length / numQuarters))
                    })
                })
            }

            //the length each bar should be
            quarter_bars.push({
                length: getBarLength(quarterBins[i])
            })
        }
        //console.log(selectedData)
        //console.log(quarter_bars)
    })

    function getBarLength(quarter) {

        //console.log(quarter)

        //add all elements of array
        let reducer = (accumulator, curr) => accumulator + curr;

        //total number of elements of selected lectures in the quarter
        let quarter_length = quarter.lectures.map(e => e.data.length).reduce(reducer, 0)
       
        //total number of elements in the selected lectures
        let total_length = selectedData.map(e => e.data.length).reduce(reducer, 0)

        //console.log("Quarter_length:", quarter_length, "Total length:", total_length, "bar_length:", sliceYScale(quarter_length / total_length))

        if (total_length === 0){
            return sliceYScale(0)
        }
        else
            return sliceYScale(quarter_length / total_length)
    }

</script>

<div>
    <h4 style="margin:0;">Breakdown Per Quarter of Lecture</h4>           
    <svg style="height: 260px; padding: 20px">
        <g id="per-quarter-bar-chart" transform="translate(0,10)">
           <text style="font-size:0.5em; color: grey;" transform="translate(10, {sliceBarChartHeight / 1.35}) rotate(270)">Percentage of total time spend in quadrant.</text>
            <g class="chart-axes" transform="translate(20,0)">
                <line y1="0" y2="{sliceBarChartHeight}" stroke="black" stroke-width="1"/>

                {#each sliceYScaleTicks as tick}
                    <g transform="translate(0, {sliceYScale(tick)})">
                        <line x1="0" x2="5" stroke="black" stroke-width="0.5" />
                        <text x="10" y="6">{tick}</text>
                    </g>
                
                {/each}
            </g>

            <g id="per-quarter-bar-chart-bars" transform="translate(55,0)">
                {#if lectures !== undefined && selectedData !== undefined}
                    {#each quarter_bars as bar, idx}
                        <rect 
                            x="{(sliceBarWidth + 20) * idx}"
                            y="{bar.length}"
                            height={sliceBarChartHeight - bar.length}
                            width="{sliceBarWidth}"
                            style="fill:black;"
                        />
                        <text x="{(sliceBarWidth + 20) * idx}" y="{sliceBarChartHeight + 20}">Quar. {idx + 1}</text>
                        
                    {/each}
                {/if}
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