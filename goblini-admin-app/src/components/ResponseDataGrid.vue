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
        <td></td>
        <th
            v-for="guess in guesserData.guessArray"
            :key="guess.guessId"
            @click="$emit('showGuess', guess)"
        >{{guess.name}}</th>
      </tr>
      <tr
          v-for="(question, questionId) in guesserData.questions"
          :key="questionId"
      >
        <th @click="$emit('showQuestion', question)">{{question.question}}</th>
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
    border-collapse: collapse;
  }
  td, th {
    padding: 1px;
    border: 1px solid black;
  }
  th {
    background: lightblue;
    background-clip: padding-box;
    cursor: pointer;
  }
</style>
