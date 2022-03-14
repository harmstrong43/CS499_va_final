<script>
    import { onMount, afterUpdate } from "svelte";
	import { scaleLinear, scaleOrdinal } from "d3-scale"; 
	import { schemeReds, schemeCategory10 } from "d3-scale-chromatic";

	
    export let lectures;
    export let selectedSection = undefined;
    export let selectedLectureNumbers = undefined;
    export var getSelectedData; 
	export let heatmapRange;
    

	const numPrevPoints = 10

    let heatmapXScale
    let heatmapYScale 

    let heatmapWidth;
	let heatmapHeight;

	let this_pid
	let greatest_pid = 0

	let pause_animation

	let animationColorScale
	let lecturesColorScale


    onMount(async () => {
        heatmapXScale = scaleLinear()
			.domain(heatmapRange.x)
			//.domain([-1000,2000])
			.range([0,heatmapWidth])
		
		heatmapYScale = scaleLinear()
			.domain(heatmapRange.y)
			//.domain([heatmapStartingPoint.y-1000,heatmapStartingPoint.y + 1000])
			.range([0,heatmapHeight])
		

		animationColorScale = scaleOrdinal(schemeReds)
			.domain([numPrevPoints, 0])

		lecturesColorScale = scaleOrdinal(schemeCategory10)
			.domain(lectures.map((lect, idx) => idx))

		

    });

	afterUpdate(() => {
		lectures.filter(e => e.copus === false).forEach(lect => {
			let this_lecture_greatest_pid = Math.max(...lect.data.map(e => e["PID"]))

			if (this_lecture_greatest_pid > greatest_pid)
				greatest_pid = this_lecture_greatest_pid
		})
	})
/*
	function getFillColor(pid) {
		if (this_pid !== undefined) {
			let x = this_pid - pid 
			if (x >= 0 && x <= 10 ){
				return colorScale(x)
			} else {
				return "lightgrey"
			}
		}
		return "blue"

	}
*/


	function animate(start_pid) {
		console.log("Animating")
		
		var counter = start_pid
		var timer = setInterval(() => {
			//pause when we hit 
			if (pause_animation) {
				clearInterval(timer)
			}

			//stop once we've reached the end of the animation
			if (counter >= greatest_pid) {
				clearInterval(timer)
				this_pid = undefined
			}
			this_pid = counter;
			counter++;
		}, 100)

	}

	function playButtonClick() {
		animate(0)
	}

	function pauseButtonClick() {
		pause_animation = !pause_animation
		if (!pause_animation){
			animate(this_pid)
		}
	}

	function stopButtonClick() {
		pause_animation = true
		this_pid = undefined
	}


</script>

<div style="height:80%;" bind:clientWidth={heatmapWidth} bind:clientHeight={heatmapHeight}>
	<svg style="height:100%; width:100%">
		{#if heatmapHeight !== undefined && heatmapWidth !== undefined}
			
			<g id="classroom-section-selectors-box">
				<rect
					class="classroom-section-selector"
					height="{heatmapHeight / 2}"
					width="{heatmapWidth / 2}"
					x="{heatmapWidth / 2}"
					y="0"
					fill="{selectedSection !== undefined && selectedSection === "Q1" ? "#CDCDCD" : "transparent"}"
					on:click="{() => {
						selectedSection = "Q1";
						getSelectedData()
					}}" />

				<rect
					class="classroom-section-selector"
					height="{heatmapHeight / 2}"
					width="{heatmapWidth / 2}"
					x="{heatmapWidth / 2}"
					y="{heatmapHeight / 2}"
					fill="{selectedSection !== undefined && selectedSection === "Q2" ? "#CDCDCD" : "transparent"}"
					on:click="{() => {
						selectedSection = "Q2";
						getSelectedData()
					}}" />

				<rect
					class="classroom-section-selector"
					height="{heatmapHeight / 2}"
					width="{heatmapWidth / 2}"
					x="0"
					y="{heatmapHeight / 2}"
					fill="{selectedSection !== undefined && selectedSection === "Q3" ? "#CDCDCD" : "transparent"}"
					on:click="{() => {
						selectedSection = "Q3";
						getSelectedData()
					}}" />

				<rect
					class="classroom-section-selector"
					height="{heatmapHeight / 2}"
					width="{heatmapWidth / 2}"
					x="0"
					y="0"
					fill="{selectedSection !== undefined && selectedSection === "Q4" ? "#CDCDCD" : "transparent"}"
					on:click="{() => {
						selectedSection = "Q4";
						getSelectedData()
					}}" />
			</g>
		{/if}

		<g id="heatmap-scatterplot">
			{#if lectures !== undefined && selectedLectureNumbers!== undefined}
				{#each selectedLectureNumbers as selectedLectureNum}
					{#if lectures[selectedLectureNum] !== undefined}
						{#each lectures[selectedLectureNum].data as datapoint, idx}
							<circle
								id="heatmap-datapoint-{datapoint["Nr"]}"
								style="
									{
										this_pid !== undefined 
										? this_pid - datapoint["PID"] >= 0 && this_pid - datapoint["PID"] <= 10
											//? `opacity: 100%; fill: ${animationColorScale(this_pid - datapoint["PID"])}`
											? `opacity: ${100 - ((this_pid - datapoint["PID"]) * 10)}%; fill: ${lecturesColorScale(selectedLectureNum)}`
											
											: "opacity: 30%; fill: lightgray"
										: "opacity: 30%; fill: blue;"
									}
								"
								cx={heatmapXScale(datapoint["x [pixel]"])}
								cy={heatmapHeight - heatmapYScale(datapoint["y [pixel]"])}
								r="{
									this_pid !== undefined
									? this_pid == datapoint["PID"]  
										? 8 
										: 3
									: 1.5
								}"
								
							/>

							{#if this_pid - datapoint["PID"] >= 0 && this_pid - datapoint["PID"] <= 10 && idx > 0}
								<line 
									x1={heatmapXScale(datapoint["x [pixel]"])}
									x2={heatmapXScale(lectures[selectedLectureNum].data[idx - 1]["x [pixel]"])}
									y1={heatmapHeight - heatmapYScale(datapoint["y [pixel]"])}
									y2={heatmapHeight - heatmapYScale(lectures[selectedLectureNum].data[idx - 1]["y [pixel]"])}
									opacity="{100 - ((this_pid - datapoint["PID"]) * 10)}%"
									stroke= {lecturesColorScale(selectedLectureNum)}
									stroke-width=1
								/>
							{/if}
						{/each}
					{/if}
				{/each}
			{/if}
		</g>
		
	</svg>

	
</div>
<button 
			style="width: 1000px; height: 200px; background-color: green;" 
			on:click="{() => playButtonClick()}">
			Animate
</button>
<button 
			style="width: 1000px; height: 200px; background-color: red;" 
			on:click="{() => pauseButtonClick()}">
			Pause
</button>
<input type=range value={this_pid} min=0 max={greatest_pid}>
<p>This PID: {this_pid}</p>

	

<style>
	.classroom-section-selector {
		opacity: 40%;
	}
	.classroom-section-selector:hover {
		cursor: pointer;
		fill: lightgray;
	}

	#heatmap-scatterplot circle {
		transition: r 0.2s;
	}
</style>