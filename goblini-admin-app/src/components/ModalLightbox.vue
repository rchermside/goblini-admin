<script setup>
  const props = defineProps({
    buttonNames: { type: Array, required: false, default: () => ["Done"] },
  });
</script>

<template>
  <div class="modal-shade" @click="$emit('exit', null)">
    <div class="modal-content" @click.stop="">
      <slot/>
      <div class="button-row">
        <button
            v-for="buttonName in buttonNames"
            :onClick="() => $emit('exit', buttonName)"
        >{{buttonName}}</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
  .modal-shade {
    position: fixed;
    z-index: 10;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .modal-content {
    background-color: rgb(247, 247, 247);
    padding: 6px;
    border: 3px solid rgb(0, 0, 0);
    overflow: scroll;
    max-height: 90%;
    max-width: 90%;
  }
  .button-row {
    margin-top: 6px;
    display: flex;
    flex-direction: row;
    justify-content: flex-end;
  }
</style>