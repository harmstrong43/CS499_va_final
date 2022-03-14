<!-- 
	Final Project Svelte App
-->

<svelte:head>
	<script src="https://kit.fontawesome.com/a076d05399.js" crossorigin="anonymous"></script>
	<script src="https://d3js.org/d3.v4.min.js"></script>
</svelte:head>
<script>
	import { onMount } from "svelte";
	import { scaleLinear, scaleOrdinal } from "d3-scale";
	import { schemeCategory10 } from "d3-scale-chromatic";
	import { toggle_class } from "svelte/internal";

	import Heatmap from "./components/Heatmap.svelte"
	import PerQuarterBarChart from "./components/PerQuarterBarChart.svelte"
	//import Timeline from "./components/Timeline.svelte"
	//import SliceBarCharts from "./components/SliceBarCharts.svelte"
	import Lenpixel from "./components/Lenpixel.svelte";
	import Distance from "./components/Distance.svelte";
	import Copusbars from "./components/Copusbars.svelte";
	//import SliceBarCharts from "./components/SliceBarCharts.svelte"	
	
	let lectures = [];
	let instructorNumbers = [0,1,2,3];
	let instructorNames = ["Instructor A", "Instructor B (Lecture 1)", "Instructor B (Lecture 2)", "Instructor B (Lecture 3)"];
	let pixelID;
	let lenPixel;
	let distancePixel;
	let colorScale;
	let lectureActivity;
	let questionAskingActivity;
	let questionAnsweringActivity;
	let otherActivity;
	let count = 0;
	let copusActivity = [];
	let lenLines = [];
	let distLines = [];
	let currentIndex;
	let legendLabels = [];
	let clicked = 0;
	let hovered = false;
	let bars = {
        lecture: [],
        asking: [],
        answering: [],
        other: []
    };


	const copusInterval = 120;

	let instructors =  new Set()
	
	let selectedData =[];

	//quarters of classroom selected
	let selectedSections = new Set();
	const allSections = [0,1,2,3]
	
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
		"static/COPUS_instructorB.json"
	]
	
	const otherCopus = [
		"Instructor-1o1",
		"Instructor-Adm",
		"Instructor-DV",
		"Instructor-FUp",
		"Instructor-MG",
		"Instructor-Other",
		"Instructor-RtW",
		"Instructor-W",
		"Instructor-_Si"
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

	function sum(arr) {
		return arr.reduce(function (a, b) {
			return a + b;
		}, 0);
	}

	//Returns values for whether the instructor is Lecturing, Asking questions,
	// or answering (0 or 1) for a given instructor and timestamp in the movement data
	function getCopusActivity(instructor_name, index) {
		//get relevant lecture object
		let copusForm = lectures.filter(lecture => lecture.copus === true && lecture.instructor_name === instructor_name)[0].data;
		//Set default return value to 0 (no activity)
		let outputActivity = [0, 0, 0, 0];
		//Get a COPUS object with the information for variables that go in the "Other" category
		var subsetObj = otherCopus
		.reduce(function (obj2, key) {
			if (key in copusForm[index])
				obj2[key] = copusForm[index][key];
			return obj2;
		}, {});
		//Determine what instructor is doing at given time (index)
		if (copusForm[index]['Instructor-Lec'] === 1) {
			outputActivity[0] += 1;
		}
		if (copusForm[index]['Instructor-PQ'] === 1 || copusForm[index]['Instructor-CQ'] === 1) {
			outputActivity[1] += 1;
		}
		if (copusForm[index]['Instructor-AnQ'] === 1) {
			outputActivity[2] += 1;
		}
		else if (sum(Object.values(subsetObj)) >= 1) {
			outputActivity[3] += 1;
		}
		return outputActivity;
	}
	
	//Function to change the opacity of each instructor's portion of the chart when highlighted
	function highlightCharts(index, lenOpacity, distOpacity, barOpacity){
		lenLines[index].style.opacity = lenOpacity;
		bars.lecture[index].style.opacity = barOpacity;
		bars.asking[index].style.opacity = barOpacity;
		bars.answering[index].style.opacity = barOpacity;
		bars.other[index].style.opacity = barOpacity;
		distLines[index].style.opacity = distOpacity;
	}

	$:{
		if(clicked > 0 && legendLabels !== undefined){
			if(legendLabels.filter(x => x.classList.contains("unclicked") === false).length > 0) {
				//make unselected ones lighter
				legendLabels.filter(x => x.classList.contains("unclicked")).forEach(y => {
					highlightCharts(legendLabels.indexOf(y), 0.2, 0.2, 0.2);
				})
				//highlight selected ones
				legendLabels.filter(x => x.classList.contains("unclicked") === false).forEach(y => {
					highlightCharts(legendLabels.indexOf(y), 0.8, 0.6, 1);
				})
			}
			else {
				//make them all stand out if none are selected
				legendLabels.forEach(y => {
					highlightCharts(legendLabels.indexOf(y), 0.8, 0.6, 1);
				})
			}
		}
	}

	$: {
		if(lenLines !== undefined && selectedLectureNumbers !== undefined) {
			if(hovered === true){
				//highlight hovered instructor
				highlightCharts(currentIndex, 0.8, 0.6, 1);
				//grey out ones that aren't clicked or hovered
				legendLabels.filter(x => x.classList.contains("unclicked")).forEach(y => {
					let index = legendLabels.indexOf(y);
					if(index !== currentIndex){
						highlightCharts(index, 0.2, 0.2, 0.2);
					}
			})
			}
			else {
				//if at least one is clicked
				console.log("Legend: ", legendLabels);
				if(legendLabels.filter(x => x.classList.contains("unclicked") === false).length > 0) {
					legendLabels.filter(x => x.classList.contains("unclicked")).forEach(y => {
					//make ones that aren't clicked lighter
					highlightCharts(legendLabels.indexOf(y), 0.2, 0.2, 0.2)
					})
				}
				//if nothing is clicked
				else if (legendLabels.filter(x => x.classList.contains("unclicked")).length >= selectedLectureNumbers.length){
					//Highlight Everthing
					legendLabels.filter(x => x.classList.contains("unclicked")).forEach(y => {
						highlightCharts(legendLabels.indexOf(y), 0.8, 0.6, 1);
					})
				}
			}
		}
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

		//Loop through non-COPUS objects, adding relevant COPUS actions
		//Count gives the frame number
		lectures.filter(lecture => lecture.copus === false).forEach(lecture => {
			lecture.data.forEach(frame => {
				copusActivity = getCopusActivity(lecture.instructor_name, Math.floor(count / copusInterval));
				frame['lecturing'] = copusActivity[0];
				frame['asking_questions'] = copusActivity[1];
				frame['answering_questions'] = copusActivity[2];
				frame['other'] = copusActivity[3];
				count += 1;
			});
			//After looping through a, reset count to 0.
			//The count will not reset after each lecture in B
			//Note this will not work if the data are different
			if (lecture.instructor_name === "a"){
				count = 0;
			}
		});

		//Declare pixelID, lenPixel, and distancePixel (for readability)
		pixelID = lectures.filter(lecture => lecture.copus === false).map(movement => movement['data'].map(rows => rows['PID']));
		console.log("PixelID: ", pixelID);
		lenPixel = lectures.filter(lecture => lecture.copus === false).map(movement => movement['data'].map(rows => rows['Len [pixel]']));
		distancePixel = lectures.filter(lecture => lecture.copus === false).map(movement => movement['data'].map(rows => rows['D2P [pixel]']));

		console.log("Lectures: ", lectures);

		//store activity types as their own variables
		lectureActivity = lectures.filter(lecture => lecture.copus === false).map(x => x['data'].map(lecturing => lecturing['lecturing']));
		questionAskingActivity = lectures.filter(lecture => lecture.copus === false).map(x => x['data'].map(questions => questions['asking_questions']));
		questionAnsweringActivity = lectures.filter(lecture => lecture.copus === false).map(x => x['data'].map(answers => answers['answering_questions']));
		otherActivity = lectures.filter(lecture => lecture.copus === false).map(x => x['data'].map(answers => answers['other']));
		instructors = Array.from(instructors)
		console.log("Lectures: ", lectures)
		console.log("Instructors: ", instructors);

		

		//start out with all instructors, quarters, and lectures selected
		selectedInstructors = instructors
		selectedLectureNumbers=[0,1,2,3]

		//add all sections to selectedSections
		allSections.forEach(i => selectedSections.add(i)) 


		//all non-copus data selected to start
		let temp_data = []
		lectures.forEach((lect, idx) => {
			if (lect.copus === false) {
				temp_data.push({
					lecture_num: idx,
					data: lect.data,
					instructor_name: lect.instructor_name,
					copus: lect.copus
				})
			}
		})

		selectedData = temp_data

		colorScale = scaleOrdinal(schemeCategory10)
		.domain(selectedLectureNumbers);
		
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
		//console.log("Selected lecture:", selectedLectureNumbers, "Selected sections:", selectedSections)
		let starting_point = setStartingPoint()
	
		selectedData = []

		//console.log("Starting Point:", starting_point)
		if (selectedSections.has(0))
			selectedData.push(...filterSelectedData(starting_point.x, Infinity, starting_point.y, Infinity))
		if (selectedSections.has(1))
			selectedData.push(...filterSelectedData(starting_point.x, Infinity, Number.NEGATIVE_INFINITY, starting_point.y))
		if (selectedSections.has(2))
			selectedData.push(...filterSelectedData(Number.NEGATIVE_INFINITY, starting_point.x, Number.NEGATIVE_INFINITY, starting_point.y))
		if (selectedSections.has(3))
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
					{lectures}
					bind:selectedSections = {selectedSections}
					bind:selectedLectureNumbers = {selectedLectureNumbers}
					getSelectedData = {getSelectedData}
					heatmapRange = {dataRange} />
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
				
			</div>

			<!--<Timeline {lectures} {selectedLectureNumbers} range={dataRange} />-->


		</div>
		<div id="chart-view" class="view-panel">
			<div class="view-title">Instructor Statistics</div>
			<div id="view-list">
				<!--<p id="stat0">Instructor X spent the most time at location: LOC</p>
				<p id="stat1">Instructor X spent the most time doing this activity: ACT</p>
				<p id="stat2">Instructor X traveled this maximum distance away from the original location: DIST</p>
				<p id="stat3">Continues...</p>-->
				
				<!--
				<h4 id="slice-select-title">Select a Quarter of Room</h4>
				<select multiple bind:value={selectedSections} on:change="{() => getSelectedData()}">
					
					<option value=0>Q1</option>
					<option value=1>Q2</option>
					<option value=2>Q3</option>
					<option value=3>Q4</option>
				</select>
				-->

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
		selectedLectureNumbers={selectedLectureNumbers} />
	-->
	</div>

	<div id="container">
		<div id="sidebar" style="width: 450px;">
			<div id="len-pixel" class="view-panel">
				<div class="view-title">Pixel ID vs Len[Pixel]</div>
				{#if pixelID !== undefined}
					<Lenpixel lectures = {lectures} colorScale = {colorScale} 
					lenPixel = {lenPixel} pixelID = {pixelID} selectedLectureNumbers = {selectedLectureNumbers}
					bind:lines={lenLines} bind:currentIndex={currentIndex} bind:hovered={hovered}/>
				{/if}
			</div>
		</div>
		<div id="middle" style="width: 450px">
			<div id="Distance-to-current-point" class="view-panel">
				<div class="view-title">Distance to Current Point</div>
				{#if pixelID !== undefined}
					<Distance lectures = {lectures} colorScale = {colorScale} 
					pixelID = {pixelID} distancePixel = {distancePixel} selectedLectureNumbers={selectedLectureNumbers}
					bind:distLines={distLines} bind:currentIndex={currentIndex} bind:hovered={hovered}/>
				{/if}
			</div>
		</div>

		<div id="main-section" style="width: 450px;">
			<div id="COPUS-bars" class="view-panel">
				<div class="view-title">COPUS Data</div>
				{#if lectureActivity !== undefined}
					<Copusbars colorScale = {colorScale} selectedData={selectedData}
					 lectureActivity = {lectureActivity} questionAnsweringActivity = {questionAnsweringActivity}
					 questionAskingActivity = {questionAskingActivity} otherActivity = {otherActivity} selectedLectureNumbers={selectedLectureNumbers}
					 bind:bars={bars} bind:hovered={hovered} bind:currentIndex={currentIndex} />
				{/if}
			</div>
		</div>
		<div id  = "right-side" style="width: 180px;">
			<div id = "legend-frame" class="view-panel">
				<div class = "view-title">Legend</div>
					<svg id = "legend" width="175px" height = "80px">
						<g id = "legend-lines" transform = "translate(10, 15)">
							{#if colorScale !== undefined}
								{#each instructorNumbers as instructor}
								{console.log("included instructor: ", selectedLectureNumbers.includes(instructor))}
									<line id="legend-instructor-{instructor}"
									class = "legend-line" style = "{selectedLectureNumbers.includes(instructor) ? "stroke: " + colorScale(instructor) : "display: none"}"
									x2 = "20" y1 = {(60 / instructorNumbers.length) * instructor} y2 = {(60 / instructorNumbers.length) * instructor} />
									<text 
									class="legend-labels unclicked" 
									style = "{selectedLectureNumbers.includes(instructor) ? "" : "display: none"}"
									bind:this={legendLabels[instructor]} 
									x = "25" y= {(60 / instructorNumbers.length) * instructor + 3}
										on:click={() => {
												legendLabels[instructor].classList.toggle("unclicked")
												currentIndex = instructor;
												clicked += 1;}}
									>
										{instructorNames[instructor]}</text>
								{/each}
							{/if}
						</g>
					</svg>
			</div>

		</div>
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
	#sidebar, #main-section {
		display: flex;
		flex-direction: column;
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
	line.legend-line {
		stroke-width: 4;
	}
	text.legend-labels {
		text-decoration: underline;
		font-size: 12px;
		fill: black;
	}
	text.unclicked {
		text-decoration: none;
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
		margin: 9px 0;
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
