<script>
	import { onMount } from "svelte";
	import { scaleLinear, scaleOrdinal } from "d3-scale";
	import { schemeCategory10 } from "d3-scale-chromatic";
	import Lenpixel from "./Lenpixel.svelte";
	import Distance from "./Distance.svelte";
	import Copusbars from "./Copusbars.svelte";
	import { toggle_class } from "svelte/internal";

	let lectures = [];
	let instructors = [0, 1, 2, 3];
	let instructor_names = ["Instructor A", "Instructor B (Lecture 1)", "Instructor B (Lecture 2)", "Instructor B (Lecture 3)"];
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
	let type;
	let hovered = false;
	let bars = {
        lecture: [],
        asking: [],
        answering: [],
        other: []
    };


	const copusInterval = 120;


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
			return "a"
		else if (filename.includes("instructorB")) 
			return "b"
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
		if(clicked > 0 && legendLabels != undefined){
			if(legendLabels.filter(x => x.classList.contains("unclicked")).length !== instructors.length) {
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
		if(lenLines !== undefined) {
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
				if(legendLabels.filter(x => x.classList.contains("unclicked")).length !== instructors.length) {
					legendLabels.filter(x => x.classList.contains("unclicked")).forEach(y => {
					//make ones that aren't clicked lighter
					highlightCharts(legendLabels.indexOf(y), 0.2, 0.2, 0.2)
					})
				}
				//if nothing is clicked
				else if (legendLabels.filter(x => x.classList.contains("unclicked")).length === instructors.length){
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
		lenPixel = lectures.filter(lecture => lecture.copus === false).map(movement => movement['data'].map(rows => rows['Len [pixel]']));
		distancePixel = lectures.filter(lecture => lecture.copus === false).map(movement => movement['data'].map(rows => rows['D2P [pixel]']));

		console.log("Lectures: ", lectures);

		colorScale = scaleOrdinal(schemeCategory10)
		.domain(instructors);

		//store activity types as their own variables
		lectureActivity = lectures.filter(lecture => lecture.copus === false).map(x => x['data'].map(lecturing => lecturing['lecturing']));
		questionAskingActivity = lectures.filter(lecture => lecture.copus === false).map(x => x['data'].map(questions => questions['asking_questions']));
		questionAnsweringActivity = lectures.filter(lecture => lecture.copus === false).map(x => x['data'].map(answers => answers['answering_questions']));
		otherActivity = lectures.filter(lecture => lecture.copus === false).map(x => x['data'].map(answers => answers['other']));
		
	});

	

</script>

<main>
	<h1>Visual Analytics Final Project - Dan's Line Charts</h1>

	<div id="container">
		<div id="sidebar" style="width: 450px;">
			<div id="len-pixel" class="view-panel">
				<div class="view-title">Pixel ID vs Len[Pixel]</div>
				{#if pixelID !== undefined}
					<Lenpixel lectures = {lectures} colorScale = {colorScale} 
					lenPixel = {lenPixel} pixelID = {pixelID}
					bind:lines={lenLines} bind:currentIndex={currentIndex} bind:hovered={hovered}/>
				{/if}
			</div>
		</div>
		<div id="middle" style="width: 450px">
			<div id="Distance-to-current-point" class="view-panel">
				<div class="view-title">Distance to Current Point</div>
				{#if pixelID !== undefined}
					<Distance lectures = {lectures} colorScale = {colorScale} pixelID = {pixelID} distancePixel = {distancePixel}
					bind:distLines={distLines} bind:currentIndex={currentIndex} bind:hovered={hovered}/>
				{/if}
			</div>
		</div>

		<div id="main-section" style="width: 450px;">
			<div id="COPUS-bars" class="view-panel">
				<div class="view-title">COPUS Data</div>
				{#if lectureActivity !== undefined}
					<Copusbars colorScale = {colorScale}
					 lectureActivity = {lectureActivity} questionAnsweringActivity = {questionAnsweringActivity}
					 questionAskingActivity = {questionAskingActivity} otherActivity = {otherActivity} 
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
								{#each instructors as instructor}
									<line id="legend-instructor-{instructor}"
									class = "legend-line" style = "stroke: {colorScale(instructor)}"
									x2 = "20" y1 = {(60 / instructors.length) * instructor} y2 = {(60 / instructors.length) * instructor} />
									<text class="legend-labels unclicked" bind:this={legendLabels[instructor]} x = "25" y= {(60 / instructors.length) * instructor + 3}
										on:click={() => {
												legendLabels[instructor].classList.toggle("unclicked")
												currentIndex = instructor;
												clicked += 1;
												type = "All";}}
									>
										{instructor_names[instructor]}</text>
								{/each}
							{/if}
						</g>
					</svg>
			</div>

		</div>
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
	#sidebar, #main-section {
		display: flex;
		flex-direction: column;
	}
	.view-panel {
		border: 2px solid #eee;
		margin-bottom: 15px;
		margin-right: 15px;
	}
	.view-title {
		background-color: #f3f3f3;
		font-size: 1.0rem;
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


</style>
