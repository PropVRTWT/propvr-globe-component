<script setup>
import globeComp from './components/globeComp.vue'
const N = 30;
const usedIds = new Set();
const gData = [...Array(N).keys()].map(() => ({
    id: generateRandomId(),
    lat: (Math.random() - 0.5) * 180,
    lng: (Math.random() - 0.5) * 360,
    size: 7 + Math.random() * 30,
    color: ['red', 'white', 'blue', 'green'][Math.round(Math.random() * 3)]
}));

function generateRandomId() {
    return Math.random().toString(36).substr(2, 9);
}

const markerSvg = `<svg viewBox="-4 0 36 36">
  <path fill="currentColor" d="M14,0 C21.732,0 28,5.641 28,12.6 C28,23.963 14,36 14,36 C14,36 0,24.064 0,12.6 C0,5.641 6.268,0 14,0 Z"></path>
  <circle fill="black" cx="14" cy="14" r="7"></circle>
</svg>`;
function emitClickData(id){
  console.log(id)
}
</script>

<template>
  <globeComp :Data="gData" :markerIcon="markerSvg"  @emitClickData="emitClickData" style="height:100%;width:100%;overflow: hidden" />
</template>

