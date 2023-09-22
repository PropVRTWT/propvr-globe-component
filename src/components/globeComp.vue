<script setup>
import { ref, onMounted, defineEmits, defineProps, defineExpose, onUnmounted,onBeforeUnmount } from 'vue';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';
import { CSS2DRenderer } from 'three/examples/jsm/renderers/CSS2DRenderer.js';
import {
  WebGLRenderer,
  Scene, PerspectiveCamera,
  TextureLoader, PointLight, Mesh,
  Color,
  MeshStandardMaterial, WebGLCubeRenderTarget, ACESFilmicToneMapping, EquirectangularReflectionMapping, SRGBColorSpace,
  Cache
} from 'three';
import { Lensflare, LensflareElement } from 'three/examples/jsm/objects/Lensflare.js';
import ThreeGlobe from 'three-globe';
import * as TWEEN from '@tweenjs/tween.js'
const globeViz = ref(null);
const globeMesh = ref();
const svgIcon = ref();
let renderers = [new WebGLRenderer(), new CSS2DRenderer()];
let scene = new Scene();
let camera = new PerspectiveCamera();
const props = defineProps(['Data', 'markerIcon', 'settings'])
const emits = defineEmits(['emitClickData', 'loaderImageLoaded', 'initialAnimationEnd', 'selectedAnimationEnd', 'allTextureLoaded']);
const texturePromises = [];
let controls = 0;
let Globe;
let lensflare;


function init(){
  renderers[0].toneMapping = ACESFilmicToneMapping;
  renderers[0].toneMappingExposure = 1.5;
  Globe = new ThreeGlobe()
    .globeImageUrl('/assets/globe/earthTexture.jpg')
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
      Globe.onGlobeReady(() => {
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
  texturePromises.push(new Promise((resolve) => {
    new TextureLoader().load('/assets/globe/earthEmissive.jpg', texture => {
      GlobeMaterial.emissiveMap = texture;
      GlobeMaterial.emissiveIntensity = 0.5;
      GlobeMaterial.emissive = new Color(1, 1, 1)
      resolve();
    });
  }))

  texturePromises.push(new Promise((resolve) => {
    new TextureLoader().load('/assets/globe/earthNormal.jpg', texture => {
      GlobeMaterial.normalMap = texture;
      resolve();
    });
  }))

  texturePromises.push(new Promise((resolve) => {
    new TextureLoader().load('/assets/globe/earthRough.jpg', texture => {
      GlobeMaterial.roughnessMap = texture;
      resolve();
    });
  }))

  texturePromises.push(new Promise((resolve) => {
    new TextureLoader().load("/assets/globe/spaceenv.jpg", texture => {
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
  const textureEquirec = textureLoader.load('/assets/globe/spacebg.jpg');
  textureEquirec.mapping = EquirectangularReflectionMapping;
  textureEquirec.colorSpace = SRGBColorSpace;
  scene.background = textureEquirec;

  const sunLight = new PointLight(0xffffff, 0.2, 20000);
  const sunTextureLoader = new TextureLoader();
  const textureFlare = sunTextureLoader.load("/assets/globe/sun3.webp");
  const textureFlare0 = sunTextureLoader.load('/assets/globe/lensflare.png');
  lensflare = new Lensflare();
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
  controls.screenSpacePanning = false;
  controls.enablePan = false;
  controls.dampingFactor = 0.05;
  controls.minPolarAngle =  1.2;
	controls.maxPolarAngle =  1.2;
  controls.minAzimuthAngle = -0.75;
  controls.maxAzimuthAngle = 1.2;
  controls.enableDamping = true;
  controls.autoRotate = false;
  controls.autoRotateSpeed *= 0.25;



  Globe.setPointOfView(camera.position, Globe.position);
  controls.addEventListener('change', () => Globe.setPointOfView(camera.position, Globe.position));

  Promise.all(texturePromises).then(() => {
    //initial camera and marker animation
    emits("allTextureLoaded");
    startInitialAnimation()
  });

  window.addEventListener('resize', onWindowResize, false);

}

function animate(){
  controls.update();
  renderers.forEach(r => r.render(scene, camera));
  requestAnimationFrame(animate); // enable mouse drag

}
onMounted(() => {
 
  init();
  animate();

})

const handleClick = (id) => {
    emits('emitClickData', id);
}

const animateCameraPosition = (currentPosition, targetPosition, duration, setDistance = true) => {
  return new Promise((resolve) => {
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
  return new Promise((resolve) => {
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

function loaderImageLoaded() {
  emits('loaderImageLoaded')
}

const startAnimate = () => {
  Promise.all([
    animateCameraPosition(camera.position, { "x": 80.66770691334082, "y": 39.40691406419208, "z": 86.68052099645045 }, 2000, false),
    animateCameraPosition(controls.target, { "x": -0.14623203298954673, "y": 37.84872465037691, "z": -0.879793351262042 }, 2000, false),
    animateOpacity(globeViz.value, { opacity: 1 }, { opacity: 0 }, 2000)
  ]).then(() => {
    emits('selectedAnimationEnd');
  })
}

async function startInitialAnimation() {

  Promise.all([
    animateCameraPosition(camera.position, { x: 197.58794914248855, y: 79.45985245939976, z: 211.29395211599356 }, 2000),
    animateCameraPosition(controls.target, { x: 0, y: 0, z: 0 }, 2000),
    animateOpacity(svgIcon.value, { opacity: 0 }, { opacity: 1 }, 2000)]).then(() => {
      emits('initialAnimationEnd');
    })

}


onBeforeUnmount(()=>{

  if (globeMesh.value) {
    scene.traverse(function (obj) {
      if (obj.material) {
        obj.material.dispose();
        if (obj.material.map) {
          obj.material.map.dispose();
        }
        if (obj.material.lightMap) {
          obj.material.lightMap.dispose();
        }
        if (obj.material.aoMap) {
          obj.material.aoMap.dispose();
        }
        if (obj.material.emissiveMap) {
          obj.material.emissiveMap.dispose();
        }
        if (obj.material.bumpMap) {
          obj.material.bumpMap.dispose();
        }
        if (obj.material.normalMap) {
          obj.material.normalMap.dispose();
        }
        if (obj.material.displacementMap) {
          obj.material.displacementMap.dispose();
        }
        if (obj.material.roughnessMap) {
          obj.material.roughnessMap.dispose();
        }
        if (obj.material.metalnessMap) {
          obj.material.metalnessMap.dispose();
        }
        if (obj.material.alphaMap) {
          obj.material.alphaMap.dispose();
        }
      }
      if (obj.geometry) {
        obj.geometry.dispose();
        obj.geometry.attributes.color = {};
        obj.geometry.attributes.normal = {};
        obj.geometry.attributes.position = {};
        obj.geometry.attributes.uv = {};
        obj.geometry.attributes = {};
        obj.material = {};
      }
    })
  }

  for (var elem in Cache.files) {
    Cache.files[elem] = "";
    Cache.remove(elem);
  }

  cancelAnimationFrame(animate);

  renderers[0].dispose();
  controls.dispose();
  TWEEN.removeAll();
  Globe._destructor();
  lensflare.dispose();

  camera = null;
  scene = null;
  Globe = null;
  lensflare = null;
  renderers = [];
  globeMesh.value = null;
  globeMesh.value = null;
  svgIcon.value = null;

  window.removeEventListener('resize', onWindowResize);

  console.log("before Unmounted")
})


onUnmounted(() => {

  console.log("UnMounted")

});

defineExpose({
  startAnimate,
  startInitialAnimation
})

function onWindowResize() {

  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();

  renderers.forEach((r, idx) => {
    r.setSize(window.innerWidth, window.innerHeight);
    if (idx > 0) {
      r.domElement.style.position = 'absolute';
      r.domElement.style.top = '0px';
      r.domElement.style.pointerEvents = 'none';
      r.domElement.style.cursor = 'pointer'
    }
  });
} 

</script>

<template>
  <div style="height:100%;width:100%;overflow: hidden;cursor:grab;" id="globeViz" ref="globeViz"></div>
</template>
