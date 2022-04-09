<script setup>
  import {computed, ref, watchEffect} from 'vue';
  import ThreeTabComponent from "./ThreeTabComponent.vue";
  import GuessList from "./GuessList.vue";
  import QuestionList from "./QuestionList.vue";
  import ResponseDataGrid from "./ResponseDataGrid.vue";
  import ModalLightbox from "./ModalLightbox.vue";
  import FactorDetails from "./FactorDetails.vue";

  const URL_PREFIX = "https://d3rqhmpqgpvmke.cloudfront.net/guessers/";
  const URL_SUFFIX = "Guesser.json"
  const guesserName = ref("");
  const guesserData = ref(null);
  const rawDataDisplayed = ref(false);
  const singleFactorDisplayed = ref(null);
  const content = computed(() => {
    if (guesserData.value === null) {
      return "";
    } else {
      return JSON.stringify(guesserData.value, null, 2);
    }
  });

  watchEffect(async () => {
    if (guesserName.value !== "") {
      const url = URL_PREFIX + guesserName.value + URL_SUFFIX;
      const jsonData = await (await fetch(url)).json(); // fetch and parse
      guesserData.value = jsonData;
    }
  });

  function showSingleFactor(factorSpec) {
    singleFactorDisplayed.value = factorSpec;
  }


  /*
   * Called after an edit and is passed the update object. This performs the same
   * work as the server does to update the structure so we can see the changes
   * immediately (even though they won't be available on the server for several
   * minutes).
   */
  function onUpdateGuesser(update) {
    console.log("onUpdateGuesser", update);
    // --- Verify top-level stuff ---
    if (update.guesserType !== guesserName.value || update.version !== 1) {
      return;
    }

    // --- Process the questions ---
    for (const questionUpdate of update.questions) {
      if (questionUpdate.qID < 0) {
        throw new Error("Admin app doesn't support adding questions.");
      } else {
        // --- Updated Question ---
        if (questionUpdate.qID >= guesserData.value.questions.length) {
          throw new Error(`Question ID ${questionUpdate.qID} does not exist.`);
        }
        const question = guesserData.value.questions[questionUpdate.qID];
        if ("question" in questionUpdate) {
          question.question = questionUpdate.question;
        }
        if ("verified" in questionUpdate) {
          question.verified = questionUpdate.verified;
        }
      }
    }

    // --- Process the guesses ---
    for (const guessUpdate of update.guesses) {
      if (guessUpdate.gID < 0) {
        throw new Error("Admin app doesn't support adding guesses.");
      } else {
        // --- Updated Guess ---
        if (guessUpdate.gID >= guesserData.value.guessArray.length) {
          throw new Error(`Guess ID ${guessUpdate.gID} does not exist.`);
        }
        const guess = guesserData.value.guessArray[guessUpdate.gID];
        if ("guess" in guessUpdate) {
          guess.name = guessUpdate.guess;
        }
        if ("verified" in guessUpdate) {
          guess.verified = guessUpdate.verified;
        }
      }
    }

    // --- Process the Answers ---
    for (const answerUpdate of update.answers) {
      // --- Find the qID ---
      const qID = answerUpdate.qID;
      if (qID < 0) {
        throw new Error("Admin app doesn't support answering new questions.");
      }
      if (qID >= guesserData.value.questions.length) {
        throw new Error(`Answer has qID ${qID} that does not exist`);
      }

      // --- Find the gID ---
      const gID = answerUpdate.gID;
      if (gID < 0) {
        throw new Error("Admin app doesn't support answering new guesses.");
      }
      if (gID >= guesserData.value.guessArray.length) {
        throw new Error(`Answer has gID ${gID} that does not exist`);
      }

      // --- Determine new counts of yeses, nos, and responses ---
      let newNumYeses;
      let newNumNos;
      let newNumResponses;
      const question = guesserData.value.questions[qID];
      if ("counts" in answerUpdate) {
        if (answerUpdate.counts.length !== 3) {
          throw new Error("Answer has counts that isn't 3 items long.");
        }
        newNumYeses = answerUpdate.counts[0];
        newNumNos = answerUpdate.counts[1];
        newNumResponses = newNumYeses + newNumNos + answerUpdate.counts[2];
      } else {
        if (!("increments" in answerUpdate)) {
          throw new Error("Answer must have counts or increments.");
        }
        if (answerUpdate.increments.length !== 3) {
          throw new Error("Answer has increments that isn't 3 items long.");
        }
        const oldNumYeses = question.numYeses[gID.toString()] ?? 0;
        const oldNumNos = question.numNos[gID.toString()] ?? 0;
        const oldNumResponses = question.numResponses[gID.toString()] ?? 0;
        const incrYeses = answerUpdate.increments[0];
        const incrNos = answerUpdate.increments[1];
        const incrResponses = incrYeses + incrNos + answerUpdate.increments[2];
        newNumYeses = oldNumYeses + incrYeses;
        newNumNos = oldNumNos + incrNos;
        newNumResponses = oldNumResponses + incrResponses;
      }

      // --- Set new counts ---
      question.numYeses[gID.toString()] = newNumYeses;
      question.numNos[gID.toString()] = newNumNos;
      question.numResponses[gID.toString()] = newNumResponses;
    }
  }

</script>

<template>
  <div class="whole-thing">
    <h1>Goblini Admin Screen</h1>
    <div class="controls-row">
      <select v-model="guesserName">
        <option disabled value="">Please select one</option>
        <option value="animal">Animals</option>
        <option value="dnd">D&D</option>
      </select>
      <button v-if="guesserData != null" @click="rawDataDisplayed = !rawDataDisplayed">Raw JSON</button>
    </div>
    <div v-if="rawDataDisplayed">
      <p>Full data file:</p>
      <textarea v-model="content" placeholder="Select guesser..."></textarea>
    </div>
    <div v-if="!rawDataDisplayed && guesserData != null" class="main-display">
      <three-tab-component>
        <template v-slot:title0>Guesses</template>
        <template v-slot:item0>
          <guess-list :guesser-data="guesserData" @show-factor="showSingleFactor"/>
        </template>

        <template v-slot:title1>Questions</template>
        <template v-slot:item1>
          <question-list :guesser-data="guesserData" @show-factor="showSingleFactor"/>
        </template>

        <template v-slot:title2>Response Data</template>
        <template v-slot:item2>
          <response-data-grid
              :guesser-data="guesserData"
              @show-factor="showSingleFactor"
          />
        </template>
      </three-tab-component>
    </div>
    <div>...</div>
    <modal-lightbox
        v-if="singleFactorDisplayed !== null"
        @exit="() => singleFactorDisplayed = null"
        :button-names="[]"
    >
      <factor-details
          :guesser-type="guesserName"
          :guesser-data="guesserData"
          :factor-spec="singleFactorDisplayed"
          @exit="() => singleFactorDisplayed = null"
          @update-guesser="onUpdateGuesser"
      />
    </modal-lightbox>
  </div>
</template>

<style scoped>
  .whole-thing {
    width: 100%;
    height: 100%;
  }
  .controls-row {
    display: flex;
    justify-content: space-between;
    margin-bottom: 5px;
  }
  textarea {
    width: 80vw;
    height: 70vh;
  }
  .main-display {
    max-height: 100%;
  }
</style>
