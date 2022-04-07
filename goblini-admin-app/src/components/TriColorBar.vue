<script setup>
  const props = defineProps({
    width: Number,
    height: Number,
    greenVal: Number,
    yellowVal: Number,
    redVal: Number,
  });
  const pixelsPerCount = props.width / (props.greenVal + props.yellowVal + props.redVal);
  const greenBar = props.greenVal * pixelsPerCount;
  const yellowBar = props.yellowVal * pixelsPerCount;
  const redBar = props.redVal * pixelsPerCount;
</script>

<template>
  <svg :width="width" :height="height">
    <rect x="0" y="0" height="8" :width="greenBar" style="fill: rgb(0,216,0)"/>
    <rect :x="greenBar" y="0" height="8" :width="yellowBar" style="fill: rgb(216,216,0)"/>
    <rect :x="greenBar + yellowBar" y="0" height="8" :width="redBar" style="fill: rgb(216,0,0)"/>
  </svg>
</template>

<style scoped>

</style>