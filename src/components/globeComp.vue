<script setup>
import { ref, onMounted } from 'vue';
import { TrackballControls } from 'three/examples/jsm/controls/TrackballControls.js';
import { CSS2DRenderer } from 'three/examples/jsm/renderers/CSS2DRenderer.js';
import * as THREE from 'three';
import  ThreeGlobe  from 'three-globe';
const globeViz = ref(null);
const renderers = [new THREE.WebGLRenderer(), new CSS2DRenderer()];
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera();

onMounted(()=>{
  const markerSvg = `<svg viewBox="-4 0 36 36">
      <path fill="currentColor" d="M14,0 C21.732,0 28,5.641 28,12.6 C28,23.963 14,36 14,36 C14,36 0,24.064 0,12.6 C0,5.641 6.268,0 14,0 Z"></path>
      <circle fill="black" cx="14" cy="14" r="7"></circle>
    </svg>`;
  const N = 30;
  const gData = [...Array(N).keys()].map(() => ({
        lat: (Math.random() - 0.5) * 180,
        lng: (Math.random() - 0.5) * 360,
        size: 7 + Math.random() * 30,
        color: ['red', 'white', 'blue', 'green'][Math.round(Math.random() * 3)]
    }));

  const Globe = new ThreeGlobe()
        .globeImageUrl('https://unpkg.com/three-globe/example/img/earth-blue-marble.jpg')
        .bumpImageUrl('https://unpkg.com/three-globe/example/img/earth-topology.png')
        .htmlElementsData(gData)
        .htmlElement(d => {
          const el = document.createElement('div');
          el.innerHTML = markerSvg;
          el.style.color = d.color;
          el.style.width = `${d.size}px`;
          return el;
        });

  renderers.forEach((r, idx) => {
    r.setSize(window.innerWidth, window.innerHeight);
    if (idx > 0) {
      r.domElement.style.position = 'absolute';
      r.domElement.style.top = '0px';
      r.domElement.style.pointerEvents = 'none';
    }
    globeViz.value.appendChild(r.domElement);
  });    

  scene.add(Globe);
  scene.add(new THREE.AmbientLight(0xcccccc));
  scene.add(new THREE.DirectionalLight(0xffffff, 0.6));

  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  camera.position.z = 500;

  const tbControls = new TrackballControls(camera, renderers[0].domElement);
  tbControls.minDistance = 101;
  tbControls.rotateSpeed = 5;
  tbControls.zoomSpeed = 0.8;

  Globe.setPointOfView(camera.position, Globe.position);
  tbControls.addEventListener('change', () => Globe.setPointOfView(camera.position, Globe.position));

  (function animate() {
    tbControls.update();
    renderers.forEach(r => r.render(scene, camera));
    requestAnimationFrame(animate);
  })();
})

</script>

<template>
  <div style="height:100%;width:100%;overflow: hidden;" id="globeViz" ref="globeViz"></div>
</template>
