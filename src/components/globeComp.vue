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
const usedIds = new Set();
const props=defineProps(['gData','markerSvg'])
const emits = defineEmits(['emitClickData'])
onMounted(()=>{

  const Globe = new ThreeGlobe()
        .globeImageUrl('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg')
        .bumpImageUrl('https://unpkg.com/three-globe/example/img/earth-topology.png')
        .htmlElementsData(props.gData)
        .htmlElement(d => {
          let id;
          do {
            id = generateRandomId();
          } while (usedIds.has(id));
          
          usedIds.add(id);
          const el = document.createElement('div');
          el.innerHTML = props.markerSvg;
          el.style.color = d.color;
          el.style.width = `${d.size}px`;
          el.style.pointerEvents = 'auto';
          el.id=id
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
    renderers.forEach(r => r.render(scene, camera));
    requestAnimationFrame(animate);
  })();



  function generateRandomId() {
    return Math.random().toString(36).substr(2, 9);
  }
  function handleClick(id) {
   emits('emitClickData',id);
  }
})
</script>

<template>
  <div style="height:100%;width:100%;overflow: hidden;cursor:grab;" id="globeViz" ref="globeViz"></div>
</template>
