<script setup>
  import {computed, ref, watchEffect} from 'vue'
  import ThreeTabComponent from "./ThreeTabComponent.vue";
  import GuessList from "./GuessList.vue";
  import QuestionList from "./QuestionList.vue";
  import ResponseDataGrid from "./ResponseDataGrid.vue";
  import ModalLightbox from "./ModalLightbox.vue";
  import GuessDetails from "./GuessDetails.vue";
  import QuestionDetails from "./QuestionDetails.vue";

  const URL_PREFIX = "https://d3rqhmpqgpvmke.cloudfront.net/guessers/";
  const URL_SUFFIX = "Guesser.json"
  const guesserName = ref("");
  const guesserData = ref(null);
  const singleGuessDisplayed = ref(null);
  const singleQuestionDisplayed = ref(null);
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

  function showSingleGuess(guess) {
    singleGuessDisplayed.value = guess;
  }

  function showSingleQuestion(question) {
    singleQuestionDisplayed.value = question;
  }

</script>

<template>
  <div>
    <h1>Goblini Admin Screen</h1>
    <select v-model="guesserName">
      <option disabled value="">Please select one</option>
      <option value="animal">Animals</option>
      <option value="dnd">D&D</option>
    </select>
    <p>Full data file:</p>
    <textarea v-model="content" placeholder="Select guesser..."></textarea>
    <div v-if="guesserData != null">
      <three-tab-component>
        <template v-slot:title0>Guesses</template>
        <template v-slot:item0>
          <guess-list :guesser-data="guesserData" @show-guess="showSingleGuess"/>
        </template>

        <template v-slot:title1>Questions</template>
        <template v-slot:item1>
          <question-list :guesser-data="guesserData" @show-question="showSingleQuestion"/>
        </template>

        <template v-slot:title2>Response Data</template>
        <template v-slot:item2>
          <response-data-grid
              :guesser-data="guesserData"
              @show-guess="showSingleGuess"
              @show-question="showSingleQuestion"
          />
        </template>
      </three-tab-component>
    </div>
    <div>...</div>
    <modal-lightbox
        v-if="singleGuessDisplayed !== null"
        @exit="() => singleGuessDisplayed = null"
        :button-names="['Back']"
    >
      <guess-details :guesser-data="guesserData" :guess="singleGuessDisplayed"/>
    </modal-lightbox>
    <modal-lightbox
        v-if="singleQuestionDisplayed !== null"
        @exit="() => singleQuestionDisplayed = null"
        :button-names="['Back']"
    >
      <question-details :guesser-data="guesserData" :question="singleQuestionDisplayed"/>
    </modal-lightbox>
  </div>
</template>

<style scoped>
  textarea {
    width: 800px;
    height: 150px;
  }
</style>