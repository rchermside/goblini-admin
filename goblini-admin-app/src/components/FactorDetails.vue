<!--
  --  A "factor" is either a question or a guess. This component displays details for a single
  --  factor, including all values of the other factor and the answers for them. It is passed
  --  several helper functions that allow us to share code between the two different uses
  --  (displaying questions for a guess and displaying guesses for a question).
  -->
<script setup>
  const props = defineProps({
    guesserData: Object,
    factorType: String, // either "question" or "guess"
    factor: Object, // if factorType === "question" this is a Question; if factorType === "guess" this is a guess
  });

  // This has TWO lists of functions to use for the various parts of the page, depending on
  // the value of factorType.
  const functionList = {
    guess: {
      title: props.factor.name,
      otherFactorList: props.guesserData.questions,
      otherFactorName: function(question) {
        return question.question;
      },
      yeses: function(question) {
        return question.numYeses[props.factor.guessId.toString()] || 0;
      },
      nos: function(question) {
        return question.numNos[props.factor.guessId.toString()] || 0;
      },
      maybes: function(question) {
        const responseCount = question.numResponses[props.factor.guessId.toString()] || 0;
        return (responseCount - yeses(question)) - nos(question);
      }
    },
    question: {
      title: props.factor.question,
      otherFactorList: props.guesserData.guessArray,
      otherFactorName: function(guess) {
        return guess.name;
      },
      yeses: function(guess) {
        return props.factor.numYeses[guess.guessId.toString()] || 0;
      },
      nos: function(guess) {
        return props.factor.numNos[guess.guessId.toString()] || 0;
      },
      maybes: function(guess) {
        const responseCount = props.factor.numResponses[guess.guessId.toString()] || 0;
        return (responseCount - yeses(guess)) - nos(guess);
      }
    },
  };

  const title = functionList[props.factorType]["title"];
  const otherFactorList = functionList[props.factorType]["otherFactorList"];
  const otherFactorName = functionList[props.factorType]["otherFactorName"];
  const yeses = functionList[props.factorType]["yeses"];
  const nos = functionList[props.factorType]["nos"];
  const maybes = functionList[props.factorType]["maybes"];
</script>

<template>
  <div>
    <h2>{{title}}</h2>
    <div class="factor-list">
      <div class="other-factor-row">
        <div></div>
        <div class="answer-heading">Yes</div>
        <div class="answer-heading">No</div>
        <div class="answer-heading">Maybe</div>
      </div>
      <div
          v-for="(otherFactor, index) in otherFactorList"
          :key="index"
          class="other-factor-row"
      >
        <div class="other-factor-cell">{{otherFactorName(otherFactor)}}</div>
        <div class="answer-count">{{yeses(otherFactor)}}</div>
        <div class="answer-count">{{nos(otherFactor)}}</div>
        <div class="answer-count">{{maybes(otherFactor)}}</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
  .factor-list {
    display: grid;
    grid-template-columns: max-content 1fr 1fr 1fr;
    overflow: scroll;
    margin: 5px;
    height: 400px;
  }
  .other-factor-row {
    display: contents;
  }
  .other-factor-row:last-of-type div {
    border-bottom: 1px solid black;
  }
  .answer-heading {
    border-left: 1px solid black;
    border-top: 1px solid black;
    text-align: center;
    padding: 0 2px;
  }
  .answer-heading:last-of-type {
    border-right: 1px solid black;
  }
  .answer-count {
    border-left: 1px solid black;
    border-top: 1px solid black;
    text-align: right;
    padding: 0 2px;
  }
  .answer-count:last-of-type {
    border-right: 1px solid black;
  }
  .other-factor-cell {
    border-top: 1px solid black;
    border-left: 1px solid black;
    max-width: 400px;
  }
</style>