<script setup>
  const props = defineProps({
    guesserData: Object,
  });
</script>

<template>
  <div class="question-list">
    <div
        v-for="(question, index) in guesserData.questions"
        :key="index"
    >
      <button @click="$emit('showQuestion', question)">view</button>
      <span>{{question.question}}</span>
    </div>
  </div>
</template>

<style scoped>
  .question-list {
    display: flex;
    flex-direction: column;
    overflow-y: scroll;
    height: 200px;
    border: 1px solid black;
    margin: 5px;
  }
  button {
    margin-right: 5px;
  }
</style>
