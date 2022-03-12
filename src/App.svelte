<!-- 
	Final Project Svelte App
-->

<svelte:head>
	<script src="https://kit.fontawesome.com/a076d05399.js" crossorigin="anonymous"></script>
</svelte:head>

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
	<h1>Visual Analytics Final Project</h1>
	<h2>Instructor Movement Data Visualization</h2>
	<select bind:value={selectedSection} on:change="{() => getSelectedData()}">
		<option value="N/A"></option>
		<option value="Q1">Q1</option>
		<option value="Q2">Q2</option>
		<option value="Q3">Q3</option>
		<option value="Q4">Q4</option>
	</select>
	<h4 id="lecture-select-title">Select a Lecture</h4>
	{#if selectedLectureNumbers !== undefined && filenames !== undefined && lectures !== undefined} 
		{#each filenames as filename, idx}
			{#if lectures[idx].copus === false}
				<label>
					<input type="checkbox" value={idx}
						bind:group={selectedLectureNumbers}
						on:change="{() => getSelectedData()}">
						{filename.split("/")[1]}
				</label>
			{/if}
		{/each}
	{/if}

<div id="container">
		<div id="video-view" class="view-panel">
			<div class="view-title">Instructor Path Player</div>

			<div class="view-anim">
				<div id="instructor-select">
					<p id="instructor-select-title">Selected Instructor:</p>
					<p id="instructor-select-id">A</p>
				</div>
				<div id="instructor-activity">
					<p id="instructor-activity-word">LECTURING</p>
				</div>
			</div>

			<Heatmap 
				lectures={lectures}
				bind:selectedSection={selectedSection}
				bind:selectedLectureNumbers={selectedLectureNumbers}
				getSelectedData={getSelectedData}
				heatmapRange={dataRange} />

			<div class="view-underbar">
				<div class="view-icons">
					<i class="fas fa-fast-backward"id="i-back"></i>
					<i class="fas fa-step-backward"id="i-backstep"></i>
					<i class="fas fa-play"id="i-play"></i>
					<i class="fas fa-step-forward"id="i-forwardstep"></i>
					<i class="fas fa-fast-forward"id="i-forward"></i>
				</div>
			</div>
		</div>
		<div id="chart-view" class="view-panel">
			<div class="view-title">Instructor Statistics</div>
			<div id="view-list">
				<p id="stat0">Instructor X spent the most time at location: LOC</p>
				<p id="stat1">Instructor X spent the most time doing this activity: ACT</p>
				<p id="stat2">Instructor X traveled this maximum distance away from the original location: DIST</p>
				<!--<p id="stat3">Continues...</p>-->

				<PerQuarterBarChart
					lectures={lectures}
					selectedData={selectedData}
					selectedLectureNumbers={selectedLectureNumbers} />

			</div>
		</div>
	<!--
	<SliceBarCharts
		lectures={lectures}
		selectedData={selectedData}
		selectedSection={selectedSection}
		selecetdLectureNumbers={selectedLectureNumbers} />
	-->
	</div>

</main>

<style>

/*
	COLORS:

	TITLE BKG (gray-purple): #B6B3BD;
	BUTTON BKG (gray-green): #B5BDB3;
	BUTTON NKG 2 (gray-blue): #A9B3C6;

	LIGHT BORDER: #D4D3D9;
	DARK BORDER: #615f6d;
*/

/*
	MAIN DOCUMENT
*/
	:global(body)
	{
		background-color: #EAE9EC;
	}
	h1 {
		font-size: 2rem;
		font-weight: 500;
		margin-top: 0;
		width:100%;
		text-align: center;
		margin:5px;
	}
	h2
	{
		font-size: 1.2rem;
		font-weight: 500;
		width:100%;
		text-align: center;
		margin:2px;
	}

/*
	VIDEO VIEW MAIN
*/
	#container {
		display: flex;
		width:auto;
	}
	.view-panel {
		border: 2px solid #D4D3D9;
		margin:15px;
		position:relative;
		height:600px;
	}
	.view-title {
		background-color: #B6B3BD;
		font-size: 1.2rem;
		margin-bottom: 8px;
		padding: 3px 8px 5px 12px;
	}
	.view-anim
	{
		position:relative;
		padding:5px;
	}
	#video-view
	{
		width:70%;
	}

/*
	VIDEO VIEW SELECTED INSTRUCTOR
*/
	#instructor-select
	{
		background-color:#B5BDB3;
		border: 2px solid #D4D3D9;
		border-radius: 15px;
		width:140px;
		height:70px;
		position:absolute;
		margin:0;
		right:20px;
		padding-top:5px;
	}
	#instructor-select-title
	{
		margin:0;
		padding:0;
		font-size: 15px;
		border-bottom:2px solid #615f6d;
	}
	#instructor-select p
	{
		text-align: center;
		font-weight: 500;

	}
	#instructor-select-id
	{
		font-size: 30px;
		margin-top: 5px;
		position:relative;
		margin:0;
	}

/* 
	VIDEO VIEW INSTRUCTOR ACTIVITY
*/
	#instructor-activity
	{
		background-color:#a9b3c6;
		border: 2px solid #D4D3D9;
		border-radius: 15px;
		width:100px;
		height:70px;
		position:absolute;
		margin:0;
		right:180px;
		padding-top:0;
		display:flex;
	}
	#instructor-activity-word
	{
		text-align: center;
		font-weight: 500;
		position:relative;
		margin:0;
		display:flex;
		width:100%;
		align-items:center;
		justify-content:center;
		font-size:17px;
	}

/*
	VIDEO VIEW UNDERBAR
*/
	.view-underbar
	{
		position:absolute;
		bottom:0;
		width:100%;
		background-color:#B6B3BD;
	}
	.view-icons
	{
		display:flex;
		justify-content:center;
		padding:5px;
	}

/* 
	CHART VIEW MAIN
*/
	#chart-view
	{
		width:50%;
	}
	.fas
	{
		font-size:50px;
		margin-top:5px;
		margin-bottom:5px;
		margin-left:15px;
		margin-right:15x;
	}
	#view-list
	{
		padding:10px;
		padding-top:2px;

	}
	#view-list p
	{
		margin:15px;
		text-align: left;
		display:block;
	}



</style>
