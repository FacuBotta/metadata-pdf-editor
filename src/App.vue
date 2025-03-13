<script setup lang="ts">
import { RouterView } from 'vue-router'
import NavBar from './components/layout/NavBar.vue'
import FooterApp from './components/layout/FooterApp.vue'
</script>

<template>
  <header class="absolute top-0 left-0 w-full">
    <NavBar />
  </header>
  <main class="min-h-screen mx-auto pt-24 flex justify-center">
    <RouterView />
  </main>
  <FooterApp />
</template>

<style scoped></style>
