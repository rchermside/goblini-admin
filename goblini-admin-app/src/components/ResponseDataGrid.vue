<script setup>
  import ResponseCell from "./ResponseCell.vue";

  const props = defineProps({
    guesserData: Object,
  });
  const emit = defineEmits(["show-factor"]);

  function onHeaderClick(factorType, factorId, factor) {
    emit("show-factor", {factorType, factorId, factor});
  }
</script>

<template>
  <div>
    <table class="response-data-grid">
      <tr>
        <td class="fixed-top-row fixed-left-col"></td>
        <th
            v-for="guess in guesserData.guessArray"
            :key="guess.guessId"
            @click="onHeaderClick('guess', guess.guessId, guess)"
            class="fixed-top-row"
        >{{guess.name}}</th>
      </tr>
      <tr
          v-for="(question, questionId) in guesserData.questions"
          :key="questionId"
      >
        <th @click="onHeaderClick('question', questionId, question)" class="fixed-left-col">{{question.question}}</th>
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
    border-bottom: 1px solid var(--color-border);
    border-right: 1px solid var(--color-border);
  }
  th {
    background: var(--color-splash);
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
    background: var(--color-background);
  }
</style>
