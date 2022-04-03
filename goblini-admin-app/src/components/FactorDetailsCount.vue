<!--
  --  This is an internal component for FactorDetails. It renders the "count" of a certain answer
  --  for one question/guess pair. It renders differently (and behaves differently) depending on
  --  the mode.
  -->
<script setup>
  defineProps({
    mode: String, // either "view" or "edit" or "entry"
    answer: String, // either "yes", "no", or "maybe"
    value: Number, // a non-negative integer giving the count
    rowNum: Number, // a non-negative number giving the row
  });

  defineEmits(["input", "blur", "entry"]);
</script>

<template>
  <div class="wrapper">
    <div v-if="mode === 'view'" class="view-count">{{value}}</div>
    <input type="number" class="edit-count"
           v-if="mode === 'edit'"
           :value="value"
           @input="$emit('input', $event)"
           @blur="$emit('blur', $event)"
    />
    <div v-if="mode === 'entry'" class="entry-count">
      <label>
        <input type="radio" :name="`entry_${rowNum}`" @change="$emit('entry', $event)"/>
        {{answer}}
      </label>
    </div>
  </div>
</template>

<style scoped>
  .wrapper {
    display: contents;
  }
  .view-count {
    width: 4em;
    text-align: right;
    padding: 0 2px;
  }
  .edit-count {
    width: 4em;
    text-align: right;
  }
  .entry-count {
    width: 5em;
    text-align: center;
    padding: 0 2px;
  }
  .entry-count label {
    padding-left: 2px;
  }
</style>
