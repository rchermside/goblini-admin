<!--
  --  A "factor" is either a question or a guess. This component displays details for a single
  --  factor, including all values of the other factor and the answers for them. It is passed
  --  several helper functions that allow us to share code between the two different uses
  --  (displaying questions for a guess and displaying guesses for a question).
  -->
<script setup>
  const props = defineProps({
    guesserType: String,
    guesserData: Object,
    factorSpec: Object, // contains three fields:
            // factorType: String -- either "question" or "guess"
            // factorId: Number -- the qID or gID of the factor (whichever applies)
            // factor: Object -- if factorType === "question" this is a Question; if factorType === "guess" this is a guess
  });

  // This has TWO lists of functions to use for the various parts of the page, depending on
  // the value of factorType.
  const functionList = {
    guess: {
      title: props.factorSpec.factor.name,
      otherFactorList: props.guesserData.questions,
      otherFactorName: function(question) {
        return question.question;
      },
      yeses: function(question) {
        return question.numYeses[props.factorSpec.factor.guessId.toString()] || 0;
      },
      nos: function(question) {
        return question.numNos[props.factorSpec.factor.guessId.toString()] || 0;
      },
      maybes: function(question) {
        const responseCount = question.numResponses[props.factorSpec.factor.guessId.toString()] || 0;
        return (responseCount - yeses(question)) - nos(question);
      }
    },
    question: {
      title: props.factorSpec.factor.question,
      otherFactorList: props.guesserData.guessArray,
      otherFactorName: function(guess) {
        return guess.name;
      },
      yeses: function(guess) {
        return props.factorSpec.factor.numYeses[guess.guessId.toString()] || 0;
      },
      nos: function(guess) {
        return props.factorSpec.factor.numNos[guess.guessId.toString()] || 0;
      },
      maybes: function(guess) {
        const responseCount = props.factorSpec.factor.numResponses[guess.guessId.toString()] || 0;
        return (responseCount - yeses(guess)) - nos(guess);
      }
    },
  };


  const updates = {
    "version": 1,
    "guesserType": props.guesserType,
    "questions": [],
    "guesses": [],
    "answers": []
  };

  function onFactorEdit(event) {
    // --- Identify the qID or gID we are updating ---
    const idToUpdate = props.factorSpec.factorId;

    // --- Set things that depend on the factorType ---
    let listToUpdate;
    let idFieldName;
    let nameFieldName;
    if (props.factorSpec.factorType === "question") {
      listToUpdate = updates.questions;
      idFieldName = "qID";
      nameFieldName = "question";
    } else if (props.factorSpec.factorType === "guess") {
      listToUpdate = updates.guesses;
      idFieldName = "gID";
      nameFieldName = "guess"
    } else {
      throw new Error("Invalid factorType");
    }

    // --- Locate or create the existingItem to update ---
    let existingItem = listToUpdate.find(x => x[idFieldName] === idToUpdate);
    if (existingItem === undefined) {
      // Create new update instruction
      existingItem = {idFieldName: idToUpdate};
      listToUpdate.push(existingItem);
    }

    // --- Edit existingItem ---
    existingItem[nameFieldName] = event.target.value;

    // --- Logging we will take out later ---
    console.log("onFactorEdit", updates); // FIXME: Remove after the updates actually go through
  }

  function onCountEdit(event, otherFactorId, otherFactor, answer) {
    // --- Identify the qID and gID we are updating ---
    let qIDToUpdate;
    let gIDToUpdate;
    if (props.factorSpec.factorType === "question") {
      qIDToUpdate = props.factorSpec.factorId;
      gIDToUpdate = otherFactorId;
    } else if (props.factorSpec.factorType === "guess") {
      qIDToUpdate = otherFactorId;
      gIDToUpdate = props.factorSpec.factorId;
    } else {
      throw new Error("Invalid factorType");
    }

    // --- Locate or create the existingItem to update ---
    let existingItem = updates.answers.find(x => x.qID === qIDToUpdate && x.gID === gIDToUpdate);
    if (existingItem === undefined) {
      // Create new answer instruction
      existingItem = {
        qID: qIDToUpdate,
        gID: gIDToUpdate,
        counts: [
          yeses(otherFactor),
          nos(otherFactor),
          maybes(otherFactor),
        ],
      };
      updates.answers.push(existingItem);
    }

    // --- Edit existingItem ---
    const countPos = {yes: 0, no: 1, maybe: 2}[answer];
    existingItem.counts[countPos] = parseInt(event.target.value);

    // --- Logging we will take out later ---
    console.log("onFactorEdit", updates); // FIXME: Remove after the updates actually go through
  }


  const funcs = functionList[props.factorSpec.factorType];
  const title           = funcs["title"];
  const otherFactorList = funcs["otherFactorList"];
  const otherFactorName = funcs["otherFactorName"];
  const yeses           = funcs["yeses"];
  const nos             = funcs["nos"];
  const maybes          = funcs["maybes"];
</script>

<template>
  <div>
    <h2>
      <input type="text" :value="title" @blur="onFactorEdit"/>
    </h2>
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
        <div class="answer-count">
          <input type="number"
              :value="yeses(otherFactor)"
              @input="onCountEdit($event, index, otherFactor, 'yes')"
          />
        </div>
        <div class="answer-count">
          <input type="number"
                 :value="nos(otherFactor)"
                 @input="onCountEdit($event, index, otherFactor, 'no')"
          />
        </div>
        <div class="answer-count">
          <input type="number"
                 :value="maybes(otherFactor)"
                 @input="onCountEdit($event, index, otherFactor, 'maybe')"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
  h2 input {
    font-size: 24px;
  }
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
    padding: 0;
  }
  .answer-count:last-of-type {
    border-right: 1px solid black;
  }
  .other-factor-cell {
    border-top: 1px solid black;
    border-left: 1px solid black;
    max-width: 400px;
  }
  .answer-count input {
    width: 4em;
    text-align: right;
  }
</style>