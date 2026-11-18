---
name: react-three-fiber
description: Build 3D scenes in React with React Three Fiber (@react-three/fiber), @react-three/drei and three.js. Use when the user asks for 3D graphics, WebGL, three.js in React, R3F, Canvas scenes, models (GLTF/GLB), cameras, lights, materials, shaders or animation in a React app.
---

# React Three Fiber

React Three Fiber (R3F) is a React renderer for three.js. Every three.js class is available as a lowercase JSX element (`new THREE.Mesh()` becomes `<mesh />`), and constructor arguments go in `args`.

## Setup

```bash
npm install three @react-three/fiber @react-three/drei
npm install -D @types/three   # TypeScript projects
```

## Minimal scene

```jsx
import { Canvas } from '@react-three/fiber'
import { OrbitControls } from '@react-three/drei'

export default function App() {
  return (
    <div style={{ width: '100vw', height: '100vh' }}>
      <Canvas camera={{ position: [3, 3, 3], fov: 50 }}>
        <ambientLight intensity={0.5} />
        <directionalLight position={[5, 5, 5]} />
        <mesh>
          <boxGeometry args={[1, 1, 1]} />
          <meshStandardMaterial color="orange" />
        </mesh>
        <OrbitControls />
      </Canvas>
    </div>
  )
}
```

The `<Canvas>` fills its parent, so the parent must have a size.

## Core rules

- **Hooks only inside `<Canvas>`.** `useFrame`, `useThree` and `useLoader` work only in components rendered inside the Canvas.
- **Animate with `useFrame` and refs, not state.** Mutate `ref.current` each frame; calling `setState` 60 times a second re-renders React.

  ```jsx
  function Spinner() {
    const ref = useRef()
    useFrame((state, delta) => { ref.current.rotation.y += delta })
    return <mesh ref={ref}><boxGeometry /><meshNormalMaterial /></mesh>
  }
  ```

- **Props map to properties.** `position={[x, y, z]}`, `rotation-x={Math.PI / 2}` (dash notation sets nested properties), `attach="material"` to attach a child to a parent property.
- **Wrap async loading in `<Suspense>`.** `useGLTF`, `useTexture` and `useLoader` suspend.
- **Reuse geometries and materials** with `useMemo`, or use `<Instances>` / `instancedMesh` for many copies of one object.
- **Events** work on meshes: `onClick`, `onPointerOver`, `onPointerOut`; call `e.stopPropagation()` to stop hits on objects behind.

## Loading models

```jsx
import { Suspense } from 'react'
import { useGLTF } from '@react-three/drei'

function Model(props) {
  const { scene } = useGLTF('/model.glb')
  return <primitive object={scene} {...props} />
}
useGLTF.preload('/model.glb')

// <Suspense fallback={null}><Model /></Suspense>
```

`npx gltfjsx model.glb` turns a model into a reusable JSX component.

## Useful drei helpers

| Helper | Use |
| --- | --- |
| `OrbitControls` | Mouse orbit / zoom / pan |
| `Environment` | HDRI lighting and reflections (`preset="city"`) |
| `useGLTF`, `useTexture` | Load models and textures |
| `Html` | Place DOM elements in 3D space |
| `Text` | SDF text in the scene |
| `ContactShadows`, `Sky`, `Stars` | Quick staging |
| `Bounds`, `Center` | Fit and center content |

## Performance checklist

- Set `dpr={[1, 2]}` on `<Canvas>` to cap pixel ratio.
- Use `frameloop="demand"` for static scenes and call `invalidate()` when something changes.
- Prefer instancing for repeated objects; merge static geometry.
- Compress models (Draco / Meshopt) and textures (KTX2).
- Dispose of nothing manually: R3F disposes objects when they unmount.

## Common mistakes

- Using `useFrame` outside the Canvas → "Hooks can only be used within the Canvas component".
- A Canvas with zero height → blank screen.
- Black meshes with `meshStandardMaterial` → no lights in the scene.
- Mixing versions: R3F v8 pairs with React 18, R3F v9 with React 19.
