Final Project - CS 499 - Visual Analytics

Hannah Armstrong
Dan Hickey
Brian Cebra

TODO:

-Import and clean data into arrays/objects
	
-Skeleton code for a blank HTML site with a container for the visualizations (moving to a published website will be LAST)


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
			Go to Time = 0 button
			Start Animation button
			Go to Time = last button

-Box that contains summary and line charts (on the right)
	Elements:
		Statistic 1
		Statistic 2

		Line Chart 1
		Line Chart 2

-Animation Script
	Play -> every x seconds / frames update
			Instructor position
			Instructor movement type
			Current Highlighted Action
	Add new circle for instructor position, make last circle lower opacity to start generating the heatmap
		