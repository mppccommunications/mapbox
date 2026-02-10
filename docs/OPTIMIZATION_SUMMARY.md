# Church Model Optimization Summary

## Date: February 9, 2026

## Original Problem

The church 3D model was appearing **inverted/upside-down** in Mapbox GL JS when using the original configuration:

```javascript
'model-scale': [-0.03, -0.03, -0.03],  // Negative scale inverted the model
'model-rotation': [0, 0, 18],
'model-translation': [-35, -20, 0]
```

**Root Cause**: The negative scale values (`-0.03`) were used as a workaround to flip an incorrectly oriented model, but this created an inverted geometry that appeared upside-down in the 3D viewport.

## Optimization Process

### 1. Blender Model Cleanup (mppc-micro2.glb)
The model was reprocessed in Blender to:
- **Bake all transforms**: Applied all rotation, scale, and location transforms directly to the mesh geometry
- **Convert to Y-up coordinate system**: Standardized the model's up-axis to Y (industry standard)
- **Remove negative scales**: Eliminated the need for hacky scale workarounds
- **Optimize geometry**: Clean mesh structure for better performance

### 2. Model Export Settings
- Format: glTF Binary (.glb)
- Up-axis: Y-up
- Forward-axis: -Z forward
- All transforms applied to geometry
- Compressed for web delivery

## Why the Solution Works

Mapbox GL JS uses a **Z-up coordinate system** for 3D models, where:
- X-axis: East-West
- Y-axis: North-South
- Z-axis: Vertical (up)

Since the optimized model uses **Y-up** coordinates (standard for most 3D software), we need to convert it:

**The 90° pitch rotation** converts Y-up to Z-up:
```javascript
'model-rotation': [90, 0, 18]  // [pitch, yaw, roll]
```

- **Pitch 90°**: Rotates the model 90° around the X-axis, mapping Y-up to Z-up
- **Yaw 0°**: No horizontal rotation needed
- **Roll 18°**: Maintains the building's alignment with the street grid

With the transforms baked into the geometry and Y-up standardized, we can use **positive scale values** that represent actual world-space dimensions.

## Final Working Settings

```javascript
map.addLayer({
  id: 'church',
  type: 'model',
  slot: 'middle',
  source: 'model',
  minzoom: 15,
  layout: { 'model-id': ['get','model-uri'] },
  paint: {
    'model-opacity': 1,
    'model-scale': [0.03, 0.03, 0.03],        // Positive scale, no inversion
    'model-rotation': [90, 0, 18],             // Y-up to Z-up conversion
    'model-translation': [-35, -20, 0],        // Fine-tune positioning
    'model-emissive-strength': 0.2,
    'model-color': '#f5e0c3',
    'model-color-mix-intensity': 0.3
  }
});
```

### Key Parameters Explained

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `model-scale` | `[0.03, 0.03, 0.03]` | Uniform positive scale - model appears at correct size |
| `model-rotation` | `[90, 0, 18]` | 90° pitch converts Y-up to Z-up; 18° roll aligns with streets |
| `model-translation` | `[-35, -20, 0]` | Fine-tune horizontal positioning (West, South, no vertical) |
| `model-uri` | `mppc-micro2.glb` | Optimized model with baked transforms |

## Benefits of This Approach

1. **Correct Orientation**: Model displays right-side-up without geometric inversion
2. **Clean Configuration**: Positive scale values, no hacky negative workarounds
3. **Better Performance**: Baked transforms reduce runtime calculations
4. **Maintainable**: Clear parameter meanings, easier to adjust and debug
5. **Industry Standard**: Y-up model with documented Z-up conversion

## Verification

Test the configuration by:
1. Loading the map at coordinates: `-80.826542, 35.193527`
2. Setting pitch to 60° and bearing to -50°
3. Confirming the church model appears upright with correct roof orientation
4. Verifying the steeple points skyward (positive Z direction)

---

**Model File**: `mppc-micro2.glb`
**Deployed Location**: `https://mppccommunications.github.io/mapbox/models/mppc-micro2.glb`
**Configuration File**: `260209 Mapbox Code Block - OPTIMIZED.txt`
