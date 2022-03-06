Final Project - CS 499 - Visual Analytics

Hannah Armstrong
Dan Hickey
Brian Cebra

DO NOT forget to run "npm install" command before attempting to launch to localhost/8080
(commands: "npm run build", "npm run start". ctrl + c, "y" to exit the launch)

TODO:

-Import and clean data into arrays/objects

ASSIGNED:

-Hannah: Base site divs
-Dan: Graph image data
-Brian: Data Import, Graph sections of classroom data


Site Elements:

-Box that "plays" movement (on the left)
	Elements:
		Instructor location, shown by an icon, shows movement type
		Background - Shapes representing the classroom's position, text showing specific object locations
		Top bar (Left to Right): 
			Action icon set
			Text showing classroom type
			Instructor ID
			(DEBUG USE:) Frame #
		Bottom bar:
			Button to show list of selectable sessions
			Timeline with frame ticks, see start through end with a movable header 
			Play/pause/go to end/go to beginning button
		Invisible:
			Split classroom into sections, track aggragated data per section and display on hover (per lecture or for all)
				Instructor spends x% time here
				etc...

-Box that contains summary and line charts (on the right)
	Elements:
		Statistic 1
		Statistic 2
		etc...

		Line Chart: Distance between points, plotted for each instructor
		Chart: Total distance traveled (Maybe updates with the animation)
		Chart: Velocity over time
		etc... 

-Animation Script
	Play -> every x seconds / frames update
			Instructor position
			Instructor movement type
			Current Highlighted Action
	Add new circle for instructor position, make last circle lower opacity to start generating the heatmap
		