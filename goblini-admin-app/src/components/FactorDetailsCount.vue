<!--
  --  This is an internal component for FactorDetails. It renders the "count" of a certain answer
  --  for one question/guess pair. It renders differently (and behaves differently) depending on
  --  the mode.
  -->
<script setup>
  const props = defineProps({
    mode: String, // either "view" or "edit" or "entry"
    answer: String, // either "yes", "no", or "maybe"
    value: Number, // a positive integer giving the count
  });

  const emit = defineEmits(["input", "blur"]);
</script>

<template>
  <div v-if="mode === 'view'" class="view-count">{{value}}</div>
  <input type="number" class="edit-count"
      v-if="mode === 'edit'"
      :value="value"
      @input="$emit('input', $event)"
      @blur="$emit('blur', $event)"
  />
</template>

<style scoped>
  .view-count {
    width: 4em;
    text-align: right;
    padding: 0 2px;
  }
  .edit-count {
    width: 4em;
    text-align: right;
  }
</style>
