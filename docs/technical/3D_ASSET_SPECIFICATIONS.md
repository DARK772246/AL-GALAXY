# AI GALAXY - 3D Asset Specifications & Animation Guidelines

## 1. Asset Overview

### 1.1 Asset Categories
- **Celestial Bodies**: Planets, stars, moons representing AI tools
- **Environmental Assets**: Galaxy backdrop, nebulae, cosmic dust
- **UI Elements**: 3D interface components, particle effects
- **Animation Assets**: Orbital paths, tool interactions, transitions

### 1.2 Technical Constraints
- **Platform**: React Native with Skia/Reanimated
- **Performance Target**: 60fps on mid-range devices (3GB RAM)
- **File Size Limit**: Maximum 2MB per asset, 50MB total
- **Polygon Count**: <10,000 triangles per major asset
- **Texture Resolution**: 512x512 for planets, 256x256 for UI elements

---

## 2. Celestial Body Specifications

### 2.1 Planet Design System

**Size Classification:**
```typescript
enum PlanetSize {
  DWARF = 0.5,    // Niche tools, new releases
  SMALL = 0.75,   // Specialized tools
  MEDIUM = 1.0,   // Popular tools
  LARGE = 1.25,   // Major platforms
  GIANT = 1.5     // Industry leaders (ChatGPT, etc.)
}
```

**Planet Categories with Visual Themes:**

**Chat Planets (Communication Tools)**
- Base Color: Electric Blue (#00D4FF)
- Surface: Pulsing communication waves
- Atmosphere: Soft blue glow with text particles
- Examples: ChatGPT, Claude, Bard

**Image Generation Planets (Creative Tools)**
- Base Color: Plasma Pink (#FF006E)
- Surface: Swirling artistic textures
- Atmosphere: Rainbow particle effects
- Examples: Midjourney, DALL-E, Stable Diffusion

**Coding Planets (Development Tools)**
- Base Color: Matrix Green (#39FF14)
- Surface: Circuit board patterns
- Atmosphere: Code symbol particles
- Examples: GitHub Copilot, Cursor, Replit

**Music/Audio Planets (Audio Tools)**
- Base Color: Solar Gold (#FFB700)
- Surface: Sound wave visualizations
- Atmosphere: Musical note particles
- Examples: Mubert, Soundraw, Boomy

### 2.2 Planet Asset Structure

**File Organization:**
```
assets/3d/planets/
├── chat/
│   ├── chatgpt.json         # Geometry data
│   ├── chatgpt_diffuse.png  # Base texture
│   ├── chatgpt_normal.png   # Normal map
│   └── chatgpt_glow.png     # Emission map
├── image/
├── coding/
└── music/
```

**Asset Properties:**
```typescript
interface PlanetAsset {
  id: string;
  category: AICategory;
  size: PlanetSize;
  textures: {
    diffuse: string;     // Base color texture
    normal: string;      // Surface detail normals
    emission: string;    // Glow/light emission
    roughness?: string;  // Surface roughness map
  };
  geometry: {
    vertices: Float32Array;
    indices: Uint16Array;
    normals: Float32Array;
    uvs: Float32Array;
  };
  animation: PlanetAnimation;
}
```

### 2.3 Orbital Mechanics

**Orbit Configuration:**
```typescript
interface OrbitConfig {
  radius: number;        // Distance from galaxy center
  speed: number;         // Rotation speed (radians/second)
  inclination: number;   // Orbital plane tilt
  eccentricity: number;  // Orbit ellipse (0 = circle, 1 = parabola)
  phase: number;         // Starting position offset
  direction: 1 | -1;     // Clockwise or counterclockwise
}

// Orbit rings by category
const ORBIT_RINGS = {
  INNER_RING: { radius: 200, speed: 0.5 },    // Featured tools
  MIDDLE_RING: { radius: 400, speed: 0.3 },   // Popular tools
  OUTER_RING: { radius: 600, speed: 0.1 },    // Specialized tools
  ASTEROID_BELT: { radius: 800, speed: 0.05 } // New/experimental tools
};
```

---

## 3. Environmental Assets

### 3.1 Galaxy Background

**Skybox Specifications:**
- **Format**: Cubemap (6 faces, 1024x1024 each)
- **Content**: Deep space with distant stars and nebulae
- **Color Temperature**: Cool blues and purples
- **Brightness**: Low ambient to enhance planet visibility

**Nebula Effects:**
```typescript
interface NebulaConfig {
  position: Vector3;
  size: number;
  density: number;
  color: ColorRGB;
  animation: {
    drift: Vector3;      // Slow movement vector
    pulse: number;       // Brightness oscillation
    swirl: number;       // Rotation speed
  };
}
```

### 3.2 Particle Systems

**Star Field:**
- **Count**: 5000-10000 points
- **Distribution**: Uniform random in sphere
- **Brightness**: Varied based on distance
- **Twinkling**: Subtle opacity animation

**Cosmic Dust:**
- **Purpose**: Depth perception and atmosphere
- **Behavior**: Slow drift with parallax effect
- **Density**: Lower in center, higher at edges
- **Interaction**: Particles avoid planets

**Shooting Stars:**
- **Trigger**: New tool announcements
- **Path**: Curved trajectory across view
- **Effect**: Bright streak with particle trail
- **Frequency**: 1-2 per minute during active use

---

## 4. Animation System

### 4.1 Planet Animations

**Idle Animations:**
```typescript
const planetIdleAnimation = {
  rotation: {
    axis: [0, 1, 0],           // Y-axis rotation
    speed: 0.1,                // Slow rotation
    variation: 0.02            // Slight speed variation
  },
  float: {
    amplitude: 5,              // Vertical movement range
    frequency: 0.5,            // Float speed
    offset: Math.random()      // Randomized phase
  },
  glow: {
    minIntensity: 0.8,
    maxIntensity: 1.2,
    pulseSpeed: 2.0
  }
};
```

**Interaction Animations:**
```typescript
const planetHoverAnimation = {
  scale: {
    from: 1.0,
    to: 1.1,
    duration: 200,
    easing: 'easeOutCubic'
  },
  glow: {
    from: 1.0,
    to: 1.5,
    duration: 200,
    easing: 'easeOutCubic'
  },
  particles: {
    spawn: true,
    type: 'sparkles',
    count: 20
  }
};
```

### 4.2 Camera Animations

**Navigation Transitions:**
```typescript
interface CameraAnimation {
  position: {
    from: Vector3;
    to: Vector3;
    duration: number;
    easing: EasingFunction;
  };
  rotation: {
    from: Vector3;
    to: Vector3;
    duration: number;
    easing: EasingFunction;
  };
  fov: {
    from: number;
    to: number;
    duration: number;
  };
}

// Example: Zoom to planet
const zoomToPlanetAnimation: CameraAnimation = {
  position: {
    from: currentPosition,
    to: planetPosition.add(offset),
    duration: 1000,
    easing: 'easeInOutCubic'
  },
  rotation: {
    from: currentRotation,
    to: lookAt(planetPosition),
    duration: 1000,
    easing: 'easeInOutCubic'
  },
  fov: {
    from: 60,
    to: 45,
    duration: 800
  }
};
```

### 4.3 UI Animation Guidelines

**Modal Appearances:**
- **Entry**: Scale from 0.8 to 1.0 with fade-in
- **Exit**: Scale to 0.9 with fade-out
- **Duration**: 300ms with easeOutCubic
- **Backdrop**: Blur increase with opacity fade

**Loading States:**
- **Planet Loading**: Fade-in with gentle scale animation
- **Texture Loading**: Progressive appearance with shimmer
- **Error States**: Shake animation with color change

---

## 5. Performance Optimization

### 5.1 Level of Detail (LOD) System

```typescript
interface LODConfig {
  distances: number[];     // Distance thresholds
  geometries: string[];   // Geometry variants (high->low poly)
  textures: string[];     // Texture resolutions
}

const planetLOD: LODConfig = {
  distances: [100, 300, 600, 1200],
  geometries: ['high.json', 'medium.json', 'low.json', 'billboard.json'],
  textures: ['1024px', '512px', '256px', '128px']
};
```

### 5.2 Culling Strategies

**Frustum Culling:**
- Only render objects within camera view
- Implement bounding sphere tests
- Early rejection for distant objects

**Occlusion Culling:**
- Hide planets behind larger objects
- Use simplified geometry for occlusion tests
- Dynamic visibility updates

**Distance Culling:**
- Fade out very distant objects
- Reduce update frequency for far objects
- Skip animation calculations when not visible

### 5.3 Memory Management

**Texture Streaming:**
- Load high-resolution textures on demand
- Unload textures for off-screen objects
- Use compressed formats (ASTC, ETC2)

**Geometry Pooling:**
- Reuse geometry instances for similar objects
- Batch similar objects for reduced draw calls
- Implement object pooling for particles

---

## 6. Implementation Guidelines

### 6.1 Asset Pipeline

**Content Creation Workflow:**
1. **Modeling**: Create base geometry in Blender/Maya
2. **Texturing**: Generate PBR textures in Substance Painter
3. **Optimization**: Reduce polygons, optimize UV mapping
4. **Export**: Convert to React Native compatible formats
5. **Testing**: Validate performance on target devices

**Quality Assurance:**
- Visual consistency across all assets
- Performance impact testing
- Memory usage monitoring
- Loading time optimization

### 6.2 Integration with React Native

**Asset Loading:**
```typescript
import { useAsset } from '@/hooks/useAsset';

const PlanetRenderer = ({ toolId }: { toolId: string }) => {
  const { geometry, textures, loading } = useAsset(`planets/${toolId}`);
  
  if (loading) return <PlaceholderPlanet />;
  
  return (
    <SkiaCanvas>
      <Planet 
        geometry={geometry}
        textures={textures}
        animation={planetAnimations[toolId]}
      />
    </SkiaCanvas>
  );
};
```

**Performance Monitoring:**
```typescript
import { usePerformanceMonitor } from '@/hooks/usePerformanceMonitor';

const GalaxyRenderer = () => {
  const { fps, memoryUsage, renderTime } = usePerformanceMonitor();
  
  // Automatically adjust quality based on performance
  const quality = fps < 45 ? 'low' : fps < 55 ? 'medium' : 'high';
  
  return <Galaxy quality={quality} />;
};
```

---

## 7. Future Enhancements

### 7.1 Advanced Visual Effects

**Planned Features:**
- Real-time reflections on planet surfaces
- Dynamic lighting from nearby "stars"
- Advanced particle physics for debris
- Procedural planet generation for new tools

### 7.2 Interactive Elements

**Enhanced Interactions:**
- Planets that respond to user proximity
- Tool-specific visual behaviors (e.g., coding planets showing code)
- Seasonal events with special visual themes
- User-customizable planet appearances

### 7.3 AR/VR Preparation

**Future Platform Support:**
- Higher polygon counts for VR rendering
- Spatial audio integration
- Hand tracking compatibility
- Haptic feedback integration

---

This specification ensures that all 3D assets in AI GALAXY maintain consistent quality, performance, and visual appeal while providing a solid foundation for future enhancements and platform expansions.