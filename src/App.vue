<script setup>
import { onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'
import InputForm from './components/InputForm.vue'
const initMatrix = () => {
  // Configuration de base
  const scene = new THREE.Scene()
  const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)
  const renderer = new THREE.WebGLRenderer({ alpha: true })
  renderer.setSize(window.innerWidth, window.innerHeight)
  document.getElementById('matrix-background').appendChild(renderer.domElement)

  // Création des caractères Matrix
  const characters = '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ'
  const fontSize = 25
  const columns = Math.floor(window.innerWidth / fontSize)
  const drops = []

  // Création de la texture avec canvas
  const canvas = document.createElement('canvas')
  canvas.width = fontSize * columns
  canvas.height = window.innerHeight
  const context = canvas.getContext('2d')
  context.font = `${fontSize}px monospace`
  
  // Initialisation des gouttes
  for (let i = 0; i < columns; i++) {
    drops[i] = 1
  }

  // Création du matériau et de la géométrie
  const texture = new THREE.CanvasTexture(canvas)
  const material = new THREE.MeshBasicMaterial({
    map: texture,
    transparent: true,
    color: 0x00ff00
  })
  const geometry = new THREE.PlaneGeometry(window.innerWidth, window.innerHeight)
  const plane = new THREE.Mesh(geometry, material)
  scene.add(plane)

  camera.position.z = 500

  // Animation
  const animate = () => {
    requestAnimationFrame(animate)
    context.fillStyle = 'rgba(0, 0, 0, 0.05)'
    context.fillRect(0, 0, canvas.width, canvas.height)
    context.fillStyle = '#0F0'
    
    for (let i = 0; i < drops.length; i++) {
      const text = characters[Math.floor(Math.random() * characters.length)]
      context.fillText(text, i * fontSize, drops[i] * fontSize)
      if (drops[i] * fontSize > canvas.height && Math.random() > 0.975) {
        drops[i] = 0
      }
      drops[i]++
    }
    
    texture.needsUpdate = true
    renderer.render(scene, camera)
  }

  animate()

  // Gestion du redimensionnement
  const handleResize = () => {
    camera.aspect = window.innerWidth / window.innerHeight
    camera.updateProjectionMatrix()
    renderer.setSize(window.innerWidth, window.innerHeight)
  }
  window.addEventListener('resize', handleResize)

  return () => {
    window.removeEventListener('resize', handleResize)
    renderer.dispose()
  }
}

onMounted(() => {
  const cleanup = initMatrix()
  onBeforeUnmount(cleanup)
})
</script>

<template>
  <div id="matrix-background"></div>
  <div class="content">
    <div class="backdrop-container">
      <slot>
        <InputForm />
      </slot>
    </div>
  </div>
</template>

<style scoped>
#matrix-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
}

.content {
  position: relative;
  z-index: 1;
  padding: 1rem;
}

.backdrop-container {
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(8px);
  border-radius: 1rem;
  padding: 1rem;
  max-width: 1200px;
  margin: 1rem auto;
  
  @media (min-width: 640px) {
    padding: 2rem;
    margin: 2rem auto;
  }
}
</style>

