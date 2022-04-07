<!--
  --  A "factor" is either a question or a guess. This component displays details for a single
  --  factor, including all values of the other factor and the answers for them. It is passed
  --  several helper functions that allow us to share code between the two different uses
  --  (displaying questions for a guess and displaying guesses for a question).
  -->
<script setup>
  import {ref, watch, onBeforeUnmount} from "vue";
  import FactorDetailsCount from "./FactorDetailsCount.vue";

  const props = defineProps({
    guesserType: String,
    guesserData: Object,
    factorSpec: Object, // contains three fields:
            // factorType: String -- either "question" or "guess"
            // factorId: Number -- the qID or gID of the factor (whichever applies)
            // factor: Object -- if factorType === "question" this is a Question; if factorType === "guess" this is a guess
            // initialMode: String -- either "view" or "edit" or "entry"
  });

  const emit = defineEmits(["exit", "update-guesser"]);

  const mode = ref(props.factorSpec.initialMode);

  watch(mode, () => {
    saveChanges();
  });

  // This has TWO lists of functions to use for the various parts of the page, depending on
  // the value of factorType.
  const functionList = {
    guess: {
      title: props.factorSpec.factor.name,
      verified: props.factorSpec.factor.verified,
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
      verified: props.factorSpec.factor.verified,
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

  const funcs = functionList[props.factorSpec.factorType];
  const title           = funcs["title"];
  const verified        = funcs["verified"];
  const otherFactorList = funcs["otherFactorList"];
  const otherFactorName = funcs["otherFactorName"];
  const yeses           = funcs["yeses"];
  const nos             = funcs["nos"];
  const maybes          = funcs["maybes"];


  const updates = {
    "version": 1,
    "guesserType": props.guesserType,
    "questions": [],
    "guesses": [],
    "answers": []
  };

  function resetUpdates() {
    updates.questions = [];
    updates.guesses = [];
    updates.answers = [];
  }


  // Do this when the component begins to exit
  onBeforeUnmount(() => {
    saveChanges();
  });


  /*
   * Called when it's time to save. If there are no pending updates, this does nothing; if there are
   * any then they get written to the server.
   */
  function saveChanges() {
    // --- decide if we need to save at all ---
    const numUpdates = updates.questions.length + updates.guesses.length + updates.answers.length;
    if (numUpdates === 0) {
      return;
    }

    // --- Make a copy of the updates ---
    // Design note: we want to update the guesser in memory AFTER the fetch completes successfully.
    //   But the updates variable will have been reset by that time so we make a copy of it beforehand.
    //   I had also been worried that the component might have been removed before that happens and
    //   then the update might not happen, but testing shows that the event is still delivered.
    const copyOfUpdates = {...updates};

    // --- Send updates to the server ---
    const guesserType = props.guesserType;
    const updateType = "STRUCTURED_DATA_UPDATE";
    const url = `https://55sksmvv43.execute-api.us-east-1.amazonaws.com/dev/goblini/1/update/${guesserType}/${updateType}`;
    fetch(url, {
      method: "PUT",
      mode: "cors",
      headers: {"Content-Type": "application/json"},
      body: JSON.stringify(updates),
    }).then(() => {
      console.log("saveChanges() completed successfully.");
      emit("update-guesser", copyOfUpdates); // update the local model
    }).catch((error) => {
      console.log("saveChanges() FAILED with error:", error);
    });

    // --- Clear out the updates
    resetUpdates();
  }


  /*
   * Called when the user modifies the verified flag.
   */
  function onVerifiedChange(event) {
    onFactorEdit(undefined, event.target.checked);
  }

  /*
   * Called when the user edits the name of the factor (question or guess).
   */
  function onNameChange(event) {
    onFactorEdit(event.target.value, undefined);
  }

  /*
   * Called when the factor we are editing is modified. Either newName or newVerified
   * can be undefined to indicate that we're leaving that alone.
   */
  function onFactorEdit(newName, newVerified) {
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

    // --- Locate or create the item to update ---
    let item = listToUpdate.find(x => x[idFieldName] === idToUpdate);
    if (item === undefined) {
      // Create new update instruction
      item = {};
      item[idFieldName] = idToUpdate;
      listToUpdate.push(item);
    }

    // --- Edit item ---
    if (newName !== undefined) {
      item[nameFieldName] = newName;
    }
    if (newVerified !== undefined) {
      item.verified = newVerified;
    }
  }


  function onCountInput(event, otherFactorId, otherFactor, answer) {
    const fieldHasFocus = event.target === document.activeElement;
    if (!fieldHasFocus) {
      // They must have clicked to change the value. Since we won't lose focus later
      // we have to apply the edit now.
      onCountBlur(event, otherFactorId, otherFactor, answer);
    }
    // In all other cases, we can wait until they leave ("blur") the field.
  }

  /*
   * Called when the user makes a change to the count of answers for a question/guess pair.
   */
  function onCountBlur(event, otherFactorId, otherFactor, answer) {
    // --- Prohibit the value from going below zero ---
    let intValue = parseInt(event.target.value);
    if (isNaN(intValue) || intValue < 0) {
      intValue = 0
      event.target.value = intValue;
    }

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

    // --- Locate or create the item to update ---
    let item = updates.answers.find(x => x.qID === qIDToUpdate && x.gID === gIDToUpdate);
    if (item === undefined) {
      // Create new answer instruction
      item = {
        "qID": qIDToUpdate,
        "gID": gIDToUpdate,
        counts: [
          yeses(otherFactor),
          nos(otherFactor),
          maybes(otherFactor),
        ],
      };
      updates.answers.push(item);
    }

    // --- Edit item ---
    const countPos = {yes: 0, no: 1, maybe: 2}[answer];
    item.counts[countPos] = parseInt(intValue);
  }


  /*
   * Called when the user selects 'yes', 'no', or 'maybe' during entry of a set of answers.
   * Should generate an update which will set use increment for the answers.
   */
  function onEntry(event, otherFactorId, otherFactor, answer) {
    console.log("onEntry(", event, otherFactorId, otherFactor, answer, ") -> ", event.target.checked, answer, otherFactorId); // FIXME: Return

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

    // --- Locate or create the item to update ---
    let item = updates.answers.find(x => x.qID === qIDToUpdate && x.gID === gIDToUpdate);
    if (item === undefined) {
      // Create new answer instruction
      item = {
        "qID": qIDToUpdate,
        "gID": gIDToUpdate,
      };
      updates.answers.push(item);
    }

    // --- Edit item ---
    item.increments = [
      answer === "yes" ? 1 : 0,
      answer === "no" ? 1 : 0,
      answer === "maybe" ? 1 : 0,
    ];
  }
</script>

<template>
  <div>
    <h2>
      <span v-if="mode === 'view'">{{title}}</span>
      <input v-if="mode !== 'view'" type="text" :value="title" @blur="onNameChange"/>
    </h2>
    <div class="factor-list">
      <div class="other-factor-row first-row">
        <div class="top-left-cell">
          <label>
            <input type="checkbox"
                :checked="verified"
                @change="onVerifiedChange"
                :disabled="mode === 'view'"
            />
            Verified
          </label>
        </div>
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
          <factor-details-count
              :mode="mode"
              answer="yes"
              :value="yeses(otherFactor)"
              :rowNum="index"
              @input="event => onCountInput(event, index, otherFactor, 'yes')"
              @blur="event => onCountBlur(event, index, otherFactor, 'yes')"
              @entry="event => onEntry(event, index, otherFactor, 'yes')"
          />
        </div>
        <div class="answer-count">
          <factor-details-count
              :mode="mode"
              answer="no"
              :value="nos(otherFactor)"
              :rowNum="index"
              @input="event => onCountInput(event, index, otherFactor, 'no')"
              @blur="event => onCountBlur(event, index, otherFactor, 'no')"
              @entry="event => onEntry(event, index, otherFactor, 'no')"
          />
        </div>
        <div class="answer-count">
          <factor-details-count
              :mode="mode"
              answer="maybe"
              :value="maybes(otherFactor)"
              :rowNum="index"
              @input="event => onCountInput(event, index, otherFactor, 'maybe')"
              @blur="event => onCountBlur(event, index, otherFactor, 'maybe')"
              @entry="event => onEntry(event, index, otherFactor, 'maybe')"
          />
        </div>
      </div>
    </div>
    <div class="button-row">
      <button @click="$emit('exit', null)">Done</button>
      <button @click="mode = 'view'" :disabled="mode === 'view'">View</button>
      <button @click="mode = 'edit'" :disabled="mode === 'edit'">Edit</button>
      <button @click="mode = 'entry'" :disabled="mode === 'entry'">Entry</button>
      <button @click="resetUpdates(); $emit('exit', null)">Cancel</button>
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
  .other-factor-row:nth-child(even) div {
    background: var(--color-background-contrast);
  }
  .other-factor-row div {
    border-bottom: 1px solid var(--color-border);
  }
  .other-factor-row.first-row div {
    position: sticky;
    top: 0;
    z-index: 1;
    background: var(--color-background-soft);
    border-bottom: 1px solid var(--color-border);
  }
  .answer-heading {
    border-left: 1px solid var(--color-border);
    border-top: 1px solid var(--color-border);
    text-align: center;
    padding: 0 2px;
  }
  .answer-heading:last-of-type {
    border-right: 1px solid var(--color-border);
  }
  .answer-count {
    border-left: 1px solid var(--color-border);
    text-align: right;
    padding: 0;
  }
  .answer-count:last-of-type {
    border-right: 1px solid var(--color-border);
  }
  .other-factor-cell {
    border-left: 1px solid var(--color-border);
    padding: 0 2px;
    max-width: 400px;
  }
  .button-row {
    margin-top: 6px;
    margin-left: 2px;
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
  }
  .button-row button {
    margin: 0 3px;
  }
</style>
