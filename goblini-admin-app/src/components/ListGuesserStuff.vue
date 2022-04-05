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
