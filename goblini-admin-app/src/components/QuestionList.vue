<script setup>
  const props = defineProps({
    guesserData: Object,
  });
  const emit = defineEmits(["show-factor"]);

  function onButtonClick(index, question) {
    emit("show-factor", {
      factorType: "question",
      factorId: index,
      factor: question,
      initialMode: "view",
    });
  }
</script>

<template>
  <div class="question-list">
    <div
        v-for="(question, index) in guesserData.questions"
        :key="index"
    >
      <button @click="onButtonClick(index, question)">view</button>
      <span>{{question.question}}</span>
    </div>
  </div>
</template>

<style scoped>
  .question-list {
    display: flex;
    flex-direction: column;
    overflow-y: scroll;
    min-height: 200px;
    margin: 5px;
  }
  button {
    margin-right: 5px;
  }
</style>
