<script setup lang="ts">
import { RouterLink } from 'vue-router'
</script>

<template>
  <nav
    class="w-full max-w-[300px] mx-auto h-14 bg-slate-500/50 mt-2 flex items-center justify-around rounded-3xl"
  >
    <RouterLink to="/">Home</RouterLink>
    <RouterLink to="/about">About</RouterLink>
  </nav>
</template>
