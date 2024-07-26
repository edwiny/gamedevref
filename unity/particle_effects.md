# Particle effects notes
## Using sprites for 2D Particle Effects

1. Create a material, and use the 'Particle/Standard Unlit' shader.
2. In the Maps section of the Inspector, drag the sprite to what looks like a checkbox.
3. In the color/hdr section, set to white.


New create a GameObject and add the Particle System component.

1. Renderer module:
  * Set "Render Mode" to "Billboard" and ensure "Render Alignment" is set to "World".
  * Drag the material over into the Material field.
4. In the main module
  * Set "Simulation Space" to "World" for better performance in 2D.
  * Disable the "3D Start Size" and "3D Start Rotation" options in the main module.
  * Set Gravity Source to 2D
