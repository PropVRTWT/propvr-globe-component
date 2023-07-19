<script setup>
import { ref, onMounted,defineEmits ,defineProps} from 'vue';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { CSS2DRenderer } from 'three/examples/jsm/renderers/CSS2DRenderer.js';
import {AmbientLight,WebGLRenderer,Scene,PerspectiveCamera,DirectionalLight} from 'three'
import  ThreeGlobe  from 'three-globe';
const globeViz = ref(null);
const renderers = [new WebGLRenderer(), new CSS2DRenderer()];
const scene = new Scene();
const camera = new PerspectiveCamera();
const props=defineProps(['Data','markerIcon'])
const emits = defineEmits(['emitClickData','ZoomInStop'])
const zoom = ref(1);
const targetZoom = ref(1); // Zoom level to animate to on marker click
const animationDuration = 800; // Duration of the animation in milliseconds
let animationStartTime = 0; // Variable to store the animation start time
const globeOpacity = ref(1);
onMounted(()=>{

  const Globe = new ThreeGlobe()
        .globeImageUrl('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg')
        .bumpImageUrl('https://unpkg.com/three-globe/example/img/earth-topology.png')
        .htmlElementsData(props.Data)
        .htmlElement(d => {
          const el = document.createElement('div');
          el.innerHTML = props.markerIcon;
          el.style.color = d.color;
          el.style.width = `${d.size}px`;
          el.style.pointerEvents = 'auto';
          el.id=d.id
          el.addEventListener('click', event=>handleClick(el.id));
          return el;
        });

  renderers.forEach((r, idx) => {
    r.setSize(window.innerWidth, window.innerHeight);
    if (idx > 0) {
      r.domElement.style.position = 'absolute';
      r.domElement.style.top = '0px';
      r.domElement.style.pointerEvents = 'none';
      r.domElement.style.cursor = 'pointer'
    }
    globeViz.value.appendChild(r.domElement);
  });    

  scene.add(Globe);
  scene.add(new AmbientLight(0xcccccc));
  scene.add(new DirectionalLight(0xffffff, 0.6));

  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  camera.position.z = 500;

  const controls = new OrbitControls(camera, renderers[0].domElement);
  // camera.position.set(0, 0, 7);
  controls.target.set(0, 0, 0);
  controls.enablePan = false;
  controls.minDistance = 300;
  controls.maxDistance = 280;
  controls.enableDamping = true;
  controls.autoRotate = true;
  controls.autoRotateSpeed *= 0.25;


  Globe.setPointOfView(camera.position, Globe.position);
  controls.addEventListener('change', () => Globe.setPointOfView(camera.position, Globe.position));

  (function animate() {
  controls.update();

  // Calculate the animation progress
  const now = Date.now();
  const elapsed = now - animationStartTime;
  const progress = Math.min(elapsed / animationDuration, 1);

  // Apply the zoom animation
  zoom.value = 1 + (targetZoom.value - 1) * progress;
  camera.zoom = zoom.value;
  camera.updateProjectionMatrix();

  // Apply fade-out effect to the globe
  if (progress < 1) {
    const initialOpacity = 1;
    const finalOpacity = 0.3; // Set the final opacity you desire (e.g., 0 for fully transparent, 0.5 for semi-transparent)
    globeOpacity.value = initialOpacity + (finalOpacity - initialOpacity) * progress;
  } else {
    // Reset opacity after animation ends
    globeOpacity.value = 1;
    if (animationStartTime !== 0) {
      emits('ZoomInStop');
      animationStartTime = 0; // Reset animation start time to prevent further console logs
    }
  }

  // Set the opacity of the globe
  Globe.globeMaterial().opacity = globeOpacity.value;

  renderers.forEach((r) => r.render(scene, camera));
  requestAnimationFrame(animate);
})();





  function handleClick(id) {
// Start the animation when a marker is clicked
animationStartTime = Date.now();

// Update the targetZoom based on your marker data or logic
targetZoom.value = 2;

emits('emitClickData', id);
  }

})
</script>

<template>
  <div style="height:100%;width:100%;overflow: hidden;cursor:grab;" id="globeViz" ref="globeViz"></div>
</template>
