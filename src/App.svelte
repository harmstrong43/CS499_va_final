<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->
<script>
	import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale";

	import Heatmap from "./components/Heatmap.svelte"
	import PerQuarterBarChart from "./components/PerQuarterBarChart.svelte"
	import SliceBarCharts from "./components/SliceBarCharts.svelte"
	
	let lectures = [];
	
	let selectedData;
	let selectedSection;
	
	let lectureSelectVal;
	let selectedLectureNumbers;

	let timelineXScale
	let timelineYScale

	const dataRange = {
		x: [0,1919],
		y: [0,800]
	}

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

		

		//start out with all data selected
		selectedData = []
		selectedLectureNumbers=[0,1,2,3]
		lectures.forEach((lecture) => selectedData.push(lecture.data))
		
	});

	function filterSelectedData(xmin, xmax, ymin, ymax) {
		let data = []
		selectedLectureNumbers.forEach((num) => {
			data.push(...lectures[num].data.filter((e) => {
				return e["x [pixel]"] > xmin 
					&&  e["x [pixel]"] < xmax 
					&& e["y [pixel]"] > ymin 
					&& e["y [pixel]"] < ymax
			}))
		})		
		return data
		//getTotalDistance()
	}

	function setStartingPoint(){
		return {
			x: dataRange.x[1]/2,
			y: dataRange.y[1]/2
		}
	}

	function getSelectedData(){
		//console.log("Selected lecture:", selectedLectureNumbers, "Selected section:", selectedSection)
		let starting_point = setStartingPoint()

		selectedData = []
		
		//console.log("Starting Point:", starting_point)
		if (selectedSection =="Q1")
			selectedData.push(...filterSelectedData(starting_point.x, Infinity, starting_point.y, Infinity))
		if (selectedSection == "Q2")
			selectedData.push(...filterSelectedData(starting_point.x, Infinity, Number.NEGATIVE_INFINITY, starting_point.y))
		if (selectedSection == "Q3")
			selectedData.push(...filterSelectedData(Number.NEGATIVE_INFINITY, starting_point.x, Number.NEGATIVE_INFINITY, starting_point.y))
		if (selectedSection == "Q4")
			selectedData.push(...filterSelectedData(Number.NEGATIVE_INFINITY, starting_point.x, starting_point.y, Infinity))
		
		//console.log(selectedData)		
	}
</script>

<main>
	<h1>Section of Classroom View</h1>
	<select bind:value={selectedSection} on:change="{() => getSelectedData()}">
        <option value="Q1">Q1</option>
        <option value="Q2">Q2</option>
        <option value="Q3">Q3</option>
        <option value="Q4">Q4</option>
    </select>
    <h4 id="lecture-select-title">Select a Lecture</h4>

	
		
	{#if selectedLectureNumbers !== undefined && filenames !== undefined} 
		{#each filenames as filename, idx}
			<label>
					<input type="checkbox" value={idx}
						bind:group={selectedLectureNumbers}
						on:change="{() => getSelectedData()}"
						>
						{filename.split("/")[1]}
			</label>
		{/each}
	
	{/if}


<div id="container">

	<!--
	<SliceBarCharts
		lectures={lectures}
		selectedData={selectedData}
		selectedSection={selectedSection}
		selecetdLectureNumbers={selectedLectureNumbers} />
	-->
	<Heatmap 
		lectures={lectures}
		bind:selectedSection={selectedSection}
		bind:selectedLectureNumbers={selectedLectureNumbers}
		getSelectedData={getSelectedData}
		heatmapRange={dataRange} />

	<PerQuarterBarChart
		lectures={lectures}
		selectedData={selectedData}
		selectedLectureNumbers={selectedLectureNumbers} />

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
</style>
