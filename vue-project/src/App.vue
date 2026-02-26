<script setup>
import { ref } from 'vue'
import AppHeader from './components/AppHeader.vue'
import XPCalculator from './components/xpcalculator.vue'
import ScrollCalculator from './components/ScrollCalculator.vue'
import SpellParchoCalculator from './components/SpellParchoCalculator.vue'

const currentView = ref('xp')

const changeView = (view) => {
  currentView.value = view
}
</script>

<template>
  <AppHeader :currentView="currentView" @change-view="changeView" />
  <main>
    <XPCalculator v-show="currentView === 'xp'" />
    <ScrollCalculator v-if="currentView === 'scrolls'" />
    <SpellParchoCalculator v-if="currentView === 'spells'" />
  </main>
</template>

<style>
body {
  margin: 0;
  padding: 0;
  background: #000000;
  background-attachment: fixed;
  min-height: 100vh;
  font-family: 'Outfit', sans-serif;
}
</style>
