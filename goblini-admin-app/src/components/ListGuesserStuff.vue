<script setup>
  import {computed, ref, watchEffect} from 'vue'
  import GuessList from "./GuessList.vue";
  import QuestionList from "./QuestionList.vue";
  import ResponseDataGrid from "./ResponseDataGrid.vue";

  const URL_PREFIX = "https://d3rqhmpqgpvmke.cloudfront.net/guessers/";
  const URL_SUFFIX = "Guesser.json"
  const guesserName = ref("");
  const guesserData = ref(null);
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

</script>

<template>
  <div>
    <h1>Guesser Stuff</h1>
    <select v-model="guesserName">
      <option disabled value="">Please select one</option>
      <option value="animal">Animals</option>
      <option value="dnd">D&D</option>
    </select>
    <p>Full data file:</p>
    <textarea v-model="content" placeholder="Select guesser..."></textarea>
    <div v-if="guesserData != null">
      <guess-list :guesser-data="guesserData"/>
      <hr/>
      <question-list :guesser-data="guesserData"/>
      <hr/>
      <response-data-grid :guesser-data="guesserData"/>
    </div>
  </div>
</template>

<style scoped>
  textarea {
    width: 500px;
    height: 150px;
  }
</style>