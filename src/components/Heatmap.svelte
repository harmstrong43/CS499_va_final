<script>
    import { onMount } from "svelte";
	import { scaleLinear } from "d3-scale"; 

    export let lectures;
    export let selectedSection = undefined;
    export let selectedLectureNumbers = undefined;
    export var getSelectedData; 
	export let heatmapRange;
    
    let heatmapXScale
    let heatmapYScale 

    let heatmapWidth;
	let heatmapHeight;


    onMount(async () => {
        heatmapXScale = scaleLinear()
			.domain(heatmapRange.x)
			//.domain([-1000,2000])
			.range([0,heatmapWidth])
		
		heatmapYScale = scaleLinear()
			.domain(heatmapRange.y)
			//.domain([heatmapStartingPoint.y-1000,heatmapStartingPoint.y + 1000])
			.range([0,heatmapHeight])
    });

</script>
	<div style="height:80%;" bind:clientWidth={heatmapWidth} bind:clientHeight={heatmapHeight}>
		<svg style="height:{heatmapHeight}; width:{heatmapWidth};">
			<g id="heatmap-scatterplot">
				{#if lectures !== undefined && selectedLectureNumbers!== undefined}
					{#each selectedLectureNumbers as selectedLectureNum}
						{#if lectures[selectedLectureNum] !== undefined}
							{#each lectures[selectedLectureNum].data as datapoint}
								<circle
									id="heatmap-datapoint-{datapoint["Nr"]}"
									cx={heatmapXScale(datapoint["x [pixel]"])}
									cy={heatmapHeight - heatmapYScale(datapoint["y [pixel]"])}
									r=1.5
									fill="blue"
									opacity="0.3"
								/>
							{/each}
						{/if}
					{/each}
				{/if}
			</g>

			<!--
			<rect
				height="{heatmapHeight}"
				width="{heatmapWidth}"
				x="0"
				y="0"
				fill="transparent"
				stroke="black"
				stroke-width="1" />

			-->
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
	<!--
			<g id="lecture-timeline">
				{#if lectures !== undefined && selectedLectureNum !== undefined && lectures[selectedLectureNum] !== undefined}
					{#each selectedData as datapoint}

					{/each}
				{/if}
			</g>

			-->
			<!--
			
			<circle 
				id="classroom-outline"
				cx="{heatmapWidth / 2}"
				cy="{heatmapWidth / 2}"
				r="{heatmapWidth / 2}"
				stroke="black"
				stroke-width="10"
				style="fill:none;"
				
			/>
			<circle
				r="{heatmapWidth / 4}"
				cx="{heatmapWidth / 2}"
				cy="{heatmapWidth / 2}"
				fill="transparent"
				stroke="tomato"
				stroke-width="{heatmapWidth / 2}"
				stroke-dasharray="{heatmapWidth / 2.6} 4"
			/>

			-->
		</svg>
	</div>

<style>
	.classroom-section-selector {
		opacity: 40%;
	}
	.classroom-section-selector:hover {
		cursor: pointer;
		fill: lightgray;
	}
</style>