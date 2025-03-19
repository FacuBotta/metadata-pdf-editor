<script setup lang="ts">
import { RouterLink } from 'vue-router'
import GitHubIcon from '../icons/GitHubIcon.vue'
</script>

<template>
  <nav
    class="w-full max-w-[300px] mx-auto h-14 bg-slate-500/50 mt-2 flex items-center justify-around rounded-3xl"
  >
    <RouterLink active-class="border-b-2 border-blue-500" to="/">Home</RouterLink>
    <RouterLink active-class="border-b-2 border-blue-500" to="/about">About</RouterLink>
    <a href="https://github.com/FacuBotta/metadata-pdf-editor" class="flex gap-1" target="_blank"
      ><GitHubIcon /> Repo</a
    >
  </nav>
</template>
