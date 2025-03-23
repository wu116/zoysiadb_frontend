<script setup lang="ts">
import { ElMessage } from 'element-plus'
import { ref } from 'vue'

defineProps<{ msg: string }>()
</script>

<template>
  <h1 color="$ep-color-primary">
    {{ msg }}
  </h1>

  <p>
    This website built basing on:
  </p>
</template>
