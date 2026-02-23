# Calligraphy Tool - Context Documentation

## Project Overview
An HTML/CSS/JS application for drawing letters using a Wacom tablet or mouse. Uses SVG for rendering calame-shaped strokes.

---

## Interface Layout

### Left Hand Side (LHS) Panel
Located on the left side of the screen with controls organized in sections:

**Letter Selection**
- List of letters A-Z, 0-9 for selecting the current letter to edit
- Click to switch between letters, each creates a separate drawing layer

**Drawing Controls**
- `thickness`: Controls the width of the calame stroke (1-50 range)
- `length`: Controls the length of the calame (1-50 range)  
- `angle`: Controls the rotation angle of the calame tip in degrees (0-360)
- `interval`: Time in ms between automatic point placement during continuous drawing

**Guide Lines**
- `baseline`, `xheight`, `top`, `bottom`: Vertical guide positions
- `padleft`, `maxleft`, `center`, `maxright`, `padright`: Horizontal guide positions

**Action Buttons**
- `Clear`: Remove all points for current letter
- `Undo`: Revert last action
- `Delete Point`: Remove selected point
- `Delete Stroke`: Remove selected stroke
- `Simplify`: Reduce number of points using Douglas-Peucker algorithm
- `Resample`: Redistribute points evenly along path
- `Save`: Download JSON file with all letter data
- `Export SVG`: Export current letter as SVG file
- `Clear Selection`: Deselect all points

**Mode Toggle**
- `Draw Mode`: Drawing mode with calame brush
- `Edit Mode`: Select and manipulate points/strokes

**Calame Preview**
- Small 60x60 SVG preview showing the current calame shape (rectangle representing thickness and angle)

### Right Side (Canvas)
- SVG canvas in portrait orientation (400x560 viewBox)
- Contains guide lines and drawing layer
- Supports mouse and touch input

---

## Drawing Logic

### Stroke Modes

1. **Continuous Drag Drawing (default)**
   - Hold mouse button and drag to draw
   - Points are added based on `MIN_POINT_DISTANCE` (8 units)
   - Creates smooth curved strokes

2. **Pen Tool Mode (ALT + click)**
   - Press and hold ALT, then click to place individual points
   - Each click adds a point connected to previous point with a line segment
   - Release ALT to finish stroke
   - Multiple strokes can be created by pressing ALT again

3. **Straight Line Mode (SHIFT)**
   - Hold SHIFT while drawing to constrain to horizontal/vertical/45° lines
   - Can be combined with ALT (SHIFT+ALT+click) for straight segment drawing

### Calame Shape Algorithm
Located in `getCalamePoints(x, y, prevX, prevY)`:

```
1. If no previous point: create a simple rectangle centered at (x,y)
2. Otherwise:
   - Calculate direction vector from prev to current point
   - Compute perpendicular normal (nx, ny)
   - Extend tip by (tipX, tipY) = (cos(angle)*length, sin(angle)*length)
   - Return 4 points forming a parallelogram shape:
     [prev + offset, current + offset, current + offset + tip, prev + offset + tip]
```

### Rendering
- Uses `smoothPoints()` for Bezier curve smoothing
- Uses `simplifyPoints()` (Douglas-Peucker algorithm) to reduce point count

---

## Key Functions

| Function | Purpose |
|----------|---------|
| `startDrawing(e)` | Initiates drawing on mousedown |
| `draw(e)` | Handles mousemove for continuous drawing |
| `stopDrawing(e)` | Finalizes stroke on mouseup |
| `getCalamePoints(x, y, prevX, prevY)` | Calculates calame polygon shape |
| `renderLetter()` | Renders all strokes and UI elements |
| `simplifyPoints(points, tolerance)` | Douglas-Peucker simplification |
| `smoothPoints(points)` | Bezier curve smoothing |

---

## Issues & Lessons Learned

### Issue 1: Calame Preview Shape Was Triangular
**Problem**: The calame preview in the LHS panel displayed as a triangle instead of rectangle.

**Root Cause**: In `updateCalamePreview()`, one bottom point incorrectly used `cy - thickness/2` instead of `cy + thickness/2`.

**Fix**: Changed both bottom points to use `cy + thickness/2`.

**Lesson**: Always verify coordinate symmetry when defining polygon vertices.

---

### Issue 2: Drawing Preview Didn't Match Drawing Output
**Problem**: The calame preview showed a different size than what was drawn.

**Root Cause**: The preview multiplied length by 0.8 (`length * 0.8`) while the actual drawing used full length.

**Fix**: Removed the 0.8 multiplier to make preview match actual output.

**Lesson**: Preview and actual rendering must use identical parameters.

---

### Issue 3: ALT Key Not Detected Before Click
**Problem**: Tried to implement pen-tool mode with ALT key, but `altKeyPressed` was false when clicking because the browser handles ALT key differently.

**Solution**: 
- Use toggle behavior: press ALT once to enter pen mode, press again to exit
- Alternatively, rely on keydown firing before mousedown in most cases
- Added proper event listeners with preventDefault

**Lesson**: Browser key event handling varies. Test modifier keys thoroughly across browsers. Consider toggle behavior vs. hold behavior.

---

### Issue 4: Pen Tool Mode Logic Errors
**Problem**: Even after fixing ALT detection, the pen tool mode was broken:
- Normal drag drawing stopped working
- Points were being added incorrectly

**Root Causes**:
1. `startDrawing` always checked `if (lastPoint && lastPoint.stroke)` to add to existing stroke, even in normal mode
2. The `draw` function had incorrect early return logic
3. The stroke-ending logic was in the wrong place

**Fix**:
- Only add to existing stroke in pen tool mode (`if (altKeyPressed && lastPoint && lastPoint.stroke)`)
- Normal drag mode always creates new stroke
- Moved stroke finalization to keyup handler for ALT

**Lesson**: Clearly separate mode-specific logic paths. Test each mode independently.

---

### Issue 5: Keyboard Shortcut Collision
**Problem**: Initially tried to use "L" key for pen tool mode, but "L" is already used for letter selection (pressing a letter key selects that letter).

**Fix**: Switched to ALT key instead, which wasn't used for any other function.

**Lesson**: Check existing keyboard shortcuts before assigning new ones. Document all keybindings.

---

## How to Avoid These Errors

1. **Before adding new keyboard shortcuts**: Review existing key handlers and ensure no conflicts
2. **When creating preview visuals**: Use the exact same parameters as the actual rendering code
3. **When implementing mode-specific behavior**: Use clear conditional branches, test each mode independently
4. **When debugging key events**: Add logging, test in browser console, account for browser differences
5. **Document all keybindings** in a central location to prevent future conflicts
