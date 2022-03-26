<script setup>
  const props = defineProps({
    guesserData: Object,
  });
</script>

<template>
  <div>
    <h2>Questions</h2>
    <div class="question-list">
      <div
          v-for="(question, index) in guesserData.questions"
          :key="index"
      >{{question.question}}</div>
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
</style>
