<script setup>
import globeComp from './components/globeComp.vue'
import { ref } from 'vue';
const markerIcon = ref(false);

let requestOptions = {
      method: "GET",
      redirect: "follow",
    };

let url = 'https://storagecdn.propvr.ai/DevAssets%2FDigitalTwinApp%2FCoordinates_Icon.svg?alt=media';
fetch(url, requestOptions)
  .then((response) => response.text())
    .then((result) => {
        markerIcon.value = result;
      })
    .catch((error) => console.log("error", error));

const gData = ref([
      {
        id:"88115",
        "lat" : 23.677746332552502,
        "lng" : 44.06750079798167,
        "name":"Saudi Arabia",
        "size":"fit-content",
      }

])

const settings = {
  "hoverColor":"",
  "defaultColor":"",
  "textClass":"white h4 m-0",
}

function emitClickData(id){
  console.log(id)
}

function imageloaded(){
  
}
</script>

<template>
  <globeComp v-if="markerIcon" :Data="gData" :settings="settings" :markerIcon="markerIcon" @loaderImageLoaded="imageloaded"  @emitClickData="emitClickData" style="height:100%;width:100%;overflow: hidden" />
</template>

<style>
.h4{
  font-size: min(max(14px, calc(0.875rem + ((1vw - 3px) * 0.1327))), 17px);
}
.white{
 color: #fff;
}
.m-0{
  margin:0 !important;
}
</style>

