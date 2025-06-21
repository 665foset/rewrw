<template>
  <div class="min-h-screen app-container">
    <AppHeader />
    <AppMain />
    <AppFooter :currentYear="currentYear" />
  </div>
</template>

<script setup>
import { ref } from 'vue';
import AppHeader from './components/AppHeader.vue';
import AppMain from './components/AppMain.vue';
import AppFooter from './components/AppFooter.vue';

const currentYear = new Date().getFullYear();
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Nunito:ital,wght@0,300;0,400;0,600;0,700;1,300;1,400&display=swap");

html, body, #app {
  height: 100%; 
  margin: 0; 
  padding: 0;
}

body {
  font-family: 'Nunito', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  font-size: 100%;
  background-color: #f9fafb; 
}

.app-container {
  display: flex;
  flex-direction: column;
  min-height: 100vh; 
  background-color: #f9fafb; 
  color: #1f2937;
}

.container {
  max-width: 1200px;
  margin-left: auto;
  margin-right: auto;
  padding-left: 1rem;
  padding-right: 1rem;
}
</style>
