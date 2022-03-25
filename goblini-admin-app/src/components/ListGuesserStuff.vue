<template>
  <div>
    <h1>Guesser Stuff</h1>
    <select v-model="guesserName">
      <option disabled value="">Please select one</option>
      <option value="animal">Animals</option>
      <option value="dnd">D&D</option>
    </select>
    <p>Some text goes here.</p>
    <textarea v-model="content" placeholder="Select guesser..."></textarea>
  </div>
</template>

<script>
  import {ref, watchEffect} from 'vue'

  const URL_PREFIX = "https://d3rqhmpqgpvmke.cloudfront.net/guessers/";
  const URL_SUFFIX = "Guesser.json"
  const content = ref("");
  const guesserName = ref("");

  watchEffect(async () => {
    if (guesserName.value !== "") {
      const url = URL_PREFIX + guesserName.value + URL_SUFFIX;
      const jsonData = await (await fetch(url)).json(); // fetch and parse
      const prettyString = JSON.stringify(jsonData, null, 2); // pretty print it
      content.value = prettyString; // display it
    }
  })


  export default {
    name: "ListGuesserStuff",
    setup() {
      return {
        content,
        guesserName,
      };
    }
  }
</script>

<style scoped>
  textarea {
    width: 500px;
    height: 150px;
  }
</style>