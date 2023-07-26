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
  Color,
MeshStandardMaterial,WebGLCubeRenderTarget, ACESFilmicToneMapping, CineonToneMapping, RepeatWrapping, EquirectangularReflectionMapping, SRGBColorSpace
  } from 'three';
import { Lensflare,LensflareElement } from 'three/examples/jsm/objects/Lensflare.js';
import  ThreeGlobe  from 'three-globe';
import * as TWEEN from '@tweenjs/tween.js'
const globeViz = ref(null);
const globeMesh= ref();
const renderers = [new WebGLRenderer(), new CSS2DRenderer()];
const scene = new Scene();
const camera = new PerspectiveCamera();
const props=defineProps(['Data','markerIcon', 'settings'])
const emits = defineEmits(['emitClickData']);
let currentElement = 0;
let startAnimation = false;
let zoom = 1;
let targetZoom = 1; // Zoom level to animate to on marker click
let animationDuration = 800; // Duration of the animation in milliseconds
let animationStartTime = 0; // Variable to store the animation start time
let globeOpacity = 1;

onMounted(()=>{
  renderers[0].toneMapping=ACESFilmicToneMapping;
  renderers[0].toneMappingExposure=1.5;
  const Globe = new ThreeGlobe()
        .globeImageUrl('/earthTexture.jpg')
        // .bumpImageUrl('https://storagecdn.propvr.ai/DevAssets%2FEARTH_Refl.jpg?alt=media')
        .htmlElementsData(props.Data)
        .htmlElement(d => {
          const el = document.createElement('div');
          el.innerHTML = props.markerIcon;
          if(d.name){
            const textChild = document.createElement(`p`);
            textChild.innerHTML = d.name;
            props.settings.textClass.split(" ").map(className=>{
              textChild.classList.add(className)
            })
            
            el.appendChild(textChild)
          }
         
          // el.style.color = d.color;
          el.style.width = `${d.size}px`;
          el.style.pointerEvents = 'auto';
          el.id=d.id
          el.addEventListener('click', event=>handleClick(el.id));
          return el;
        });
Globe.showAtmosphere(true);
Globe.atmosphereColor("#c6e7f7")
Globe.atmosphereAltitude(0.15)
Globe.traverse((node)=>{
if(node instanceof Mesh){
  globeMesh.value=node;
}
})
globeMesh.value.material=new MeshStandardMaterial();
const GlobeMaterial = globeMesh.value.material;
// GlobeMaterial.color = new Color(0x3a228a);
new TextureLoader().load('/earthEmissive.jpg', texture => {
    GlobeMaterial.emissiveMap  = texture;
    GlobeMaterial.emissiveIntensity = 0.5;
    GlobeMaterial.emissive = new Color(1,1,1)
});

new TextureLoader().load('/earthNormal.jpg', texture => {
    GlobeMaterial.normalMap  = texture;
});

new TextureLoader().load('/earthRough.jpg', texture => {
    GlobeMaterial.roughnessMap   = texture;
});

const targetCube = new WebGLCubeRenderTarget(512, 512);
  new TextureLoader().load( "/spaceenv.jpg", texture => {
       // create a cube texture from the panorama
       let cubeTex = targetCube.fromEquirectangularTexture(renderers[0], texture);       
       GlobeMaterial.envMap=cubeTex.texture;
      //  GlobeMaterial.envMap.rotation=45
    })

   
GlobeMaterial.roughness=0.5
//GlobeMaterial.metalness=0.1
GlobeMaterial.envMapIntensity=2

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
  //scene.add(new AmbientLight(0xcccccc));
  // scene.add(new DirectionalLight(0xffffff, 0.6));

  // const textureLoader = new TextureLoader();
  // const texture = textureLoader.load('https://www.shutterstock.com/image-illustration/360-degree-interstellar-cloud-dust-600w-1970651411.jpg'); 
  // const material = new MeshBasicMaterial({ map: texture });
  // const geometry = new SphereGeometry( 500, 60, 40 );
  // geometry.scale( -1, 1, 1 );
  // const mesh = new Mesh(geometry, material);
  // //console.log(mesh)
  // scene.add(mesh);
  const textureLoader = new TextureLoader();
  const textureEquirec = textureLoader.load( '/spacebg.jpg' );
				textureEquirec.mapping = EquirectangularReflectionMapping;
				textureEquirec.colorSpace = SRGBColorSpace;
        scene.background = textureEquirec;



  const sunLight = new PointLight( 0xffffff, 0.2, 20000 );
  const sunTextureLoader = new TextureLoader();
  const textureFlare = sunTextureLoader.load("https://storagecdn.propvr.ai/DevAssets%2Fsun3.webp?alt=media");
	const textureFlare0 = sunTextureLoader.load( 'https://storagecdn.propvr.ai/DevAssets%2Flensflare.png?alt=media' );
  const lensflare = new Lensflare();
  lensflare.addElement( new LensflareElement( textureFlare, 512, 0 ) );
  lensflare.addElement( new LensflareElement( textureFlare0, 512, 0 ) );
  sunLight.position.set( 250, 90, 50 );
  sunLight.add( lensflare );
  scene.add( sunLight );

  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  camera.position.z = 500;

  const controls = new OrbitControls(camera, renderers[0].domElement);
  console.log(controls)
  camera.position.set(131.88064239814327, 64.1848257090222, 120.89015872538002);
  controls.target.set(-13.657160256049494, 14.83606980678348, 121.60382806417424);
  // controls.target.set(0,0,0)
  // controls.enablePan = false;
  // controls.minDistance = 300;
  // controls.maxDistance = 280;
  // controls.enableDamping = true;
  // controls.autoRotate = false;
  // controls.autoRotateSpeed *= 0.25;


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
