<script setup>
  const props = defineProps({
    guesserData: Object,
    question: Object,
  });

  function yeses(guess) {
    return props.question.numYeses[guess.guessId.toString()] || 0;
  }
  function nos(guess) {
    return props.question.numNos[guess.guessId.toString()] || 0;
  }
  function maybes(guess) {
    const responseCount = props.question.numResponses[guess.guessId.toString()] || 0;
    return (responseCount - yeses(guess)) - nos(guess);
  }
</script>

<template>
  <div>
    <h2>{{question.question}}</h2>
    <div class="guess-list">
      <div></div>
      <div class="answer-heading">Yes</div>
      <div class="answer-heading">No</div>
      <div class="answer-heading">Maybe</div>
      <div
          v-for="guess in guesserData.guessArray"
          :key="guess.guessId"
          class="guess-row"
      >
        <div>{{guess.name}}</div>
        <div class="answer-count">{{yeses(guess)}}</div>
        <div class="answer-count">{{nos(guess)}}</div>
        <div class="answer-count">{{maybes(guess)}}</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
  .guess-list {
    display: grid;
    grid-template-columns: max-content 1fr 1fr 1fr;
    overflow: scroll;
    border: 1px solid black;
    margin: 5px;
    height: 400px;
  }
  .guess-row {
    display: contents;
  }
  .answer-heading {
    border-left: 1px solid black;
    text-align: center;
    padding: 0 2px;
  }
  .answer-count {
    border-left: 1px solid black;
    border-top: 1px solid black;
    text-align: right;
    padding: 0 2px;
  }
</style>