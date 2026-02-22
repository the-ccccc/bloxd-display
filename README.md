# bloxd-display
A custom in-game screen-like display for the voxel game bloxd.io

Available functions :

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
