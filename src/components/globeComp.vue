<script setup>
import { ref, onMounted,defineEmits ,defineProps} from 'vue';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { CSS2DRenderer } from 'three/examples/jsm/renderers/CSS2DRenderer.js';
import {
  AmbientLight,WebGLRenderer,
  Scene,PerspectiveCamera,DirectionalLight,
  TextureLoader,PointLight,
  MeshBasicMaterial,SphereGeometry,
  Mesh,
  Color
  } from 'three';
import { Lensflare,LensflareElement } from 'three/examples/jsm/objects/Lensflare.js';
import  ThreeGlobe  from 'three-globe';
const globeViz = ref(null);
const renderers = [new WebGLRenderer(), new CSS2DRenderer()];
const scene = new Scene();
const camera = new PerspectiveCamera();
const props=defineProps(['Data','markerIcon'])
const emits = defineEmits(['emitClickData']);
let currentElement = 0;
let startAnimation = false;
let zoom = 1;
let targetZoom = 1; // Zoom level to animate to on marker click
let animationDuration = 800; // Duration of the animation in milliseconds
let animationStartTime = 0; // Variable to store the animation start time
let globeOpacity = 1;

onMounted(()=>{

  const Globe = new ThreeGlobe()
        .globeImageUrl('https://storagecdn.propvr.ai/DevAssets%2Fglobe_v1_image%2Fearth2.jpg?alt=media')
        // .bumpImageUrl('https://storagecdn.propvr.ai/DevAssets%2FEARTH_Refl.jpg?alt=media')
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
// console.log(Globe)
// console.log(Globe.globeMaterial())
const GlobeMaterial = Globe.globeMaterial();
new TextureLoader().load('https://storagecdn.propvr.ai/DevAssets%2Fglobe_v1_image%2Fearthemissive1.jpg?alt=media', texture => {
    GlobeMaterial.emissiveMap  = texture;
    GlobeMaterial.emissiveIntensity = 1;
    GlobeMaterial.emissive = new Color(1,1,1)
});

new TextureLoader().load('https://storagecdn.propvr.ai/DevAssets%2Fglobe_v1_image%2FEarth-normal.jpg?alt=media', texture => {
    GlobeMaterial.normalMap  = texture;
});

new TextureLoader().load('https://storagecdn.propvr.ai/DevAssets%2Fglobe_v1_image%2FRoughness3.jpg?alt=media', texture => {
    GlobeMaterial.roughnessMap   = texture;
});
console.log(GlobeMaterial)
 
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
  // scene.add(new DirectionalLight(0xffffff, 0.6));

  const textureLoader = new TextureLoader();
  const texture = textureLoader.load('https://storagecdn.propvr.ai/DevAssets%2FSpace_4K.jpg?alt=media'); 
  const material = new MeshBasicMaterial({ map: texture });
  const geometry = new SphereGeometry( 500, 60, 40 );
  geometry.scale( -1, 1, 1 );
  const mesh = new Mesh(geometry, material);
  console.log(mesh)
  scene.add(mesh);




  const sunLight = new PointLight( 0xffffff, 0.1, 20000 );
  const sunTextureLoader = new TextureLoader();
  const textureFlare = sunTextureLoader.load("https://storagecdn.propvr.ai/DevAssets%2Fsun3.webp?alt=media");
	const textureFlare0 = sunTextureLoader.load( 'https://storagecdn.propvr.ai/DevAssets%2Flensflare.png?alt=media' );
  const lensflare = new Lensflare();
  lensflare.addElement( new LensflareElement( textureFlare, 512, 0 ) );
  lensflare.addElement( new LensflareElement( textureFlare0, 1024, 0 ) );
  sunLight.position.set( 250, 90, 50 );
  sunLight.add( lensflare );
  scene.add( sunLight );

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
  controls.autoRotate = false;
  controls.autoRotateSpeed *= 0.25;


  Globe.setPointOfView(camera.position, Globe.position);
  controls.addEventListener('change', () => Globe.setPointOfView(camera.position, Globe.position));

  (function animate() {
    controls.update();
    renderers.forEach(r => r.render(scene, camera));
    requestAnimationFrame(animate); // enable mouse drag

    if(startAnimation){
      const now = Date.now();
      const elapsed = now - animationStartTime;
      const progress = Math.min(elapsed / animationDuration, 1);

      // Apply the zoom animation
      zoom = 1 + (targetZoom - 1) * progress;
      camera.zoom = zoom;
      camera.updateProjectionMatrix();

      // Apply fade-out effect to the globe
      if (progress < 1) {
        const initialOpacity = 1;
        const finalOpacity = 0.3; // Set the final opacity you desire (e.g., 0 for fully transparent, 0.5 for semi-transparent)
        globeOpacity = initialOpacity + (finalOpacity - initialOpacity) * progress;
      } 
      else {
        // Reset opacity after animation ends
        globeOpacity = 1;
        if (animationStartTime !== 0) {
          emits('emitClickData',currentElement);
          startAnimation = false;
          animationStartTime = 0; // Reset animation start time to prevent further console logs
        }
      }

      Globe.globeMaterial().opacity = globeOpacity;

    }
  })();

})


function handleClick(id) {
  //  emits('emitClickData',id);
  currentElement = id;
  animationStartTime = Date.now();
  // Update the targetZoom based on your marker data or logic
  targetZoom = 2;
  startAnimation = true;
}
</script>

<template>
  <div style="height:100%;width:100%;overflow: hidden;cursor:grab;" id="globeViz" ref="globeViz"></div>
</template>
