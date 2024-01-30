<template>
  <button @click="onClick">+</button>
  <div>{{ count }}</div>
</template>

<script setup>
import { ref } from 'vue'

let count = ref(0)
let onClick = () => count.value++
</script>
