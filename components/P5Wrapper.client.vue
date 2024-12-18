<template>
  <P5 :sketch="sketch" />
</template>

<script lang="ts" setup>
const props = defineProps(["sketch"]);
const sketch = props.sketch;
</script>

<style></style>
