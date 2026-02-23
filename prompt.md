this an html/css page + js; interface should be minimalist (10px font size, monospace, B/W, no shadow, 1px solid black border, 5px margin)

the user will be able to use a wacom tablet to draw letters in svg
there is a left panel with option box
on the right hand side there is the "canvas" on which we will draw; it's a letter format in portait orientation

on the left side, there is a list of letters a-zA-Z0-9 (we will be able to add some more, later)
when activating a letter, each letter will create a specific layer on which to draw
drawing: we lay a point every XXms (there is a control in the LHS panel), on the LHS there is a way to control the shape and angle of the calame: length, thickness, angle. 

we can save the points attached to all character (json) - we download a json file, it's droppable to restore the data
it's also possible to export a single character as svg



