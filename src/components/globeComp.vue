<script setup>
import { ref, onMounted, defineEmits, defineProps,defineExpose } from 'vue';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { CSS2DRenderer } from 'three/examples/jsm/renderers/CSS2DRenderer.js';
import {
  WebGLRenderer,
  Scene, PerspectiveCamera,
  TextureLoader, PointLight,Mesh,
  Color,
  MeshStandardMaterial, WebGLCubeRenderTarget, ACESFilmicToneMapping, EquirectangularReflectionMapping, SRGBColorSpace
} from 'three';
import { Lensflare, LensflareElement } from 'three/examples/jsm/objects/Lensflare.js';
import ThreeGlobe from 'three-globe';
import * as TWEEN from '@tweenjs/tween.js'
const globeViz = ref(null);
const globeMesh = ref();
const svgIcon = ref();
const renderers = [new WebGLRenderer(), new CSS2DRenderer()];
const scene = new Scene();
const camera = new PerspectiveCamera();
const props = defineProps(['Data', 'markerIcon', 'settings'])
const emits = defineEmits(['emitClickData','loaderImageLoaded', 'initialAnimationEnd', 'selectedAnimationEnd','allTextureLoaded']);
const texturePromises = [];
let controls = 0;

onMounted(() => {
  renderers[0].toneMapping = ACESFilmicToneMapping;
  renderers[0].toneMappingExposure = 1.5;
  const Globe = new ThreeGlobe()
    .globeImageUrl('https://storagecdn.propvr.tech/KAFD_Assets%2FGlobe%2FearthTexture.jpg?alt=media')
    .htmlElementsData(props.Data)
    .htmlElement(d => {
      const el = document.createElement('div');
      svgIcon.value = el;
      svgIcon.value.style.opacity = 0;
      el.innerHTML = props.markerIcon;
      if (d.name) {
        const textChild = document.createElement(`p`);
        textChild.innerHTML = d.name;
        props.settings.textClass.split(" ").map(className => {
          textChild.classList.add(className)
        })

        el.appendChild(textChild)
      }

      // el.style.color = d.color;
      el.style.width = `${d.size}px`;
      el.style.pointerEvents = 'auto';
      el.id = d.id
      el.addEventListener('click', event => handleClick(el.id));
      return el;
    });
  
  texturePromises.push(
      new Promise((resolve) => {
        Globe.onGlobeReady(()=>{
          resolve();
        })
      })
  );
  Globe.showAtmosphere(true);
  Globe.atmosphereColor("#c6e7f7")
  Globe.atmosphereAltitude(0.15)
  Globe.traverse((node) => {
    if (node instanceof Mesh) {
      globeMesh.value = node;
    }
  })
  globeMesh.value.material = new MeshStandardMaterial();
  const GlobeMaterial = globeMesh.value.material;
  const targetCube = new WebGLCubeRenderTarget(512, 512);
  // GlobeMaterial.color = new Color(0x3a228a);
  texturePromises.push(new Promise((resolve)=>{
    new TextureLoader().load('https://storagecdn.propvr.tech/KAFD_Assets%2FGlobe%2FearthEmissive.jpg?alt=media', texture => {
      GlobeMaterial.emissiveMap = texture;
      GlobeMaterial.emissiveIntensity = 0.5;
      GlobeMaterial.emissive = new Color(1, 1, 1)
      resolve();
    });
  }))

  texturePromises.push(new Promise((resolve)=>{
    new TextureLoader().load('https://storagecdn.propvr.tech/KAFD_Assets%2FGlobe%2FearthNormal.jpg?alt=media', texture => {
      GlobeMaterial.normalMap = texture;
      resolve();
    });
  }))

  texturePromises.push(new Promise((resolve)=>{
    new TextureLoader().load('https://storagecdn.propvr.tech/KAFD_Assets%2FGlobe%2FearthRough.jpg?alt=media', texture => {
      GlobeMaterial.roughnessMap = texture;
      resolve();
    });
  }))

  texturePromises.push(new Promise((resolve)=>{
    new TextureLoader().load("https://storagecdn.propvr.tech/KAFD_Assets%2FGlobe%2Fspaceenv.jpg?alt=media", texture => {
      // create a cube texture from the panorama
      let cubeTex = targetCube.fromEquirectangularTexture(renderers[0], texture);
      GlobeMaterial.envMap = cubeTex.texture;
      //  GlobeMaterial.envMap.rotation=45
      resolve();
    })
  }))
  
  GlobeMaterial.roughness = 0.5
  //GlobeMaterial.metalness=0.1
  GlobeMaterial.envMapIntensity = 2

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

  const textureLoader = new TextureLoader();
  const textureEquirec = textureLoader.load('https://storagecdn.propvr.tech/KAFD_Assets%2FGlobe%2Fspacebg.jpg?alt=media');
  textureEquirec.mapping = EquirectangularReflectionMapping;
  textureEquirec.colorSpace = SRGBColorSpace;
  scene.background = textureEquirec;

  const sunLight = new PointLight(0xffffff, 0.2, 20000);
  const sunTextureLoader = new TextureLoader();
  const textureFlare = sunTextureLoader.load("https://storagecdn.propvr.ai/DevAssets%2Fsun3.webp?alt=media");
  const textureFlare0 = sunTextureLoader.load('https://storagecdn.propvr.ai/DevAssets%2Flensflare.png?alt=media');
  const lensflare = new Lensflare();
  lensflare.addElement(new LensflareElement(textureFlare, 512, 0));
  lensflare.addElement(new LensflareElement(textureFlare0, 512, 0));
  sunLight.position.set(250, 90, 50);
  sunLight.add(lensflare);
  scene.add(sunLight);

  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  camera.position.z = 500;

  controls = new OrbitControls(camera, renderers[0].domElement);
  camera.position.set(131.88064239814327, 64.1848257090222, 120.89015872538002);
  controls.target.set(-13.657160256049494, 14.83606980678348, 121.60382806417424);
  //  controls.target.set(0,0,0)
  controls.enablePan = false;

  controls.enableDamping = true;
  controls.autoRotate = false;
  controls.autoRotateSpeed *= 0.25;



  Globe.setPointOfView(camera.position, Globe.position);
  controls.addEventListener('change', () => Globe.setPointOfView(camera.position, Globe.position));

  

  (function animate() {
    controls.update();
    renderers.forEach(r => r.render(scene, camera));
    requestAnimationFrame(animate); // enable mouse drag
  })();

  //animate to city
  const handleClick = (id) => {
        emits('emitClickData',id);
  }

  Promise.all(texturePromises).then(() => {
    console.log('All textures loaded successfully!');
    
    //initial camera and marker animation
    emits("allTextureLoaded");
  
  });

})

  const animateCameraPosition = (currentPosition, targetPosition, duration, setDistance = true) => {
    return new Promise((resolve)=>{
      if (!setDistance) {
        controls.minDistance = 0;
        controls.maxDistance = Infinity;
      }
      new TWEEN.Tween(currentPosition)
      .to(targetPosition, duration)
      .easing(TWEEN.Easing.Sinusoidal.InOut)
      .onStart(() => {
        controls.enabled = false;
      })
      .start()
      .onComplete(() => {
        //controls.target.set(targetPosition.x, targetPosition.y, targetPosition.z - 0.001);
        if (setDistance) {
          controls.minDistance = 300;
          controls.maxDistance = 280;
        }
        else {
          controls.minDistance = 0;
          controls.maxDistance = Infinity;
        }
        controls.enabled = true;
        resolve();
      })
    })
  }

  const animateOpacity = (el, currentOpacity, targetOpacity, duration) => {
    return new Promise((resolve)=>{
      new TWEEN.Tween(currentOpacity)
        .to(targetOpacity, duration)
        .easing(TWEEN.Easing.Sinusoidal.InOut)
        .start()
        .onUpdate(() => {
        el.style.opacity = currentOpacity.opacity
        })
        .onComplete(() => {
          el.style.opacity = targetOpacity.opacity;
          resolve();
        })
    })

  }

function loaderImageLoaded(){
  emits('loaderImageLoaded')
}

const startAnimate = ()=>{
  Promise.all([
    animateCameraPosition(camera.position, { "x": 80.66770691334082, "y": 39.40691406419208, "z": 86.68052099645045 }, 2000, false),
    animateCameraPosition(controls.target, { "x": -0.14623203298954673, "y": 37.84872465037691, "z": -0.879793351262042 }, 2000, false),
    animateOpacity(globeViz.value, { opacity: 1 }, { opacity: 0 }, 2000)
    ]).then(()=>{
      emits('selectedAnimationEnd');
  })
}

async function startInitialAnimation(){

  Promise.all([
      animateCameraPosition(camera.position, { x: 197.58794914248855, y: 79.45985245939976, z: 211.29395211599356 }, 2000),
      animateCameraPosition(controls.target, { x: 0, y: 0, z: 0 }, 2000),
      animateOpacity(svgIcon.value, { opacity: 0 }, { opacity: 1 }, 2000)]).then(()=>{
        emits('initialAnimationEnd');
      })

}

defineExpose({
  startAnimate,
  startInitialAnimation
})

</script>

<template>
    <div style="height:100%;width:100%;overflow: hidden;cursor:grab;" id="globeViz" ref="globeViz"></div>
</template>
