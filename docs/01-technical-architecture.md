# HumSim - Technical Architecture Document

## Overview
This document defines the critical architectural decisions and technical implementation strategy for the HumSim, based on the requirements outlined in the PRD.

## Critical Architectural Decisions

### 1. Animation Loop Structure
**Decision**: Single unified `requestAnimationFrame` loop

**Implementation**:
- One master loop handles physics updates, wing animation, and rendering
- Each frame: update physics → update animations → render to canvas
- 60 FPS target with frame time delta calculations

**Justification**:
- Ensures perfect synchronization between all systems
- Easier to maintain consistent frame rate
- Simpler debugging when everything updates in lockstep
- `requestAnimationFrame` is specifically designed for this pattern

### 2. State Management
**Decision**: Simple object-based state tracking

**Implementation**:
```javascript
hummingbirdState = {
  position: {x, y},
  velocity: {x, y},
  targetPosition: {x, y},
  isMoving: boolean,
  wingPhase: number,
  bodyAngle: number
}
```

**Justification**:
- PRD only specifies hover-dart-hover behavior - formal state machine is overkill
- Easier to implement smooth transitions with Anime.js
- Lower complexity = fewer bugs and better performance
- Can refactor to state machine later if needed

### 3. Motion Blur Implementation
**Decision**: Canvas trails using semi-transparent overlay

**Implementation**:
- Each frame: apply semi-transparent white `fillRect()` overlay
- Draw current hummingbird on top
- Creates natural wing blur streaks over time

**Justification**:
- Matches PRD specification exactly ("semi-transparent overlay instead of full canvas clear")
- Most performant - no extra draw calls or complex effects
- Creates authentic wing blur streaks naturally
- Broad browser compatibility

### 4. Input Handling Strategy
**Decision**: Event-driven mouse position tracking

**Implementation**:
- `mousemove` event listener updates global cursor position variable
- Animation loop reads current target position each frame
- No polling or throttling needed

**Justification**:
- Most responsive - immediate reaction to mouse movement
- Most efficient - only updates when mouse actually moves
- Standard web pattern with excellent browser support
- Fits PRD requirement for "immediate response" and "real-time" tracking

### 5. Coordinate System
**Decision**: Direct canvas coordinates throughout

**Implementation**:
- Mouse events provide canvas-relative coordinates
- All calculations use canvas pixel coordinates
- No coordinate transformations needed

**Justification**:
- Simplest implementation with best performance
- PRD explicitly states "canvas coordinates mapped to screen coordinates"
- Easier debugging - dev tools coordinates match code coordinates
- Responsive behavior handled by canvas resize events

### 6. Animation Timing & Physics Integration
**Decision**: Hybrid approach using Anime.js + custom physics

**Implementation**:
- Anime.js handles hover-dart-hover movement easing curves
- Custom physics calculate body orientation and wing speed variations
- Wing animation runs independently with speed modulation

**Justification**:
- Anime.js excels at smooth easing curves (matches PRD requirement)
- Custom physics needed for realistic body tilt and wing behavior
- Maintains performance while achieving authentic flight physics
- Separates concerns: movement smoothness vs flight realism

### 7. Canvas Clearing Strategy
**Decision**: Semi-transparent white overlay using `fillRect()`

**Implementation**:
```javascript
// Each frame:
ctx.fillStyle = 'rgba(255, 255, 255, 0.1)';
ctx.fillRect(0, 0, canvas.width, canvas.height);
// Then draw hummingbird
```

**Justification**:
- Most reliable cross-browser behavior
- PRD specifies "semi-transparent overlay instead of full canvas clear"
- Single canvas keeps complexity low
- Precise control over blur trail persistence

## System Architecture

### Core Components

#### 1. Animation Engine
- **Master Loop**: Single `requestAnimationFrame` loop
- **Frame Timing**: Delta time calculations for consistent animation
- **Update Order**: Physics → Animation → Rendering

#### 2. Physics System
- **Movement**: Anime.js easing for hover-dart-hover behavior
- **Orientation**: Custom calculations for body angle based on velocity
- **Wing Dynamics**: Speed-based wing beat frequency modulation

#### 3. Rendering Pipeline
- **Canvas Setup**: Fullscreen responsive canvas (100vw × 100vh)
- **Motion Blur**: Semi-transparent overlay technique
- **Shape Drawing**: Procedural geometric shapes (oval body, triangle wings)

#### 4. Input System
- **Mouse Tracking**: Event-driven position updates
- **Target Setting**: Immediate target position updates
- **Coordinate Mapping**: Direct canvas coordinate usage

### Data Flow

1. **Input**: Mouse move event → Update target position
2. **Physics**: Calculate movement toward target using Anime.js easing
3. **Animation**: Update wing phase and body orientation
4. **Rendering**: Apply motion blur overlay → Draw hummingbird
5. **Loop**: Repeat at 60 FPS

### Performance Considerations

#### Frame Rate Management
- Target 60 FPS with `requestAnimationFrame`
- Delta time calculations for frame rate independence
- Efficient canvas operations (minimal draw calls)

#### Memory Optimization
- Single canvas element
- Minimal object creation per frame
- Reuse animation objects where possible

#### Browser Compatibility
- HTML5 Canvas 2D API (modern browser requirement)
- Standard mouse events
- `requestAnimationFrame` with fallback

## Implementation Phases Alignment

### Phase 1: Core Infrastructure
- Implement unified animation loop
- Set up canvas and coordinate system
- Basic mouse input handling

### Phase 2: Movement System
- Integrate Anime.js for hover-dart-hover behavior
- Implement physics calculations
- Add target following logic

### Phase 3: Visual Systems
- Add wing animation with speed variation
- Implement body orientation
- Create procedural shape rendering

### Phase 4: Motion Blur & Polish
- Implement canvas trail motion blur
- Performance optimization
- Cross-browser testing

## Technical Constraints Compliance

- ✅ Browser-only implementation
- ✅ Single HTML file architecture
- ✅ Only Anime.js external dependency
- ✅ Offline functionality after initial load
- ✅ 60 FPS performance target
- ✅ Modern browser Canvas API usage

## Success Metrics

- Consistent 60 FPS performance
- Smooth hover-dart-hover movement patterns
- Realistic wing motion blur effects
- Immediate mouse response (< 16ms latency)
- Stable performance across screen sizes