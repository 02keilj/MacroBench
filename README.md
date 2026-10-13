# Macro Bench for FluidNC

A browser-based, step-by-step builder for G-code macros on [FluidNC](https://github.com/bdring/FluidNC) CNC controllers. You build a macro from blocks, see the toolpath on a top-down or 3D preview of your machine, and get G-code you can copy or download and run on the controller.

It's a single self-contained HTML page. There's no install and no server, and nothing leaves your browser.

---

## Features

### Building macros
- **Step builder.** Add steps from a colour-coded palette, edit their settings, reorder them, nest them inside blocks, duplicate and delete. A search box finds steps quickly; press Enter to add the first match.
- **Expressions everywhere.** Any number field accepts an expression such as `#<_width> / 2` or `FUP[#<_r> / 3]`. The app adds the square brackets G-code needs.
- **Variables and logic.** Set variable, If/Else, While, Repeat, Depth passes, Break and Continue. These compile to FluidNC o-code blocks (`o100 if […]` … `o100 endif`), and the o-numbers are assigned automatically.
- **Required inputs.** A guard that stops the macro with a message if the operator hasn't set a value, such as `#<_depth>`, before running. It can clear the value at the end so the next run needs a fresh one.
- **Patterns.** Spiral (inward or outward), zigzag, concentric circles and concentric rectangles. Each generates a compact parametric loop. Every pattern has an optional step-down for multi-pass cuts, can be expanded into editable steps, and can be collapsed back into a pattern.
- **Undo/redo.** Use the header buttons, Ctrl+Z or Ctrl+Shift+Z.

### Machine awareness
- **Machine panel.** Set travel limits, home position, machine safe Z, rapid rate and tool diameter. Choose whether the macro runs from a homed machine (fixed positions) or from wherever you jog to (relative).
- **Three coordinate modes on moves.** Work coordinates, machine coordinates (`G53`) and relative to the current position (`G91` … `G90`).
- **Work offsets.** `G10 L2` puts the origin at a fixed machine position. `G10 L20` makes the current position become the values you give. `G54`–`G59` are supported.

### Preview and simulation
- **Live simulation.** The macro actually runs in the browser, including variables, loops and conditions, and draws the real toolpath in machine coordinates.
- **Top and 3D views.**
  - **Top view:** cut width is drawn at the tool diameter.
  - **3D view:** orbit, pan, zoom toward the cursor, preset views (Iso, Top, Front, Right) and Z exaggeration.
- **Playback.** Play, step through moves one at a time, or scrub. The readout shows:
  - the active step and its G-code
  - work and machine position
  - feed rate
  - the value of every variable at that moment
- **Warnings.**
  - moves outside travel limits
  - cuts with no feed rate set
  - bad expressions
  - arcs whose end point isn't on the circle
  - runaway loops
  - built-in names used as variables
- **Stats.** Estimated run time, cutting and rapid distance, and Z range.

### Output
- **Full mode** keeps variables and o-code logic, and lists which advanced features the macro relies on.
- **Plain G-code mode** unrolls every loop, condition and variable into fixed numbers, for firmware without expression support.
- **Setup notes header**: an optional comment block with start conditions, units, work zero, required inputs, spindle, tool, travel range and estimated time.
- **Copy or download** the G-code. Downloads save as `.txt` because of viewer file-type limits; rename the file to `.nc` if your sender filters by extension.

### Import and projects
- **G-code import.** Paste text or open a `.nc` or `.gcode` file. It converts the following into editable steps:
  - moves, arcs, probing
  - spindle, coolant, outputs
  - offsets, tool changes, tool length offsets, program end
  - variables, and o-code `if`/`elseif`/`else`, `while`, `do`/`while` and `repeat`

  Anything else is kept as Raw G-code, so nothing is lost.
- **Projects.** Projects save automatically in your browser. You can also open, copy or download a project as JSON.

### Learning
- **Tutorial.** A guided 14-step walkthrough that builds a three-spot marking macro using a counter, Repeat and If/Else. It includes "Do it for me" and restores your previous macro afterwards.
- **Reference.** A searchable page with an entry for every step type covering:
  - what it does
  - the exact G-code it writes
  - each setting
  - an example
  - things to watch out for

  It also has concept pages for coordinates, variables, conditions, o-codes, safe heights, built-in values, output modes and the preview. The **?** button on any step opens its entry.

### Interface
- Resizable columns, a collapsible machine panel, a pinned step palette, an option to hide the G-code, and an Auto/Light/Dark theme switch.

---

## Step types

| Group | Steps |
|---|---|
| **Motion** | Rapid move (`G0`), Cutting move (`G1`), Retract, Arc (`G2`/`G3`), Probe (`G38.2`), Home (`$H`), Set work zero (`G10 L2`/`L20`), Use coordinate system (`G54`–`G59`) |
| **Patterns** | Spiral, Zigzag, Concentric circles, Concentric rectangles |
| **Machine** | Spindle (`M3`/`M4`/`M5`), Coolant (`M7`/`M8`/`M9`), Digital output (`M62`–`M65`), Wait (`G4`), Pause (`M0`), Tool change (`T… M6` / `M61 Q`), Tool length offset (`G43.1` / `G49`), Units (`G20`/`G21`), Distance mode (`G90`/`G91`), End program (`M2`/`M30`) |
| **Variables and logic** | Set variable, Required input, If/Else, While, Repeat, Depth passes, Break, Continue |
| **Notes** | Comment, Show message (`MSG`), Raw G-code |

### Built-in values
Expressions can read these values, which FluidNC sets itself:

| Value | Meaning |
|---|---|
| `#<_x>`, `#<_y>`, `#<_z>` | Current position in work coordinates |
| `#5061`, `#5062`, `#5063` | Where the last probe triggered (X, Y, Z) |
| `#<_current_tool>`, `#<_selected_tool>` | Fitted tool and requested tool |

---

## Quick start

1. Open the page and click **Tutorial** for a guided first macro, or pick an example from **File**.
2. Open the **Machine** panel and enter your travel limits, home position and safe Z.
3. Add steps from **Add a step**. New steps go after the selected step, or inside a block when you click **+ Add steps inside**.
4. Check the preview and the warnings list, then press **Play** to step through the run.
5. Copy or download the G-code.

## Running a macro on FluidNC

Save the file to the controller's SD card or local filesystem, then run it with:

```
$SD/Run=/name.nc
$LocalFS/Run=/name.nc
```

You can also point a `macro0`–`macro3` entry in your FluidNC config at the file so it runs from a button.

## Compatibility notes

- Variables, expressions and o-code flow control need a FluidNC release with expression support. Published community scripts using them target **FluidNC 3.8.2 or newer**. If your firmware rejects them, switch Output to **Plain G-code**.
- `#5063` as the probe Z result is confirmed in a published FluidNC script. The other built-in values follow LinuxCNC naming, which FluidNC is based on. Test them once on your firmware, for example by printing a value with a message step.
- What `M6` does depends on your config: either `atc_manual` (park, swap, measure on the tool setter) or a custom `m6_macro`. The preview tracks the tool number but doesn't simulate the change.
- Some users report that pauses (`M0` or feed hold) inside macros behave unreliably. Test them before relying on them.
- The preview is a prediction. Probing is drawn as the full travel to its target, estimated time ignores acceleration, and Raw G-code steps aren't simulated. Always do a dry run above the work before cutting.

## Storage and privacy

Your macro, layout and theme are saved in this browser's local storage only. Nothing is sent anywhere. Clearing site data or using a private window starts fresh, so use **File → Download project** or **Copy project JSON** to keep a backup.

## Tech

- One HTML file with vanilla JavaScript and no build step.
- Canvas 2D rendering for both the top view and the orthographic 3D view.
- A built-in expression evaluator and simulator that mirror FluidNC's LinuxCNC-style parameters and o-codes.
- Fonts: Chakra Petch, IBM Plex Sans and IBM Plex Mono, from Google Fonts.
