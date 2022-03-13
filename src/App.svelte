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
	//import SliceBarCharts from "./components/SliceBarCharts.svelte"
	
	let lectures = [];
	let instructors =  new Set()
	
	let selectedData;
	let selectedSection;
	
	let selectedLectureNumbers;
	let selectedInstructors;

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
			return "A"
		else if (filename.includes("instructorB")) 
			return "B"
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

			instructors.add(instructor_name);
		};
		instructors = Array.from(instructors)
		console.log("Lectures: ", lectures)

		

		//start out with all instructors and lectures selected
		selectedData = []
		selectedInstructors = instructors
		selectedLectureNumbers=[0,1,2,3]
		
	});

	//return subset of lectures that only contains points within coordinate bounds
	function filterSelectedData(xmin, xmax, ymin, ymax) {
		let data = []
		selectedLectureNumbers.forEach((num) => {
			if (selectedInstructors.includes(lectures[num].instructor_name)) {
				data.push({
					instructor_name: lectures[num].instructor_name,
					lecture_num: num,
					data: [...lectures[num].data.filter((e) => {
					return e["x [pixel]"] > xmin 
						&&  e["x [pixel]"] < xmax 
						&& e["y [pixel]"] > ymin 
						&& e["y [pixel]"] < ymax
					})]
				})
			}
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

	//When data is filtered (i.e. quadrant, lecture, or instructor checked or unchecked)
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
	}


	function instructorClick() {
		//remove lectures that are taught by unselected instructors
		selectedLectureNumbers = lectures.reduce((arr, e, i) => (selectedInstructors.includes(e.instructor_name) && e.copus === false && arr.push(i), arr), [])
		getSelectedData()
	}
</script>	

<main>
	<h1>Visual Analytics Final Project</h1>
	<h2>Instructor Movement Data Visualization</h2>
	

	

<div id="container">
		<div id="video-view" class="view-panel">

			<div class="view-title">Instructor Path Player</div>

			<div class="view-anim">
				<div id="instructor-select">
					<p id="instructor-select-title">Instructors
						<i class="fas fa-angle-down"id="i-dropdown"></i>
					</p>
					<p id="instructor-select-id">{selectedInstructors}</p>
					<div class="dropdown">
						<div class="dropdown-content" id="dropdown-content-instructor">
							{#if selectedLectureNumbers !== undefined && filenames !== undefined && lectures !== undefined} 
								<div class="select-options"> 
									{#each instructors as instructor}
										
										<label>
											<input type="checkbox" value={instructor}
												bind:group={selectedInstructors}
												on:change="{() => instructorClick()}">
												{instructor}
										</label>
									{/each}
								</div>
							{/if}
						</div>
					</div>
				</div>
				<div id="instructor-activity">
					<p id="instructor-activity-word">LECTURING</p>
				</div>
					<Heatmap 
					lectures={lectures}
					bind:selectedSection={selectedSection}
					bind:selectedLectureNumbers={selectedLectureNumbers}
					getSelectedData={getSelectedData}
					heatmapRange={dataRange} />
			</div>

			

			<div class="view-underbar">
				<div id="lecture-select-button">
					<p id="lecture-select-title">Lectures</p>
					<i class="fas fa-angle-down"id="i-dropdown"></i>
					<div class="dropdown">
						<div class="dropdown-content">
							<!--<h4 id="lecture-select-title">Select a Lecture</h4>-->
							{#if selectedLectureNumbers !== undefined && filenames !== undefined && lectures !== undefined} 
								<div class="select-options"> 
									{#each filenames as filename, idx}
										{#if lectures[idx].copus === false && selectedInstructors.includes(lectures[idx].instructor_name)}
											<label>
												<input type="checkbox" value={idx}
													bind:group={selectedLectureNumbers}
													on:change="{() => getSelectedData()}">
													{filename.split("/")[1]}
											</label>
										{/if}
									{/each}
								</div>
							{/if}
						</div>
					</div>
				</div>
				<div class="view-icons">
					<i class="fas fa-fast-backward fas2"id="i-back"></i>
					<i class="fas fa-step-backward fas2"id="i-backstep"></i>
					<i class="fas fa-play fas2"id="i-play"></i>
					<i class="fas fa-step-forward fas2"id="i-forwardstep"></i>
					<i class="fas fa-fast-forward fas2"id="i-forward"></i>
				</div>
			</div>
		</div>
		<div id="chart-view" class="view-panel">
			<div class="view-title">Instructor Statistics</div>
			<div id="view-list">
				<!--<p id="stat0">Instructor X spent the most time at location: LOC</p>
				<p id="stat1">Instructor X spent the most time doing this activity: ACT</p>
				<p id="stat2">Instructor X traveled this maximum distance away from the original location: DIST</p>
				<p id="stat3">Continues...</p>-->
				<h4 id="slice-select-title">Select a Quarter of Room</h4>
				<select bind:value={selectedSection} on:change="{() => getSelectedData()}">
					<option value="N/A"></option>
					<option value="Q1">Q1</option>
					<option value="Q2">Q2</option>
					<option value="Q3">Q3</option>
					<option value="Q4">Q4</option>
				</select>

				<PerQuarterBarChart
					lectures={lectures}
					selectedData={selectedData} />

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
	BUTTON BKG 2 (gray-blue): #A9B3C6;

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
		font-size: 35px;
		font-weight: 500;
		margin-top: 0;
		width:100%;
		text-align: center;
		margin:5px;
	}
	h2
	{
		font-size: 30px;
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
		justify-content:center;
	}
	.view-panel {
		border: 2px solid #D4D3D9;
		margin:15px;
		position:relative;
		height:600px;
	}
	.view-title {
		background-color: #B6B3BD;
		font-size: 25px;
		margin-bottom: 8px;
		padding: 3px 8px 5px 12px;
	}
	.view-anim
	{
		position:relative;
		padding:5px;
		height:300px;
	}
	#video-view
	{
		width:710px;
		height:440px;
	}

/*
	VIDEO VIEW LECTURE SELECTOR + BUTTON	
*/
	#lecture-select-button
	{
		background-color: #a9b3c6;
		border: 2px solid #D4D3D9;
		border-radius: 15px;
		position: absolute;
		left: 5px;
		bottom: 3px;
		padding: 5px;
		display: flex;
	}
	#lecture-select-button:hover .dropdown .dropdown-content  {display: block;}

	#lecture-select-button p
	{
		text-align: center;
		font-weight: 500;
		font-size: 15px;
	}
	#i-dropdown
	{
		padding: 3px;
		padding-top:5px;
    	margin: auto;
	}
/*
	VIDEO VIEW SELECTED INSTRUCTOR
*/
	#instructor-select
	{
		background-color:#A9B3C6;
		border: 2px solid #D4D3D9;
		border-radius: 15px;
		width:140px;
		height:70px;
		position:absolute;
		margin:5px;
		right:20px;
		padding-top:5px;
		z-index:2;
	}
	#instructor-select:hover .dropdown .dropdown-content  {display: block;}

	#dropdown-content-instructor
	{
		position: absolute;
		top: -10px;
		left: -5px;
		width: 20px;
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
		background-color: #B5BDB3;
		border: 2px solid #D4D3D9;
		border-radius: 15px;
		width: 100px;
		height: 70px;
		position: absolute;
		margin: 5px;
		right: 180px;
		padding-top: 2.5px;
		display: flex;
		padding-bottom: 2.5px;
		z-index:2;
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
		width:600px;
		height:600px;
	}
	.fas2
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

/*
	LECTURE SELECT OPTIONS
*/

	.dropdown
	{
		position: relative;
  		display: inline-block;
	}

	.dropdown-content 
	{
		display: none;
		position: absolute;
		left: -80px;
		top: 55px;
		background-color: #A9B3C6;
		min-width: 160px;
		box-shadow: 0px 8px 16px 0px rgb(0 0 0 / 20%);
		z-index: 1;
		border: 2px solid #D4D3D9;
		border-radius: 15px;
		padding:5px;
	}

	.dropdown:hover .dropdown-content {display: block;}

	.select-options {
		display: flex;
		font-size: 15px;
		flex-direction: column;
	}


</style>
