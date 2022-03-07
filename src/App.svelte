<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->
<script>
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";
	
	let lectures = [];
	
	let selectedData;
	let selectedSection;
	
	let lectureSelectVal;
	let selectedLectureNum;

	let sliceXScale
	let sliceYScale

	let sliceXScaleTicks
	let sliceYScaleTicks = []

	let percentDistance;

	const sliceBarChartHeight = 300
	const sliceBarWidth = 40

	const filenames = [
		"static/instructorA.json",
		"static/instructorB_1.json",
		"static/instructorB_2.json",
		"static/instructorB_3.json",
		"static/COPUS_instructorA.json",
		"static/COPUS_instructorA.json"
	]

	//checks to see if filename starts with COPUS
	function getCopus(filename) {
		if (filename.split("/")[1].split("_")[0] == "COPUS"){
			return true
		}
		return false
	}

	//Gets the name of the instructor from filename. This is currently a very manual process
	function getInstructorName(filename) {
		if (filename.includes("instructorA"))
			return "a"
		else if (filename.includes("instructorB")) 
			return "b"
		return null
	}

	onMount(async () => {

		//Loop through all the data files stored in filenames global variable and load them
		for(let i = 0; i < filenames.length; i++) {
			let fetched = await fetch(filenames[i])
			let fetched_json = await fetched.json();

			//is COPUS in title? TODO: Make sure this is actually something we care about
			let is_copus = getCopus(filenames[i]);

			//Get name of instructor. TODO: Do we want to change this to numbers?
			let instructor_name = getInstructorName(filenames[i]);
			
			//add new lecture object to the array with the data from the filename
			lectures.push({
				instructor_name: instructor_name,
				copus: is_copus,
				data: fetched_json
			})
		};
		console.log("Lectures: ", lectures)

		sliceYScale = scaleLinear()
			.domain([1,0])
			.range([0,sliceBarChartHeight])

		sliceYScaleTicks = sliceYScale.ticks(10)
		console.log(sliceYScaleTicks)
	});

	function filterSelectedData(xmin, xmax, ymin, ymax) {
		selectedData = lectures[selectedLectureNum].data.filter((e) => {
			return e["x [pixel]"] > xmin 
				&&  e["x [pixel]"] < xmax 
				&& e["y [pixel]"] > ymin 
				&& e["y [pixel]"] < ymax
		})
		getTotalDistance()
	}

	function getStartingPoint(){
		return ({
			x: lectures[selectedLectureNum].data[0]["x [pixel]"],
			y: lectures[selectedLectureNum].data[0]["y [pixel]"]
		})
	}

	function getSelectedData(){
		console.log("Selected lecture:", selectedLectureNum, "Selected section:", selectedSection)
		let starting_point = getStartingPoint()
		//console.log("Starting Point:", starting_point)
		if (selectedSection =="Q1")
			filterSelectedData(starting_point.x, Infinity, starting_point.y, Infinity)
		else if (selectedSection == "Q2")
			filterSelectedData(starting_point.x, Infinity, Number.NEGATIVE_INFINITY, starting_point.y)
		else if (selectedSection == "Q3")
			filterSelectedData(Number.NEGATIVE_INFINITY, starting_point.x, Number.NEGATIVE_INFINITY, starting_point.y)
		else if (selectedSection == "Q4")
			filterSelectedData(Number.NEGATIVE_INFINITY, starting_point.x, starting_point.y, Infinity)
		else
			selectedData = []
		//console.log(selectedData)					
	}

	function getTotalDistance() {
		let selectedDist = selectedData.reduce((accum, item) => {
			if (item["D2P [pixel]"] !== "NA" && item["D2P [pixel]"] !== "N/A")
				return accum + item["D2P [pixel]"]
			else
				return accum
		},0)
		let totalDist = lectures[selectedLectureNum].data.reduce((accum, item) => {
			if (item["D2P [pixel]"] !== "NA" && item["D2P [pixel]"] !== "N/A")
				return accum + item["D2P [pixel]"]
			else
				return accum
		},0)

		percentDistance = selectedDist / totalDist;
	}



</script>

<main>
	<h1>Section of Classroom View</h1>

	<div id="container">
		<div id="slice-view">
			<h4 id="slice-select-title">Select a Quarter of Classroom</h4>
			<select bind:value={selectedSection} on:change="{() => getSelectedData()}">
				<option value="Q1">Q1</option>
				<option value="Q2">Q2</option>
				<option value="Q3">Q3</option>
				<option value="Q4">Q4</option>
			</select>
			<h4 id="lecture-select-title">Select a Lecture</h4>
			<select bind:value={selectedLectureNum} on:change="{() => getSelectedData()}">
				{#each filenames as filename, idx}
					<option value="{idx}">{filename.split("/")[1]}</option>
				{/each}
			</select>

			<svg style="height: {sliceBarChartHeight + 60}; padding: 20px">
				<g id="slice-bar-chart" transform="translate(10,10)">
					<g class="chart-axes">
						<line y1="0" y2="{sliceBarChartHeight}" stroke="black" stroke-width="1"/>

						{#each sliceYScaleTicks as tick}
							<g transform="translate(0, {sliceYScale(tick)})">
								<line x1="0" x2="5" stroke="black" stroke-width="0.5" />
								<text x="10" y="6">{tick}</text>
							</g>
						
						{/each}
					</g>
					
					<g id="slice-bar-chart-bars" transform="translate(50,0)">
						<g class="slice-bar-chart-item">
							{#if selectedData && lectures}
								<rect 
									x="0"
									y="{sliceYScale(selectedData.length / lectures[selectedLectureNum].data.length)}"
									height="{sliceBarChartHeight - sliceYScale(selectedData.length / lectures[selectedLectureNum].data.length)}"
									width="{sliceBarWidth}"
									style="fill:black;"
								/>	
							{/if}
							<text x="0" y="{sliceBarChartHeight + 20}">% Time</text>
						</g>

						<g class="slice-bar-chart-item" transform="translate(80,0)">
							{#if selectedData && lectures}
								<rect 
									x="0"
									y="{sliceYScale(percentDistance)}"
									height="{sliceBarChartHeight - sliceYScale(percentDistance)}"
									width="{sliceBarWidth}"
									style="fill:black;"
								/>	
							{/if}
							<text x="0" y="{sliceBarChartHeight + 20}">% Distance</text>
						</g>
					</g>
				</g>
			</svg>
		</div>

			<!--

		<div id="heatmap" style="height: 100vh; width:100vw;">
			<svg style="height: 100%; width: 100%;">
				<circle 
					id="classroom-outline"
					cx="300"
					cy="300"
					r="200"
					stroke="black"
					stroke-width="10"
					style="fill:none;"
					
				/>
				<circle
					r="100"
					cx="300"
					cy="300"
					fill="transparent"
					stroke="tomato"
					stroke-width="200"
					stroke-dasharray="150 3"
				/>
			</svg>
		</div>
-->
	</div>
</main>

<style>
	h1 {
		font-size: 1.3rem;
		font-weight: 500;
		margin-top: 0;
	}
	#container {
		display: flex;
	}

	#slice-view {
		display: flex;
		flex-direction: column;
	}

	.chart-axes {
		font-size: 0.8em;
		color: gray;
	}

	svg {
		margin: 5px;
	}
</style>
