<script setup>
  import {computed} from 'vue';

  const props = defineProps({
    guesserData: Object,
  });

  const content = computed(() => {
    if (props.guesserData.value === null) {
      return "";
    } else {
      return JSON.stringify(props.guesserData.value, null, 2);
    }
  });
</script>

<template>
  <div>
    <h2>Guesses</h2>
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
