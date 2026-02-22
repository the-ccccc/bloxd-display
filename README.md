# bloxd-display
A custom in-game screen-like display for the voxel game bloxd.io

## Overview
The **Virtual Screen Engine** is a lightweight, purely JavaScript-based rendering system designed to simulate a pixelated 2D display inside a 3D environment. It uses the Bloxd api function `api.setDirectionArrow` to render rows of emojis acting as "pixels." 

It features an integrated 8x8 font engine, batch-rendering to prevent server/client lag, dynamic color palettes, and global screen state management.

---

## 🚀 Quick Start / Setup

### 1. Initialization
```javascript
onPlayerJoin = (id) => {
    // Create a new screen instance (global var so code blocks can access it too)
    myScreen = new Screen(
        id, 
        "main_display", 
        [0.5, 1, 0.5], // World position [x, y, z]
        "default"      // Use default color palette
    );
    
    // Draw some elements
    myScreen.fill("black");
    myScreen.drawText("HELLO WORLD", 10, 10, 1, "white");
    
    // Push the buffer to the display
    myScreen.render(); 
}
```

### 2. Processing the Render Queue
You **must** include this in your game's main `tick` function for the screens to physically render in the world.
```javascript
function tick() {
    for (const screen of globalThis.activeScreens.values()) {
        screen.onTick();
    }
}
```

---

## 🛠️ API Reference

### Global Management
The engine registers all active screens into a global map for easy access and memory management.

*   `globalThis.activeScreens`: A standard JavaScript `Map` containing all active `Screen` instances, keyed by their `screenId`.
*   `deleteScreen(screenId)`: A global helper function that finds a screen by its ID and fully deletes it, removing it from the world and memory.

---

#### Configuration Options (`options`)
The `options` parameter accepts either a string preset or a custom configuration object.

**String Presets:**
*   `"default"`: Uses standard dimensions (285x125) and loads the default color palette (Red, Orange, Yellow, Green, Blue, Purple, Brown, Black, White).
*   `"bw"`: Loads only Black (`⬛`) and White (`⬜`).

**Custom Object:**
```javascript
{
    width: 50,             // Width in pixels (emojis)
    height: 20,            // Height in pixels (emojis)
    alwaysRender: true,   // Override global alwaysRender
    colorMap: {            // Custom palette mapping
        "red": "🟥", 
        "black": "⬛", 
        "white": "⬜"
    }
}
```
*Note: The engine natively supports integer fallbacks `0` for Black and `1` for White regardless of the palette.*

---

## 🔤 The `font8x8` Dictionary
The code includes a global `font8x8` object. It acts as a ROM for character graphics. 
*   Supports Numbers (`0-9`)
*   Supports Uppercase (`A-Z`)
*   Supports Lowercase (`a-z`)
*   Supports Standard Punctuation (`!`, `?`, `@`, etc.)

Each character is defined by an array of 8 hexadecimal bytes representing an 8x8 grid. If a character passed into `drawText` does not exist in the dictionary, it will automatically fallback to the `?` character.

---

## ⚠️ Performance Tips & Best Practices

1. **Avoid `alwaysRender: true` for complex graphics:** 
   If you are drawing a complex UI or iterating through many `setPixel` calls, leave `alwaysRender` off. Make all your updates to the buffer first, and then call `screen.render()` once at the end. Otherwise, every single pixel change will restart the rendering loop.
2. **Adjusting Batch Size:**
   If the screen is rendering too slowly, you can manually increase the batch size:
   ```javascript
   myScreen.batchSize = 20; // Renders 20 rows per tick instead of 10
   ```
   *Warning: Setting this too high WILL cause interruptions. 10 is somewhat safe, 15 is okay and 20 starts to have quite a few interrupts.*
3. **Screen Size Limits:**
   Because this relies on arrow subtitles,  high resolutions (e.g., height > 125) won't display entirely (user will have to pan up and down).

---

### Example Usage :

```js
onPlayerJoin = (id) => {
  // Option 1: Basic setup
  mainScreen = new Screen(id, "screen1", [0.5, 1, 0.5], "default", true);
  
  // Option 2: Custom map with "alwaysRender" enabled
  /*mainScreen = new Screen(id, "screen1", [0.5, 1, 0.5], {
      colorMap: { red: "🟥", black: "⬛", white: "⬜" },
      width: 50,
      height: 20,
      alwaysRender: true
  });*/
  mainScreen.fill(0); // Fills black
  
  for (let i=0;i<50;i++){
    for (let j=0;j<10;j++){
      mainScreen.setPixel(8+i,8+j,"white")
    }
  }
  mainScreen.drawText("White", 10, 0, 0, "white"); 
  mainScreen.drawText("Black", 10, 9, 0, "black"); 
  mainScreen.drawText("Red", 10, 18, 0, "red"); 
  mainScreen.drawText("Orange", 10, 27, 0, "orange"); 
  mainScreen.drawText("Green", 10, 36, 0, "green"); 
  mainScreen.drawText("Brown", 10, 45, 0, "brown"); 
  mainScreen.drawText("Blue", 10, 54, 0, "blue"); 
  mainScreen.drawText("Purple", 10, 63, 0, "purple"); 
  mainScreen.drawText("Yellow", 10, 72, 0, "yellow");
  api.setClientOption(id, "cameraTint", [0,0,0,1]) //Dark screen for better experience - should match background color
  api.setCameraDirection(id, [1, 0.06, 0]);
  mainScreen.render()
}
```

# Available functions :

```js

/**
 * Helper function to globally delete an existing screen and remove it from the registry.
 * @param {string} screenId - The unique identifier of the screen to delete.
 */
function deleteScreen(screenId)

/**
 * Represents a virtual 2D screen rendered using emojis
 * @class
 */
class Screen {
  /**
   * Creates a new Screen instance.
   * @param {string|number} playerId - The ID of the player this screen is rendered for.
   * @param {string} screenId - A unique identifier for this screen instance.
   * @param {number[]} screenPos - The [x, y, z] spatial coordinates of the screen in the world.
   * @param {string|Object} [options="default"] - Screen configuration options or a preset string ("default" or "bw").
   * @param {number} [options.width=285] - The width of the screen in pixels (emojis).
   * @param {number} [options.height=125] - The height of the screen in pixels (emojis).
   * @param {boolean} [options.alwaysRender=false] - Whether to automatically trigger a render on every buffer change.
   * @param {Object<string, string>} [options.colorMap] - A mapping of color names to emoji characters.
   * @param {boolean} [alwaysRender=false] - Fallback parameter to enable auto-rendering if options is passed as a string preset.
   */
  constructor(playerId, screenId, screenPos, options = "default", alwaysRender = false)

  /**
   * Resolves a color name or integer fallback to its internal palette index.
   * @private (Palette is user-defined anyway or default)
   * @param {string|number} color - The color identifier to resolve.
   * @returns {number} The index of the color in the internal palette.
   */
  _getColorIndex(color)

  /**
   * Fills the entire screen buffer with a single color.
   * @param {string|number} color - The color to fill the screen with.
   */
  fill(color)

  /**
   * Sets the color of a specific pixel on the screen.
   * @param {number} x - The X coordinate of the pixel (0 is the left edge).
   * @param {number} y - The Y coordinate of the pixel (0 is the top edge).
   * @param {string|number} color - The color to apply to the pixel.
   */
  setPixel(x, y, color)

  /**
   * Draws a string of text onto the screen using the internal 8x8 font.
   * @param {string} text - The text string to draw.
   * @param {number} x - The starting X coordinate.
   * @param {number} y - The starting Y coordinate.
   * @param {number} [offset=1] - The number of empty pixels to insert between characters.
   * @param {string|number} [color="white"] - The color of the text.
   */
  drawText(text, x, y, offset = 0, color = "white")

  /**
   * Queues the current screen buffer to be rendered to the game world.
   * Resets the internal render pointer and flags the rendering state as active.
   */
  render()

  /**
   * Processes the render queue incrementally.
   * Must be called continuously inside tick
   * Renders up to `batchSize` rows per tick to optimize performance.
   */
  onTick()

  /**
   * Completely removes the screen from the game world and the global registry.
   * Halts any active rendering and clears all associated visual direction arrows via the external API.
   */
  delete()

  /**
   * Renders a specific row of the pixel buffer into the physical game world.
   * @private (Use screen.render instead)
   * @param {number} y - The Y coordinate (row index) to render.
   */
  _renderRow(y)
}
```
