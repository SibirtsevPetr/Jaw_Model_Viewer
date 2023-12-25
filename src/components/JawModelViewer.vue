<template>
  <div class="controls">
    <button @click="toggleJaw">{{ jawOpen ? 'Close Jaw' : 'Open Jaw' }}</button>
  </div>
  <div ref="threeContainer" class="three-container"></div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

export default {
  name: 'JawModelViewer',

  data() {
    return {
      lowerJaw: null, // Will hold a reference to the lower jaw model
      jawOpen: false, // Track if the jaw is open
    };
  },

  mounted() {
    // Initialize the Three.js scene when the component is mounted
    this.initThreeJS();
  },

  methods: {
    initThreeJS() {
      // Creating the main scene
      const scene = new THREE.Scene();

      // Setting up the camera with perspective view
      const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

      // Creating the WebGL renderer
      const renderer = new THREE.WebGLRenderer();
      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setClearColor(0xeeeeee); // Setting a light grey background color

      // Mounting the renderer's DOM element to the Vue component
      this.$refs.threeContainer.appendChild(renderer.domElement);

      // Adding ambient light (soft, from all directions) otherwise teeth are black
      const ambientLight = new THREE.AmbientLight(0xffffff); // White light
      scene.add(ambientLight);

      // Setting the camera's position
      camera.position.set(0, 100, 0); // Positioned to view both jaws

      // OrbitControls setup for interactive scene manipulation (rotate, zoom, pan)
      const controls = new OrbitControls(camera, renderer.domElement);
      controls.enableDamping = true;
      controls.dampingFactor = 0.05;

      // Adding directional light for better illumination
      const directionalLight = new THREE.DirectionalLight(0xffffff, 1);
      directionalLight.position.set(0, 1, 1);
      scene.add(directionalLight);

      // Adding hemisphere light for soft, ambient lighting
      const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444444);
      hemiLight.position.set(0, 20, 0);
      scene.add(hemiLight);

      // Setting up the GLTF loader to load 3D models
      const loader = new GLTFLoader();

      // Loading the upper jaw model, rotating it, and adding to the scene
      loader.load('/glb/upper-jaw-vertical.glb', (gltf) => {
        //gltf.scene.position.set(0, 0, -10); // Adjusted position for visibility
        scene.add(gltf.scene);
      });

      // Loading the lower jaw model and positioning it
      loader.load('/glb/lower-jaw-vertical.glb', (gltf) => {
        gltf.scene.rotation.y = Math.PI; // Flipping the lower jaw as it's upside down
        this.lowerJaw = gltf.scene; // Store a reference to the lower jaw model
        scene.add(gltf.scene);
      });

      // Animation loop for rendering the scene
      const animate = () => {
        requestAnimationFrame(animate);
        controls.update(); // Required for OrbitControls to function
        renderer.render(scene, camera);
      };
      animate();

      // Handling window resize to adjust camera aspect ratio and renderer size
      const onWindowResize = () => {
        camera.aspect = window.innerWidth / window.innerHeight;
        camera.updateProjectionMatrix();
        renderer.setSize(window.innerWidth, window.innerHeight);
      };
      window.addEventListener('resize', onWindowResize);

      // Cleanup function to remove event listeners and the renderer's DOM element
      this.$once('hook:beforeDestroy', () => {
        window.removeEventListener('resize', onWindowResize);
        this.$refs.threeContainer.removeChild(renderer.domElement);
        // Additional cleanup here if necessary
      });
    },

    toggleJaw() {
      // Check if the lower jaw is available
      if (this.lowerJaw) {
        // Toggle the jaw position
        this.lowerJaw.position.z += this.jawOpen ? -10 : +10;

        // Update the state
        this.jawOpen = !this.jawOpen;
      }
    },
  },
};
</script>

<style>
.controls {
  position: absolute;
  top: 10px;
  left: 10px;
  z-index: 10;
}

.three-container {
  width: 100%;
  height: 100vh;
}
</style>
