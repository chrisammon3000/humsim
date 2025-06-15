# HumSim - Implementation Plan

## Overview
This document provides a detailed step-by-step implementation plan for the HumSim, based on the PRD, Technical Architecture, and Implementation Specification documents.

## Implementation Phases

### Phase 1: Core Infrastructure (Foundation)

#### Step 1.1: HTML Structure & Canvas Setup
**Objective**: Create the basic HTML structure and responsive canvas foundation

**Tasks**:
- [ ] Create single HTML file with `<!DOCTYPE html>` structure
- [ ] Add fullscreen canvas element with proper styling
- [ ] Include Anime.js CDN link (only external dependency)
- [ ] Set up responsive canvas sizing (100vw × 100vh)
- [ ] Implement window resize event handler to maintain relative positioning
- [ ] Add basic CSS for canvas positioning and overflow handling

**Expected Outcome**: Canvas fills entire browser window and resizes properly

#### Step 1.2: Basic Animation Loop
**Objective**: Establish the core 60 FPS animation loop with frame timing

**Tasks**:
- [ ] Implement unified `requestAnimationFrame` loop
- [ ] Add frame timing with delta time calculations
- [ ] Set up 60 FPS target with frame drop handling (skip physics if >33ms)
- [ ] Create update → render cycle structure
- [ ] Add frame time capping (max 50ms to prevent jumps)
- [ ] Implement basic performance monitoring

**Expected Outcome**: Smooth animation loop running at 60 FPS with proper frame timing

#### Step 1.3: Mouse Input System
**Objective**: Capture and process mouse movement for real-time bird control

**Tasks**:
- [ ] Add `mousemove` event listener for real-time cursor tracking
- [ ] Implement canvas coordinate conversion from mouse events
- [ ] Add edge constraint logic (mouse position clamped to canvas bounds)
- [ ] Set up input debouncing (50ms) to prevent jittery movement
- [ ] Implement movement threshold detection (20px minimum trigger)
- [ ] Add dead zone handling (5px no-movement zone)

**Expected Outcome**: Mouse movement accurately tracked and converted to canvas coordinates

#### Step 1.4: Hummingbird State Object
**Objective**: Initialize the core data structure for hummingbird properties

**Tasks**:
- [ ] Create hummingbird state object with all required properties
- [ ] Initialize position at screen center
- [ ] Set up velocity tracking (x, y components)
- [ ] Add target position management
- [ ] Create wing animation properties (phase, BPM, state)
- [ ] Initialize body orientation tracking (angle, tilt, beak direction)
- [ ] Add movement state flags (isMoving, isHovering)

**Expected Outcome**: Complete state object ready for physics and rendering systems

---

### Phase 2: Movement Physics (Core Behavior)

#### Step 2.1: Anime.js Integration
**Objective**: Implement smooth hover-dart-hover movement using Anime.js

**Tasks**:
- [ ] Integrate Anime.js for position animations
- [ ] Implement hover-dart-hover easing with `easeOutQuart`
- [ ] Set movement duration to 1000ms (adjustable)
- [ ] Handle animation cleanup on new target changes
- [ ] Add movement threshold detection (20px minimum movement)
- [ ] Implement animation completion callbacks for state changes

**Expected Outcome**: Bird smoothly follows mouse with authentic hover-dart-hover pattern

#### Step 2.2: Physics Calculations
**Objective**: Calculate realistic body orientation and movement physics

**Tasks**:
- [ ] Calculate current speed: `Math.sqrt(velocity.x² + velocity.y²)`
- [ ] Implement body tilt formula: `speed * 0.02` (upright → horizontal)
- [ ] Add beak direction calculation: `Math.atan2(velocity.y, velocity.x)`
- [ ] Set up hover detection (within 8px of target position)
- [ ] Implement velocity tracking from position changes
- [ ] Add edge constraint enforcement

**Expected Outcome**: Bird body orientation realistically reflects movement direction and speed

#### Step 2.3: State Management
**Objective**: Track and manage hover vs flight states with proper transitions

**Tasks**:
- [ ] Implement hover vs flight state detection
- [ ] Add movement trigger logic based on mouse distance
- [ ] Handle target switching and animation interruption
- [ ] Implement state transition callbacks
- [ ] Add behavioral threshold enforcement
- [ ] Create state-dependent property updates

**Expected Outcome**: Clean state transitions between hovering and flying modes

---

### Phase 3: Visual Rendering (Shape & Animation)

#### Step 3.1: Basic Shape Rendering
**Objective**: Draw the polygonal hummingbird with independent articulated parts

**Tasks**:
- [ ] Implement hexagonal or diamond body polygon rendering (20×30px base)
- [ ] Create triangular head polygon (8×12px) that rotates independently
- [ ] Create triangular tail polygon (6×10px) that adjusts independently
- [ ] Add body rotation based on overall movement direction
- [ ] Apply speed-based body tilt transformation
- [ ] Set all polygon colors to `#000000` (pure black for unified aesthetic)
- [ ] Ensure maximum contrast with white background
- [ ] Implement responsive scaling with mathematical precision
- [ ] Add proper canvas transformation handling (save/restore) for each part
- [ ] Ensure ALL POLYGONAL SHAPES - NO CURVES anywhere
- [ ] Each part has independent transformation matrix

**Expected Outcome**: Polygonal hummingbird with independently moving parts renders correctly

#### Step 3.2: Wing System
**Objective**: Create clean triangular wings with sharp vertices and geometric precision

**Tasks**:
- [ ] Implement isosceles triangle wing shape rendering (15×25px base, scaled)
- [ ] Ensure sharp vertices with no rounded corners - mathematical precision
- [ ] Add wing positioning relative to body center
- [ ] Create wing rotation cycle using sine wave calculations
- [ ] Implement wing scaling (0.8 to 1.2 range) for folding effect
- [ ] Set wing color to `#4A7C59` (muted lighter green for subtle depth)
- [ ] Add proper wing transformation handling
- [ ] Maintain flat design aesthetic - solid fill only, no gradients

**Expected Outcome**: Wings render as clean geometric triangles with minimalist aesthetic

#### Step 3.3: Independent Part Animation Logic
**Objective**: Implement independent animation for each polygonal part

**Tasks**:
- [ ] **Wing Animation**: Start at 40 BPM (hover), transition to 120 BPM (flight)
- [ ] **Independent Wing Control**: Left and right wings animate separately with slight phase offset
- [ ] **Head Animation**: Triangular head rotates to point toward movement direction
- [ ] **Tail Animation**: Triangular tail adjusts angle based on flight dynamics
- [ ] **Body Animation**: Hexagon/diamond tilts based on speed and direction
- [ ] Add 150ms ramp up time, 300ms ramp down time for wing transitions
- [ ] Calculate independent phases for each part
- [ ] Apply ±45° wing rotation range, smaller ranges for head/tail
- [ ] Implement articulated movement - each part responds to flight state independently

**Expected Outcome**: All polygonal parts move independently creating dynamic, articulated flight behavior

---

### Phase 4: Motion Blur & Polish (Visual Effects)

#### Step 4.1: Motion Blur Implementation
**Objective**: Create sophisticated motion blur that maintains geometric minimalism

**Tasks**:
- [ ] Apply sophisticated semi-transparent overlay: `rgba(255, 255, 255, 0.15)`
- [ ] Implement overlay before each hummingbird render
- [ ] Ensure proper layering (blur → bird)
- [ ] Test trail persistence (should last 6-8 frames)
- [ ] Verify sophisticated ghosting effect during movement
- [ ] Ensure motion blur doesn't overwhelm geometric forms
- [ ] Maintain minimalist aesthetic - subtle trails, not obvious streaks

**Expected Outcome**: Wings show sophisticated motion blur that suggests motion while preserving clean design

#### Step 4.2: Performance Optimization
**Objective**: Ensure stable 60 FPS performance across target devices

**Tasks**:
- [ ] Implement frame rate monitoring and logging
- [ ] Add performance fallback (reduce blur alpha to 0.25 if FPS < 45)
- [ ] Minimize object creation per frame (reuse objects)
- [ ] Optimize canvas operations for efficiency
- [ ] Add memory usage monitoring
- [ ] Implement performance threshold alerts

**Expected Outcome**: Consistent 60 FPS performance with graceful degradation

#### Step 4.3: Final Polish
**Objective**: Complete the user experience with testing and refinement

**Tasks**:
- [ ] Test responsive scaling across various screen sizes
- [ ] Verify edge behavior (bird stops at canvas boundaries)
- [ ] Ensure movement feels natural and authentic
- [ ] Cross-browser compatibility testing (Chrome, Firefox, Safari, Edge)
- [ ] Performance testing on different devices
- [ ] Final visual polish and bug fixes

**Expected Outcome**: Polished, responsive application that meets all PRD requirements

---

## File Structure

### Single HTML File Structure
```
index.html
├── HTML Structure
│   ├── DOCTYPE and meta tags
│   ├── Canvas element
│   └── Anime.js CDN link
├── Embedded CSS
│   ├── Canvas styling (fullscreen)
│   ├── Body/HTML reset
│   └── Responsive design rules
└── Embedded JavaScript
    ├── State management
    ├── Animation loop
    ├── Physics calculations
    ├── Rendering system
    └── Input handling
```

## Implementation Priorities

### Critical Path
1. **Canvas + Animation Loop** - Must work before anything else
2. **Mouse Input + Basic Movement** - Core interaction
3. **Body Orientation** - Makes movement feel realistic
4. **Wing Animation** - Key visual feedback
5. **Motion Blur** - Final polish for realism

### Success Checkpoints

#### Phase 1 Complete
- [ ] Canvas fills screen and resizes properly
- [ ] Animation loop runs smoothly at 60 FPS
- [ ] Mouse position tracked accurately
- [ ] Hummingbird state object initialized

#### Phase 2 Complete
- [ ] Bird follows mouse with smooth easing
- [ ] Hover-dart-hover pattern clearly visible
- [ ] Body orientation reflects movement direction
- [ ] State transitions work correctly

#### Phase 3 Complete
- [ ] Procedural shapes render correctly
- [ ] Body tilts based on speed (upright hover → horizontal flight)
- [ ] Wings beat faster during flight, slower during hover
- [ ] All visual elements scale responsively

#### Phase 4 Complete
- [ ] Motion blur visible during rapid movement
- [ ] 60 FPS maintained across target browsers
- [ ] Edge constraints work properly
- [ ] Application feels polished and realistic

## Testing Strategy

### Continuous Testing Throughout Development
- **Visual Testing**: Does the bird look right at each stage?
- **Performance Testing**: Frame rate monitoring during development
- **Interaction Testing**: Mouse responsiveness and accuracy
- **Responsive Testing**: Behavior across different screen sizes

### Final Testing Checklist
- [ ] All PRD requirements met
- [ ] Technical Architecture decisions implemented correctly
- [ ] Implementation Specification values used accurately
- [ ] Cross-browser compatibility verified
- [ ] Performance targets achieved
- [ ] User experience feels intuitive and engaging

## Risk Mitigation

### Potential Issues & Solutions
- **Performance Problems**: Implement fallback strategies (reduce blur, simplify animation)
- **Browser Compatibility**: Test early and often, use standard APIs
- **Complex Animation Logic**: Build incrementally, test each component
- **Responsive Design Issues**: Test on various screen sizes throughout development

### Development Best Practices
- Commit working code frequently
- Test in target browsers during development
- Keep performance monitoring active
- Document any deviations from specifications
- Maintain clean, readable code structure

Ready to begin implementation following this plan?