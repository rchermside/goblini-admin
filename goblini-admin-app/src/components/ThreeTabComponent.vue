<!--
  I couldn't figure out how to make this handle an arbitrary number of tabs, so it's hard-coded to 3.
  Also, it has an issue with the size being hard-coded.
-->
<script setup>
  import { ref, nextTick } from 'vue';
  const activeTab = ref(0);

  async function makeActive(tabNum) {
    activeTab.value = -1;
    await nextTick();
    activeTab.value = tabNum;
  }
</script>

<template>
  <div class="tab-component">
    <div class="tab-row">
      <div class="tab" :class="{active: activeTab === 0}" @click="makeActive(0)"><slot name="title0"/></div>
      <div class="tab" :class="{active: activeTab === 1}" @click="makeActive(1)"><slot name="title1"/></div>
      <div class="tab" :class="{active: activeTab === 2}" @click="makeActive(2)"><slot name="title2"/></div>
    </div>
    <div class="body">
      <div class="box" v-if="activeTab === 0">
        <slot name="item0"></slot>
      </div>
      <div class="box" v-if="activeTab === 1">
        <slot name="item1"></slot>
      </div>
      <div class="box" v-if="activeTab === 2">
        <slot name="item2"></slot>
      </div>
    </div>
  </div>
</template>

<style scoped>
  .tab-row {
    display: flex;
    flex-direction: row;
  }
  .tab {
    display: flex;
    align-items: center;
    border: 1px solid black;
    margin: 0 2px;
    padding: 0 8px;
    cursor: pointer;
    font-weight: bold;
    color: dimgrey;
    background: #f2f2f2;
    position: relative;
    top: 2px;
  }
  .tab:first-of-type {
    margin-left: 0;
  }
  .tab:last-of-type {
    margin-right: 0;
  }
  .tab:hover, .tab:focus {
  }
  .tab.active {
    border-top: 2px solid black;
    border-left: 2px solid black;
    border-right: 2px solid black;
    border-bottom: 2px solid white;
    color: inherit;
    background: inherit;
    z-index: 1;
  }
  .body {
    border: 2px solid black;
    overflow-x: scroll;
    overflow-y: scroll;
    max-width: 800px;
    max-height: 500px;
  }
</style>