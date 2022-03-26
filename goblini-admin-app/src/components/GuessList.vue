<script setup>
  const props = defineProps({
    guesserData: Object,
  });
</script>

<template>
  <div>
    <div class="guess-list">
      <div
          v-for="guess in guesserData.guessArray"
          :key="guess.guessId"
      >{{guess.name}}</div>
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
</style>
