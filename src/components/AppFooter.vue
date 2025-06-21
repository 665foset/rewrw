<template>
  <footer class="app-footer">
    <div class="container footer-text">
    </div>
  </footer>
</template>

<script setup>
import { defineProps } from 'vue';


const props = defineProps({
  currentYear: {
    type: Number,
    required: true
  }
});
</script>

<style scoped>

.app-footer {
  background-color: #1f2937; 
  color: white;
  padding: 1.5rem;
  margin-top: 2rem;
  border-top-left-radius: 0.75rem;
  border-top-right-radius: 0.75rem;
  box-shadow: 0 -4px 6px rgba(0, 0, 0, 0.1);
}

.footer-text {
  text-align: center;
  font-size: 0.875rem;
}

.footer-note {
  margin-top: 0.5rem;
}
</style>
