<template>
  <div ref="threeContainer" class="three-container"></div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import ControlPanel from "@/components/ControlPanel.vue";
import { ref } from 'vue';

const jawOpen = ref(false);

export default {
  name: 'JawModelViewer',
  components: {ControlPanel},

  props: {
    jawOpen: Boolean
  },
  data() {
    return {
      lowerJaw: null, // Reference to the lower jaw model
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
        gltf.scene.position.set(0, 20, 0); // Adjusted position for visibility
        scene.add(gltf.scene);
      });

      // Loading the lower jaw model and positioning it
      loader.load('/glb/lower-jaw-vertical.glb', (gltf) => {
        gltf.scene.rotation.y = Math.PI; // Flipping the lower jaw as it's upside down
        gltf.scene.position.set(0, 20, 0); // Adjusted position for visibility
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

    // Method to update jaw position (moved from toggleJaw)
    updateJawPosition() {
      if (this.lowerJaw) {
        this.lowerJaw.position.z = this.jawOpen ? 10 : 0;
      }
    }
  },
  watch: {
    jawOpen() {
      this.updateJawPosition();
    }
  }
};

function toggleJaw() {
  jawOpen.value = !jawOpen.value;
}

</script>

<style>
.three-container {
  width: 100%;
  height: 100vh;
}
</style>
