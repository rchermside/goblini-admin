<script setup>
  const props = defineProps({
    guesserData: Object,
  });
</script>

<template>
  <div class="guess-list">
    <div
        v-for="guess in guesserData.guessArray"
        :key="guess.guessId"
    >
      <button @click="$emit('showGuess', guess)">view</button>
      <span>{{guess.name}}</span>
    </div>
  </div>
</template>

<style scoped>
  .guess-list {
    display: flex;
    flex-direction: column;
    overflow-y: scroll;
    height: 200px;
    border: 1px solid black;
    margin: 5px;
  }
  button {
    margin-right: 5px;
  }
</style>
