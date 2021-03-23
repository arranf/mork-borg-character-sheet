<script lang="ts">
  export let onClick;
  export let text;
</script>

<template>
  <button class="link-button" on:click={onClick}>{text || "Roll"}</button>
</template>
