<script setup lang="ts">
defineProps<{ src: string }>()
const base = import.meta.env.BASE_URL
</script>

<template>
  <img :src="base + src" />
</template>
