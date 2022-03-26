<script setup>
  import {computed} from 'vue';

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
    return (responseCount - yeses) - nos;
  });
</script>

<template>
  <td>
    <div v-if="yeses > 0">Y: {{yeses}}</div>
    <div v-if="nos > 0">N: {{nos}}</div>
    <div v-if="maybes > 0">M: {{maybes}}</div>
  </td>
</template>

<style scoped>
</style>
