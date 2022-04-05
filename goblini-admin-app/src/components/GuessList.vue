<script setup>
  const props = defineProps({
    guesserData: Object,
  });
  const emit = defineEmits(["show-factor"]);

  function onButtonClick(guess) {
    emit("show-factor", {
      factorType: "guess",
      factorId: guess.guessId,
      factor: guess,
      initialMode: "view",
    });
  }
</script>

<template>
  <div class="guess-list">
    <div
        v-for="guess in guesserData.guessArray"
        :key="guess.guessId"
    >
      <button @click="onButtonClick(guess)">view</button>
      <span>{{guess.name}}</span>
    </div>
  </div>
</template>

<style scoped>
  .guess-list {
    display: flex;
    flex-direction: column;
    overflow-y: scroll;
    min-height: 200px;
    border: 1px solid black;
    margin: 5px;
  }
  button {
    margin-right: 5px;
  }
</style>
