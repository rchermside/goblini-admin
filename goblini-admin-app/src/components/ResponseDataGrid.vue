<script setup>
  import ResponseCell from "./ResponseCell.vue";

  const props = defineProps({
    guesserData: Object,
  });
</script>

<template>
  <div>
    <table class="response-data-grid">
      <tr>
        <td class="fixed-top-row fixed-left-col"></td>
        <th
            v-for="guess in guesserData.guessArray"
            :key="guess.guessId"
            @click="$emit('showGuess', guess)"
            class="fixed-top-row"
        >{{guess.name}}</th>
      </tr>
      <tr
          v-for="(question, questionId) in guesserData.questions"
          :key="questionId"
      >
        <th @click="$emit('showQuestion', question)" class="fixed-left-col">{{question.question}}</th>
        <response-cell
            v-for="guess in guesserData.guessArray"
            :key="guess.guessId"
            :question-id="questionId"
            :question="question"
            :guess="guess"
        />
      </tr>
    </table>
  </div>
</template>

<style scoped>
  table {
    border-collapse: separate;
    border-spacing: 0;
  }
  td, th {
    padding: 1px;
    border-bottom: 1px solid black;
    border-right: 1px solid black;
  }
  th {
    background: lightblue;
    background-clip: padding-box;
    cursor: pointer;
  }
  .fixed-top-row {
    position: sticky;
    top: 0;
    z-index: 1;
  }
  .fixed-left-col {
    position: sticky;
    left: 0;
    z-index: 1;
    width: 100px;
  }
  .fixed-top-row.fixed-left-col {
    z-index: 2;
    background: white;
  }
</style>
