<script>
    import { onMount, afterUpdate } from "svelte";
	import { scaleLinear, scaleOrdinal } from "d3-scale"; 
	import { schemeReds, schemeCategory10 } from "d3-scale-chromatic";

	
    export let lectures;
    export let selectedSections = new Set()
    export let selectedLectureNumbers = [];
    export var getSelectedData; 
	export let heatmapRange;
    
	const numPrevPoints = 10
	const animationSpeed = 200

	//do elements go to front in animation? This is a bit resource intensive
	const bringToFront = true

    let heatmapXScale
    let heatmapYScale 

    let heatmapWidth;
	let heatmapHeight;

	let this_pid
	let greatest_pid = 0

	let pause_animation

	let animationColorScale
	let lecturesColorScale

	let selectedSectionsLocal = []


    onMount(async () => {
        heatmapXScale = scaleLinear()
			.domain(heatmapRange.x)
			//.domain([-1000,2000])
			.range([0,heatmapWidth])
		
		heatmapYScale = scaleLinear()
			.domain(heatmapRange.y)
			//.domain([heatmapStartingPoint.y-1000,heatmapStartingPoint.y + 1000])
			.range([0,heatmapHeight])
		
		//this isn't used
		animationColorScale = scaleOrdinal(schemeReds)
			.domain([numPrevPoints, 0])

		lecturesColorScale = scaleOrdinal(schemeCategory10)
			.domain(lectures.map((lect, idx) => idx))
		

    });

	afterUpdate(() => {
		//calculate the largest PID in the selected data. This is end of animation and timer.
		greatest_pid = 0
		selectedLectureNumbers.forEach(num => {
			let this_lecture_greatest_pid = Math.max(...lectures[num].data.map(e => e["PID"]))

			if (this_lecture_greatest_pid > greatest_pid)
				greatest_pid = this_lecture_greatest_pid
		})

		//console.log(greatest_pid)
		
		//to re-render component when selectedSections changes
		selectedSectionsLocal = selectedSections
	})

	function animate(start_pid) {
		//console.log("Animating")
		
		var counter = start_pid
		var timer = setInterval(() => {
			
			//pause when it sees that we've hit pause
			if (pause_animation) {
				clearInterval(timer)
			}

			//stop once we've reached the end of the animation
			if (counter >= greatest_pid) {
				clearInterval(timer)

				//TODO: Find out if we want to exit animation mode when animation ends
				//this_pid = undefined
			}

			//if we want to bring the current element to the front of the heatmap, that happens here
			//svelte orders elements based on position in the DOM, so this puts it at end, thus on top
			if (bringToFront) {
				var this_point = d3.select(`#heatmap-datapoint-${counter}`).node()
				if (this_point) {
					this_point.parentElement.appendChild(this_point)
				}
			}

			this_pid = counter;
			counter++;
		}, animationSpeed)

	}

	
	function playButtonClick() {
		pause_animation = false
		if (this_pid !== undefined)
			//if already in animation mode, start where we left off
			animate(this_pid)
		else
			//start at beginning
			animate(0)
	}

	function pauseButtonClick() {
		pause_animation = true

		//wait to make sure current iteration of animation has finished
		setTimeout(() => {
			//console.log("Paused")
		}, animationSpeed + 2)
	}

	
	function stopButtonClick() {
		pause_animation = true

		//wait to make sure current iteration of animation has finished
		setTimeout(() => { 
			//exit animation mode
			this_pid = undefined
		}, animationSpeed + 2)
	}

	function toStartButtonClick() {
		//pause the animation when this button is clicked
		pause_animation = true

		//wait to make sure current iteration of animation has finished
		setTimeout(() => { 
			//go back to beginning
			this_pid = 0
		}, animationSpeed + 2)
	}

	function toEndButtonClick() {
		//pause the animation when this button is clicked
		pause_animation = true
		setTimeout(() => { 
			//go to end
			this_pid = greatest_pid
		}, animationSpeed + 2)
	}

	function stepForwardButtonClick() {
		//pause the animation when this button is clicked
		pause_animation = true
		setTimeout(() => { 
			if (this_pid < greatest_pid) {
				//go to next point
				this_pid = this_pid + 1
			}
		}, animationSpeed + 2)
	}

	function stepBackwardButtonClick() {
		//pause the animation when this button is clicked
		pause_animation = true
		setTimeout(() => { 
			
			if (this_pid > 0) {
				//go to previous point
				this_pid = this_pid - 1
			}
		}, animationSpeed + 2)
	}

	//when a section is clicked, toggle whether it is selected
	function sectionClick(section_num) {
		if (selectedSections.has(section_num))
			selectedSections.delete(section_num)
		else 
			selectedSections.add(section_num)

		selectedSectionsLocal = selectedSections
		getSelectedData()
	}


</script>

<div style="height:100%;" bind:clientWidth={heatmapWidth} bind:clientHeight={heatmapHeight}>
	<svg style="height:100%; width:100%">

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


		{#if heatmapHeight !== undefined && heatmapWidth !== undefined}
			
			<g id="classroom-section-selectors-container">
				<rect
					class="classroom-section-selector"
					height="{heatmapHeight / 2}"
					width="{heatmapWidth / 2}"
					x="{heatmapWidth / 2}"
					y="0"
					fill="{selectedSectionsLocal.has(0) ? "darkgrey" : "transparent"}"
					on:click="{() => sectionClick(0)}" />

				<rect
					class="classroom-section-selector"
					height="{heatmapHeight / 2}"
					width="{heatmapWidth / 2}"
					x="{heatmapWidth / 2}"
					y="{heatmapHeight / 2}"
					fill="{selectedSectionsLocal.has(1) ? "darkgrey" : "transparent"}"
					on:click="{() => sectionClick(1)}" />

				<rect
					class="classroom-section-selector"
					height="{heatmapHeight / 2}"
					width="{heatmapWidth / 2}"
					x="0"
					y="{heatmapHeight / 2}"
					fill="{selectedSectionsLocal.has(2) ? "darkgrey" : "transparent"}"
					on:click="{() => sectionClick(2)}" />

				<rect
					class="classroom-section-selector"
					height="{heatmapHeight / 2}"
					width="{heatmapWidth / 2}"
					x="0"
					y="0"
					fill="{selectedSectionsLocal.has(3) ? "darkgrey" : "transparent"}"
					on:click="{() => sectionClick(3)}" />
			</g>
		{/if}
		
		{#if this_pid !== undefined}
			<text class="cancel-button-icon" x=25 y=45>X</text>
			<rect class="cancel-button" on:click="{() => stopButtonClick()}" />
		{/if}

	</svg>
	

</div>
<div style="height: 30px;">
	{#if this_pid !== undefined}
		<input style="width: 100%" type=range min=0 max={greatest_pid} 
			bind:value={this_pid} 
			on:click={() => pauseButtonClick()}/>
	{/if}	
</div>

<div class="view-icons">
	<button on:click={() => toStartButtonClick()}>
		<i class="fas fa-fast-backward fas2"id="i-back"></i>
	</button>
	<button on:click={() => stepBackwardButtonClick()}>
		<i class="fas fa-step-backward fas2"id="i-back"></i>
	</button>
	{#if pause_animation || this_pid === undefined}
		<button on:click="{() => playButtonClick()}">
			<i class="fas fa-play fas2"id="i-play"></i>
		</button>
	{:else}
		<button on:click="{() => pauseButtonClick()}">
			<i class="fas fa-pause fas2"id="i-pause"></i>
		</button>
	{/if}
	<button on:click={() => stepForwardButtonClick()}>
		<i class="fas fa-step-forward fas2"id="i-back"></i>
	</button>
	<button on:click="{() => toEndButtonClick()}">
		<i class="fas fa-fast-forward fas2"id="i-forward"></i>
	</button>
</div>

<!--<p>This PID: {this_pid}</p>-->

<style>
	.classroom-section-selector {
		opacity: 40%;
		z-index:1;
	}
	.classroom-section-selector:hover {
		cursor: pointer;
		fill: grey;
	}

	#heatmap-scatterplot circle {
		transition: r 0.2s;
	}

	
	.view-icons {
		display:flex;
		justify-content:center;
	}

	.view-icons button {
		font-size:30px;
		background-color: transparent;
		width: 50px;
		margin: 5px 0;
		border: none;
		margin: 0;
		cursor: pointer;
	}

	.cancel-button {
		fill: grey;
		opacity: 0%;
		height: 70px;
		width: 70px;
		cursor: pointer;
	}

	.cancel-button:hover {
		opacity: 35%;
	}

	.cancel-button-icon {
		fill: red;
		font-size:30px;
	}
</style>