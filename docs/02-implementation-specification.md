# HumSim - Implementation Specification

## Overview
This document provides exact mathematical formulas, visual specifications, performance parameters, and behavioral thresholds for implementing the HumSim based on the Technical Architecture.

## Mathematical Constants and Formulas

### Movement Physics

#### Hover-Dart-Hover Easing
- **Duration**: 800-1200ms total movement time
- **Easing Function**: `easeOutQuart` 
- **Implementation**: `anime({ targets: hummingbird, x: targetX, y: targetY, duration: 1000, easing: 'easeOutQuart' })`
- **Justification**: Quick initial acceleration with smooth deceleration to precise stop

#### Body Orientation System
```javascript
// Speed-based body tilt (upright hover → horizontal flight)
const speed = Math.sqrt(velocity.x ** 2 + velocity.y ** 2);
const bodyAngle = speed * 0.02; // 0 = upright, higher = more horizontal

// Beak direction (always points toward travel)
const beakAngle = Math.atan2(velocity.y, velocity.x);

// Combined orientation
const totalRotation = beakAngle; // Primary rotation
const bodyTilt = bodyAngle;      // Secondary tilt for flight posture
```

#### Turn Rate Calculation
- **Rotation Speed**: 200-300ms to reach new heading
- **Implementation**: Gradual rotation using Anime.js `rotateZ` property
- **Banking Angle**: 5-10 degrees lean into direction changes

#### Edge Constraints
```javascript
// Constrain target position to canvas bounds
const targetPosition = {
  x: Math.max(0, Math.min(canvas.width, mouseX)),
  y: Math.max(0, Math.min(canvas.height, mouseY))
};
```

### Wing Animation System

#### Beat Frequencies
- **Hover State**: 40 BPM (slow, visible individual beats)
- **Flight State**: 120 BPM (fast blur effect)
- **Transition Timing**:
  - Hover→Flight: 150ms ramp up
  - Flight→Hover: 300ms ramp down

#### Wing Animation Properties
```javascript
// Wing rotation cycle
const wingPhase = (Date.now() * wingBPM / 60000) % (2 * Math.PI);
const wingRotation = Math.sin(wingPhase) * 45; // ±45 degree range

// Wing scaling (simulate folding/extending)
const wingScale = 0.8 + Math.abs(Math.sin(wingPhase)) * 0.4; // 0.8 to 1.2

// Wing angle adjustment during turns
const wingAngle = baseAngle + (isturning ? bankingAngle : 0);
```

## Visual Specifications

### Hummingbird Dimensions
```javascript
// Base dimensions (at 1920px screen width) - Polygonal Architecture
const baseDimensions = {
  // Body: Hexagonal or diamond polygon
  bodyWidth: 20,     // Polygon width
  bodyHeight: 30,    // Polygon height
  
  // Wings: Triangular polygons (independently moveable)
  wingBase: 15,      // Triangle base
  wingHeight: 25,    // Triangle height
  
  // Head: Small triangular polygon
  headBase: 8,       // Triangle base for head
  headHeight: 12,    // Triangle height for head
  
  // Tail: Small triangular polygon
  tailBase: 6,       // Triangle base for tail
  tailHeight: 10     // Triangle height for tail
};

// Responsive scaling - maintains mathematical precision
const scale = canvas.width / 1920;
const dimensions = {
  bodyWidth: baseDimensions.bodyWidth * scale,
  bodyHeight: baseDimensions.bodyHeight * scale,
  wingBase: baseDimensions.wingBase * scale,
  wingHeight: baseDimensions.wingHeight * scale,
  headBase: baseDimensions.headBase * scale,
  headHeight: baseDimensions.headHeight * scale,
  tailBase: baseDimensions.tailBase * scale,
  tailHeight: baseDimensions.tailHeight * scale
};

// Design Philosophy: Polygonal minimalism
// - All shapes are polygons with sharp vertices - NO CURVES
// - Each part moves independently during flight
// - Flat design with mathematical precision
// - Clean, anti-aliased boundaries only
```

### Shape Drawing Specifications

#### Body Rendering
```javascript
// Polygonal body (hexagon or diamond) - geometric minimalism
function drawBody(ctx, position, bodyState, dimensions) {
  ctx.save();
  ctx.translate(position.x, position.y);
  ctx.rotate(bodyState.angle);
  ctx.scale(bodyState.scale, Math.cos(bodyState.tilt)); // Tilt effect
  ctx.fillStyle = '#000000'; // Pure black - bold, modern, sophisticated
  
  // Draw hexagonal body (option 1)
  ctx.beginPath();
  const sides = 6;
  const radius = dimensions.bodyWidth / 2;
  for (let i = 0; i < sides; i++) {
    const angle = (i * 2 * Math.PI) / sides;
    const x = radius * Math.cos(angle);
    const y = radius * Math.sin(angle);
    if (i === 0) ctx.moveTo(x, y);
    else ctx.lineTo(x, y);
  }
  ctx.closePath();
  ctx.fill(); // Flat color fill only - no gradients or shadows
  ctx.restore();
}
```

#### Wing Rendering  
```javascript
// Independent triangular wings - each moves separately
function drawWing(ctx, position, wingState, dimensions, isLeft) {
  ctx.save();
  ctx.translate(position.x + wingState.offset.x, position.y + wingState.offset.y);
  ctx.rotate(wingState.angle);
  ctx.scale(wingState.scale, 1);
  ctx.fillStyle = '#000000'; // Pure black - consistent throughout
  
  ctx.beginPath();
  // Isosceles triangles with sharp, precise vertices - no rounded corners
  ctx.moveTo(0, 0); // Attachment point to body
  ctx.lineTo(dimensions.wingBase/2, -dimensions.wingHeight);
  ctx.lineTo(-dimensions.wingBase/2, -dimensions.wingHeight);
  ctx.closePath();
  ctx.fill(); // Solid 100% opacity - flat design aesthetic
  ctx.restore();
}

// Render each wing independently
drawWing(ctx, position, hummingbird.leftWing, dimensions, true);
drawWing(ctx, position, hummingbird.rightWing, dimensions, false);
```

#### Head Rendering
```javascript
// Independent triangular head - points in direction of travel
function drawHead(ctx, position, headState, dimensions) {
  ctx.save();
  ctx.translate(position.x + headState.offset.x, position.y + headState.offset.y);
  ctx.rotate(headState.angle); // Points toward movement direction
  ctx.fillStyle = '#000000'; // Pure black - unified aesthetic
  
  ctx.beginPath();
  // Sharp triangular beak pointing forward
  ctx.moveTo(0, 0); // Attachment point to body
  ctx.lineTo(dimensions.headBase/2, dimensions.headHeight);
  ctx.lineTo(-dimensions.headBase/2, dimensions.headHeight);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}
```

#### Tail Rendering
```javascript
// Independent triangular tail - adjusts during flight dynamics
function drawTail(ctx, position, tailState, dimensions) {
  ctx.save();
  ctx.translate(position.x + tailState.offset.x, position.y + tailState.offset.y);
  ctx.rotate(tailState.angle); // Adjusts for flight dynamics
  ctx.fillStyle = '#000000'; // Pure black - unified aesthetic
  
  ctx.beginPath();
  // Small triangular tail
  ctx.moveTo(0, 0); // Attachment point to body
  ctx.lineTo(dimensions.tailBase/2, -dimensions.tailHeight);
  ctx.lineTo(-dimensions.tailBase/2, -dimensions.tailHeight);
  ctx.closePath();
  ctx.fill();
  ctx.restore();
}
```

### Color Palette
- **Body**: `#000000` (pure black - bold, modern, sophisticated)
- **Head**: `#000000` (same as body for visual unity)
- **Wings**: `#000000` (consistent black throughout)
- **Tail**: `#000000` (unified black aesthetic)
- **Background**: `#FFFFFF` (pure white - maximum contrast with black bird)
- **Motion Blur Overlay**: `rgba(255, 255, 255, 0.15)` (sophisticated motion blur - subtle ghosting effect)

**Design Philosophy**: Bold black polygons on white background for maximum contrast and modern minimalism. No gradients, highlights, or shadows. All polygonal shapes with sharp vertices - NO CURVES. Each part moves independently for dynamic articulation. Pure black chosen for ultimate sophistication and timeless appeal.

### Motion Blur Implementation
```javascript
// Sophisticated motion blur - creates ghosting rather than obvious streaks
ctx.fillStyle = 'rgba(255, 255, 255, 0.15)';
ctx.fillRect(0, 0, canvas.width, canvas.height);

// Then draw hummingbird on top
// Trail persistence: ~6-8 frames before fully faded
// Effect: Suggests motion without overwhelming the geometric forms
// Aesthetic: Subtle, sophisticated trails that maintain design minimalism
```

## Performance Parameters

### Frame Rate Management
```javascript
let lastFrameTime = 0;
const targetFrameTime = 16.67; // 60 FPS

function animationLoop(currentTime) {
  const deltaTime = Math.min(currentTime - lastFrameTime, 50); // Cap at 50ms
  
  if (deltaTime >= targetFrameTime) {
    // Frame drop handling
    if (deltaTime > 33) { // > 30 FPS
      // Skip physics updates, continue rendering
      render();
    } else {
      update(deltaTime);
      render();
    }
    lastFrameTime = currentTime;
  }
  
  requestAnimationFrame(animationLoop);
}
```

### Canvas Optimization Strategy
- **Approach**: Full canvas redraw every frame
- **Justification**: Motion blur overlay requires full canvas treatment
- **Performance Fallback**: If FPS < 45, reduce blur alpha to 0.25

### Memory Management
```javascript
// Reuse single state object
const hummingbirdState = {
  position: { x: 0, y: 0 },
  velocity: { x: 0, y: 0 },
  targetPosition: { x: 0, y: 0 },
  isMoving: false,
  wingPhase: 0,
  bodyAngle: 0,
  beakAngle: 0
};

// Clear Anime.js instances on new targets
let currentAnimation = null;
function startMovement(target) {
  if (currentAnimation) currentAnimation.pause();
  currentAnimation = anime({...});
}
```

## Behavioral Thresholds

### Movement Triggers
```javascript
// Target proximity detection
const HOVER_THRESHOLD = 8; // pixels - switch to hover when within this distance
const MOVEMENT_THRESHOLD = 20; // pixels - minimum mouse movement to trigger flight
const DEAD_ZONE = 5; // pixels - no movement if mouse within this distance

// Mouse movement debouncing
const DEBOUNCE_TIME = 50; // ms - ignore rapid mouse events
let lastMovementTime = 0;

function handleMouseMove(event) {
  const now = Date.now();
  if (now - lastMovementTime < DEBOUNCE_TIME) return;
  
  const distance = Math.sqrt(
    (event.clientX - hummingbird.position.x) ** 2 + 
    (event.clientY - hummingbird.position.y) ** 2
  );
  
  if (distance > MOVEMENT_THRESHOLD) {
    startFlight(event.clientX, event.clientY);
    lastMovementTime = now;
  }
}
```

### State Transition Logic
```javascript
// Movement state detection
function updateMovementState() {
  const distanceToTarget = Math.sqrt(
    (targetPosition.x - position.x) ** 2 + 
    (targetPosition.y - position.y) ** 2
  );
  
  if (distanceToTarget <= HOVER_THRESHOLD) {
    setState('hovering');
  } else if (speed > 0.1) {
    setState('flying');
  }
}

// Wing animation transitions
function updateWingFrequency(state) {
  const targetBPM = state === 'hovering' ? 40 : 120;
  const transitionTime = state === 'hovering' ? 300 : 150; // ms
  
  anime({
    targets: wingAnimation,
    bpm: targetBPM,
    duration: transitionTime,
    easing: 'easeOutQuad'
  });
}
```

### Turn vs New Flight Decision
```javascript
const TURN_THRESHOLD = Math.PI / 6; // 30 degrees - angle change needed to trigger turn
const MAX_TURN_ANGLE = Math.PI / 3; // 60 degrees - beyond this, start new flight

function handleDirectionChange(newTarget) {
  const currentAngle = Math.atan2(velocity.y, velocity.x);
  const newAngle = Math.atan2(newTarget.y - position.y, newTarget.x - position.x);
  const angleDifference = Math.abs(newAngle - currentAngle);
  
  if (angleDifference > MAX_TURN_ANGLE) {
    startNewFlight(newTarget);
  } else if (angleDifference > TURN_THRESHOLD) {
    startTurn(newAngle);
  }
}
```

### Edge Behavior
```javascript
// Constrain movement to canvas bounds
function constrainToCanvas(target) {
  return {
    x: Math.max(0, Math.min(canvas.width - hummingbird.width, target.x)),
    y: Math.max(0, Math.min(canvas.height - hummingbird.height, target.y))
  };
}

// Handle off-screen mouse
function handleMousePosition(mouseX, mouseY) {
  const constrainedTarget = constrainToCanvas({ x: mouseX, y: mouseY });
  
  if (mouseX !== constrainedTarget.x || mouseY !== constrainedTarget.y) {
    // Mouse is off-screen, fly to nearest edge point
    setTarget(constrainedTarget);
  } else {
    setTarget({ x: mouseX, y: mouseY });
  }
}
```

## Initialization Specifications

### Startup State
```javascript
// Initial hummingbird state
const initialState = {
  position: { 
    x: canvas.width / 2, 
    y: canvas.height / 2 
  },
  velocity: { x: 0, y: 0 },
  targetPosition: { 
    x: canvas.width / 2, 
    y: canvas.height / 2 
  },
  isMoving: false,
  wingBPM: 40, // Start in hover state
  bodyAngle: 0, // Upright
  beakAngle: 0 // Facing right by default
};
```

### Canvas Setup
```javascript
// Responsive canvas initialization
function initializeCanvas() {
  canvas.width = window.innerWidth;
  canvas.height = window.innerHeight;
  canvas.style.position = 'fixed';
  canvas.style.top = '0';
  canvas.style.left = '0';
  canvas.style.zIndex = '1000';
  
  // Handle window resize
  window.addEventListener('resize', () => {
    const oldWidth = canvas.width;
    const oldHeight = canvas.height;
    
    canvas.width = window.innerWidth;  
    canvas.height = window.innerHeight;
    
    // Maintain relative position
    hummingbird.position.x *= (canvas.width / oldWidth);
    hummingbird.position.y *= (canvas.height / oldHeight);
  });
}
```

## Implementation Checklist

### Phase 1: Core Infrastructure
- [ ] Canvas setup with responsive sizing
- [ ] Mouse event handling with coordinate mapping
- [ ] Basic animation loop with frame timing
- [ ] Hummingbird state object initialization

### Phase 2: Movement System  
- [ ] Anime.js integration for hover-dart-hover behavior
- [ ] Target constraint system (edge handling)
- [ ] Movement threshold and debouncing logic
- [ ] Basic shape rendering (oval body)

### Phase 3: Orientation & Animation
- [ ] Speed-based body tilt calculation
- [ ] Beak direction pointing system
- [ ] Wing animation with frequency variation
- [ ] Turn vs new flight decision logic

### Phase 4: Visual Polish
- [ ] Motion blur overlay implementation
- [ ] Wing shape rendering with animation
- [ ] Color scheme application
- [ ] Performance optimization and testing

## Testing Criteria

### Functional Requirements
- ✅ Hummingbird follows mouse cursor accurately
- ✅ Hover-dart-hover movement pattern is clearly visible
- ✅ Wing beat frequency changes between hover and flight
- ✅ Body orientation reflects movement direction and speed
- ✅ Motion blur trails are visible during movement
- ✅ Bird stops at canvas edges when mouse goes off-screen

### Performance Requirements
- ✅ Maintains 60 FPS on target browsers
- ✅ Smooth animation with no stuttering
- ✅ Responsive to window resize events
- ✅ Memory usage remains stable over time

### Visual Requirements
- ✅ Procedural shapes render correctly at all screen sizes
- ✅ Colors match specification
- ✅ Motion blur effect is subtle but visible
- ✅ Wing animation appears realistic