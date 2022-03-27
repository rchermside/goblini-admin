<script setup>
  const props = defineProps({
    guesserData: Object,
    guess: Object,
  });

  function yeses(question) {
    return question.numYeses[props.guess.guessId.toString()] || 0;
  }
  function nos(question) {
    return question.numNos[props.guess.guessId.toString()] || 0;
  }
  function maybes(question) {
    const responseCount = question.numResponses[props.guess.guessId.toString()] || 0;
    return (responseCount - yeses(question)) - nos(question);
  }
</script>

<template>
  <div>
    <h2>{{guess.name}}</h2>
    <div class="question-list">
      <div></div>
      <div class="answer-heading">Yes</div>
      <div class="answer-heading">No</div>
      <div class="answer-heading">Maybe</div>
      <div
          v-for="(question, index) in guesserData.questions"
          :key="index"
          class="question-row"
      >
        <div>{{question.question}}</div>
        <div class="answer-count">{{yeses(question)}}</div>
        <div class="answer-count">{{nos(question)}}</div>
        <div class="answer-count">{{maybes(question)}}</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
  .question-list {
    display: grid;
    grid-template-columns: max-content 1fr 1fr 1fr;
    overflow: scroll;
    border: 1px solid black;
    margin: 5px;
    height: 400px;
  }
  .question-row {
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