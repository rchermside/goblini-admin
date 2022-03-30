<script setup>
  import {computed} from 'vue';
  import TriColorBar from "./TriColorBar.vue";

  const props = defineProps({
    questionId: Number,
    question: Object,
    guess: Object
  });

  // Each is a number
  const yeses = computed(() => {
    return props.question.numYeses[props.guess.guessId.toString()] || 0;
  });
  const nos = computed(() => {
    return props.question.numNos[props.guess.guessId.toString()] || 0;
  });
  const maybes = computed(() => {
    const responseCount = props.question.numResponses[props.guess.guessId.toString()] || 0;
    return (responseCount - yeses.value) - nos.value;
  });
</script>

<template>
  <td>
    <div v-if="yeses > 0">Y: {{yeses}}</div>
    <div v-if="nos > 0">N: {{nos}}</div>
    <div v-if="maybes > 0">M: {{maybes}}</div>
    <tri-color-bar v-if="yeses + nos + maybes > 0" :width="50" :height="8" :green-val="yeses" :yellow-val="maybes" :red-val="nos"/>
  </td>
</template>

<style scoped>
</style>
