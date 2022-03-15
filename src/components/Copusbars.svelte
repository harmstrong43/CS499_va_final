<script>
    import { scaleLinear } from "d3-scale";
import { afterUpdate } from "svelte";

    export let colorScale;
    export let selectedData;
    export let lectureActivity;
	export let questionAskingActivity;
    export let questionAnsweringActivity;
    export let otherActivity;
    export let bars = {
        lecture: [],
        asking: [],
        answering: [],
        other: []
    };
    export let hovered = false;
    export let currentIndex = undefined;
    export let selectedLectureNumbers = [0,1,2,3];

    let lectureActivityArr = [];
	let questionAskingActivityArr = [];
	let questionAnsweringActivityArr = [];
	let otherActivityArr = [];
    let percentTimeScale;
    let percentTimeTicks = [];

    const instructors = [0,1,2,3];
    const chartWidth = 420;
	const chartHeight = 420;
	const yAxisPadding = 40;
	const xAxisPadding = 40;
	const yAxisLength = chartHeight - 2 * yAxisPadding;
	const xAxisLength = chartWidth - 2 * xAxisPadding;
	const barPadding = 15;
	const barWidth = 12;
	const barSections = xAxisLength / 4
    const numTicks = 11;

    //sums an array
    function sum(arr) {
		return arr.reduce(function (a, b) {
            if(a === NaN){
                a = 0;
            }
            if(b === NaN)
            {
			    b = 0;
            }
            return a + b;
		}, 0);
	}

    afterUpdate(() => {
        lectureActivity = selectedData.map(x => x['data'].map(lecturing => lecturing['lecturing']));
        console.log("selected Data: ", selectedData);
        console.log("Lecture Activity: ", lectureActivity);
		questionAskingActivity = selectedData.map(x => x['data'].map(questions => questions['asking_questions']));
		questionAnsweringActivity = selectedData.map(x => x['data'].map(answers => answers['answering_questions']));
		otherActivity = selectedData.map(x => x['data'].map(answers => answers['other']));
        lectureActivityArr = lectureActivity.map(x => x.length === 0 ? 0 : sum(x) / x.length);
        lectureActivityArr.push(sum(lectureActivityArr) / selectedData.length);
        questionAskingActivityArr = questionAskingActivity.map(x => x.length === 0 ? 0 : sum(x) / x.length);	
        questionAskingActivityArr.push(sum(questionAskingActivityArr) / selectedData.length);
        questionAnsweringActivityArr = questionAnsweringActivity.map(x => x.length === 0 ? 0 : sum(x) / x.length);	
        questionAnsweringActivityArr.push(sum(questionAnsweringActivityArr) / selectedData.length);
        otherActivityArr =  otherActivity.map(x => x.length === 0 ? 0 : sum(x) / x.length);		
        otherActivityArr.push(sum(otherActivityArr) / selectedData.length);
        console.log("Activity: ", lectureActivityArr);
    })

    percentTimeScale = scaleLinear()
    .domain([
        0,
        1
    ])
    .range([0, yAxisLength]);

    percentTimeTicks = percentTimeScale.ticks(numTicks);

</script>

<svg id = "Total-activity" width = "{chartWidth}px" height = "{chartHeight}px">
    <g id = "axes-3" transform = "translate({xAxisPadding}, {yAxisPadding})">
        <line class = "axes" 
        y2 = "{chartHeight - 2 * yAxisPadding}" />
        <line class = "axes"
        x2 = "{chartWidth - 2 * xAxisPadding}"
        y1 = "{chartHeight - 2 * yAxisPadding}" y2 = "{chartHeight - 2 * yAxisPadding}" />
        {#each percentTimeTicks.slice(1) as tick}
            <g id = "y-tick-labels-3" transform = "translate(0, {(chartHeight - 2 * yAxisPadding) - percentTimeScale(tick)})">
                <line class = "axes" x2 = "5" />
                <text class = "tick-labels" x="9">{tick.toFixed(1)}</text>
            </g> 
        {/each}
        {#if percentTimeScale !== undefined}
            {#each instructors as index}
                <g id = "bars" transform = "translate({(index + 2) * barPadding}, {(chartHeight - 2 * yAxisPadding)})">
                    <rect
                        bind:this = {bars.lecture[index]}
                        x = 0 y= {selectedLectureNumbers.includes(index) ? -1 * percentTimeScale(lectureActivityArr[selectedLectureNumbers.indexOf(index)]) : 0}
                        height = {selectedLectureNumbers.includes(index) ? percentTimeScale(lectureActivityArr[selectedLectureNumbers.indexOf(index)]) : 0}
                        width = {barWidth}
                        style = "{selectedLectureNumbers.includes(index) ? "fill: " + colorScale(index) : "display: none"}"
                        on:mouseenter={() => {
                            hovered = true;
                            currentIndex = index;
                        }}
                        on:mouseleave={() => {
                            hovered = false;
                            currentIndex = index;
                        }} 
                    />

                    <rect 
                        bind:this = {bars.asking[index]}
                        x = {barSections * 1} y= {selectedLectureNumbers.includes(index) ? -1 * percentTimeScale(questionAskingActivityArr[selectedLectureNumbers.indexOf(index)]) : 0}
                        height = {selectedLectureNumbers.includes(index) ? percentTimeScale(questionAskingActivityArr[selectedLectureNumbers.indexOf(index)]) : 0}
                        width = {barWidth}
                        style = "{selectedLectureNumbers.includes(index) ? "fill: " + colorScale(index) : "display: none"}"
                        on:mouseenter={() => {
                            hovered = true;
                            currentIndex = index;
                        }}
                        on:mouseleave={() => {
                            hovered = false;
                            currentIndex = index;
                        }}
                    />

                    <rect 
                        bind:this = {bars.answering[index]}
                        x = {barSections * 2} y= {selectedLectureNumbers.includes(index) ? -1 * percentTimeScale(questionAnsweringActivityArr[selectedLectureNumbers.indexOf(index)]) : 0}
                        height = {selectedLectureNumbers.includes(index) ? percentTimeScale(questionAnsweringActivityArr[selectedLectureNumbers.indexOf(index)]) : 0}
                        width = {barWidth}
                        style = "{selectedLectureNumbers.includes(index) ? "fill: " + colorScale(index) : "display: none"}"
                        on:mouseenter={() => {
                            hovered = true;
                            currentIndex = index;
                        }}
                        on:mouseleave={() => {
                            hovered = false;
                            currentIndex = index;
                        }}
                    />

                    <rect 
                        bind:this = {bars.other[index]}
                        x = {barSections * 3} y= {selectedLectureNumbers.includes(index) ? -1 * percentTimeScale(otherActivityArr[selectedLectureNumbers.indexOf(index)]) : 0}
                        height = {selectedLectureNumbers.includes(index) ? percentTimeScale(otherActivityArr[selectedLectureNumbers.indexOf(index)]) : 0}
                        width = {barWidth}
                        style = "{selectedLectureNumbers.includes(index) ? "fill: " + colorScale(index) : "display: none"}"
                        on:mouseenter={() => {
                            hovered = true;
                            currentIndex = index;
                        }}
                        on:mouseleave={() => {
                            hovered = false;
                            currentIndex = index;
                        }}
                    />
                </g>
            {/each}
            <g id = "average-lines" transform = "translate({2 * barPadding}, {(chartHeight - 2 * yAxisPadding)})">

                <rect class = "average-padding"
                    x = "0" y = "{-1 * percentTimeScale(lectureActivityArr.slice(-1)[0]) - 2}"
                    width = "{4 * (barPadding) - 3}" height = 4
                />

                <line class = "averages"
                    x1 = "0" x2 = "{4 * (barPadding) - 3}"
                    y1= "{-1 * percentTimeScale(lectureActivityArr.slice(-1)[0])}" y2 = "{-1 * percentTimeScale(lectureActivityArr.slice(-1)[0])}"
                />

                <rect class = "average-padding"
                    x = "{barSections * 1}" y = "{-1 * percentTimeScale(questionAskingActivityArr.slice(-1)[0]) - 2}"
                    width = "{4 * (barPadding) - 3}" height = 4
                />

                <line class = "averages"
                    x1 = "{barSections * 1}" x2 = {(4 * barPadding) + (barSections * 1) - 3}
                    y1 = {-1 * percentTimeScale(questionAskingActivityArr.slice(-1)[0])} y2 = {-1 * percentTimeScale(questionAskingActivityArr.slice(-1)[0])}
                />

                <rect class = "average-padding"
                    x = "{barSections * 2}" y = "{-1 * percentTimeScale(questionAnsweringActivityArr.slice(-1)[0]) - 2}"
                    width = "{4 * (barPadding) - 3}" height = 4
                />

                <line class = "averages"
                    x1 = "{barSections * 2}" x2 = "{(4 * barPadding) + (barSections * 2) - 3}"
                    y1 = {-1 * percentTimeScale(questionAnsweringActivityArr.slice(-1)[0])} y2 = {-1 * percentTimeScale(questionAnsweringActivityArr.slice(-1)[0])}
                />

                <rect class = "average-padding"
                    x = "{barSections * 3}" y = "{-1 * percentTimeScale(otherActivityArr.slice(-1)[0]) - 2}"
                    width = "{4 * (barPadding) - 3}" height = 4
                />

                <line class = "averages"
                    x1 = "{barSections * 3}" x2 = "{(4 * barPadding) + (barSections * 3) - 3}"
                    y1 = {-1 * percentTimeScale(otherActivityArr.slice(-1)[0])} y2 = {-1 * percentTimeScale(otherActivityArr.slice(-1)[0])}
                />
            </g>
            <g id = "bar-labels" transform = "translate({(4) * barPadding}, {(chartHeight - (7/4) * yAxisPadding)})">
                <text class = "activity-bars" x = "-20">Lecturing</text>
                <text class = "activity-bars" x = "{barSections * 1 - 32}">Asking Questions</text>
                <text class = "activity-bars" x = "{barSections * 2 - 38}">Answering Questions</text>
                <text class = "activity-bars" x = "{barSections * 3 - 13}">Other</text>
            </g>
        {/if}
    </g>
    <text 
        x = "{(chartWidth - 2 * yAxisPadding) / 2}" 
        y = "{chartHeight - 5}">
        Activity
    </text>
    <text 
    transform="translate({xAxisPadding / 2}, {(chartHeight - yAxisPadding) /2 }) rotate(270)" text-anchor = "middle">% of Time Spent</text>
</svg>

<style>

    line.axes {
		stroke: gray;
		stroke-width: 2;
	}
	line.averages {
		stroke: black;
		stroke-width: 0.8;
		opacity: 1;
	}
	rect.average-padding {
		fill: white;
		opacity: 0.8;
	}
	text.tick-labels {
		font-size: 10px;
        fill: gray;
	}
	text.activity-bars {
		font-size: 8px;
		text-align: "center";
	}
	svg {
		margin: 5px;
	}
    rect {
        transition: y 1.5s, height 1.5s;
    }

</style>