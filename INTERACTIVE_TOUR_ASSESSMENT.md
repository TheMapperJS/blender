# Assessment: Interactive Tour Add-on for Blender

## Executive Summary
Building a **comprehensive, context-aware, step-by-step informational tour** as a Blender Python Add-on is a project of **moderate to high difficulty**.

While creating the logic for a step-by-step informational slideshow that is aware of the user's current context (Object Mode, Edit Mode, etc.) is relatively straightforward using Blender's Python API, the true difficulty lies in two areas:
1. **Technical Limitation (UI Highlighting)**: The Blender Python API does not provide native support for highlighting arbitrary UI elements or retrieving the exact screen coordinates of buttons/panels.
2. **Scope of Content**: Creating a *comprehensive* tour for software as massive as Blender requires a monumental amount of content creation and maintenance.

---

## Technical Breakdown

### 1. Add-on Framework & State Management (Difficulty: Low)
Creating a Python Add-on that maintains the state of the tour (e.g., `current_step = 5`) is simple.
- You can use Blender's `Window Manager` (`bpy.context.window_manager`) to store custom properties that track whether the tour is active, which module is currently playing, and the current step index.

### 2. Context-Awareness (Difficulty: Moderate)
Making the tour context-aware is achievable.
- Blender's context (`bpy.context`) provides real-time information about the active workspace, active mode (Object, Edit, Sculpt), and active object type.
- The add-on can use application handlers (e.g., `bpy.app.handlers.depsgraph_update_post` or timer events) to detect when the context changes and prompt the user to switch to a different tour module or automatically update the tour content.

### 3. User Interface & Presentation (Difficulty: High)
This is the most technically challenging aspect.
- **The Ideal UX**: A highlighted box around a specific tool or panel, with a floating tooltip containing the description and "Next/Prev" buttons.
- **The Reality**: Blender's Python API **does not** expose the layout coordinates of UI elements (like a specific button in the Properties editor). You cannot tell the API "draw a box around the Modifiers tab."
- **Workarounds**:
    - **Dedicated Tour Panel**: The easiest approach is to have a dedicated "Tour" panel in the N-panel (Sidebar) of the 3D Viewport. The user reads instructions there and manually looks for the UI elements described.
    - **Viewport Drawing (GPU Module)**: You can draw custom graphics (like floating text boxes) inside the 3D Viewport using the `gpu` and `blf` modules. However, this is restricted to the viewport region and cannot easily point to elements in other editors (like the Outliner or Properties panel).
    - **Popups/Dialogs**: `invoke_props_dialog` can be used to show steps, but it blocks user interaction until dismissed, ruining the "tour" experience.

### 4. Comprehensive Content Creation (Difficulty: Very High / Tedious)
Blender has dozens of editors, hundreds of tools, and complex workflows.
- Mapping out a "comprehensive" tour requires writing thousands of steps.
- **Maintenance Burden**: Blender's UI changes frequently between major versions (e.g., 3.x to 4.x). A comprehensive tour add-on would require constant updates to text and hotkey descriptions every time Blender is updated.

---

## Proposed Architecture for the Add-on

If you were to proceed, the best approach would be a **Data-Driven Panel Add-on**:

1. **Content Dictionary (JSON/Python Dict)**: Define the tour steps in an external file.
   ```json
   {
     "object_mode_basics": [
       {"title": "Movement", "text": "Press 'G' to grab and move the object.", "hotkey": "G"},
       {"title": "Properties", "text": "Look at the Properties panel on the right to change object color.", "hotkey": null}
     ]
   }
   ```
2. **Context Listener**: A background timer (`bpy.app.timers`) that periodically checks `bpy.context.mode` and suggests the relevant tour if the user switches modes.
3. **UI Panel**: A standard Blender Panel (`bpy.types.Panel`) located in the 3D Viewport Sidebar (`VIEW_3D`, `UI`).
    - It displays the current step's `title` and `text`.
    - It contains standard `operator` buttons for `Next` and `Previous`.

## Conclusion
Creating a basic informational "Sidebar Tour" is achievable within a **few days to a week** for an experienced Blender Python developer. However, achieving a true "interactive overlay" that highlights UI elements is effectively impossible without modifying Blender's C/C++ source code. Furthermore, making the tour truly *comprehensive* is a massive content-writing undertaking that requires ongoing maintenance.
