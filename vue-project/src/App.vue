<script setup>
import AppHeader from './components/AppHeader.vue'
import XPCalculator from './components/xpcalculator.vue'
</script>

<template>
  <AppHeader />
  <main>
    <XPCalculator />
  </main>
</template>

<style>
body {
  margin: 0;
  padding: 0;
  background: radial-gradient(circle at top right, #1e293b, #020617);
  background-attachment: fixed;
  min-height: 100vh;
  font-family: 'Outfit', sans-serif;
}
</style>
