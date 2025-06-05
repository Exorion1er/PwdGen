<script setup lang="ts">
import PasswordGenerator from './components/PasswordGenerator.vue'
</script>

<template>
  <div class="app">
    <h1 class="title">PwdGen</h1>
    <main class="main-content">
      <PasswordGenerator />
    </main>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Raleway:wght@100&display=swap');

:root {
  --bg-primary: #1a1a1a;
  --bg-secondary: #2d2d2d;
  --text-primary: #ffffff;
  --text-secondary: #b3b3b3;
  --accent-primary: #4CAF50;
  --accent-secondary: #2196F3;
  --border-color: #404040;
}

body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
    Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  background: var(--bg-primary);
  color: var(--text-primary);
  min-height: 100vh;
}

.app {
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: stretch;
  justify-content: flex-start;
  padding: 0;
  box-sizing: border-box;
  width: 100%;
  overflow: hidden;
  position: relative;
}

.font-options {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  padding: 2rem;
  background-color: var(--bg-secondary);
  border-bottom: 1px solid var(--border-color);
}

.font-option {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
}

.font-name {
  color: var(--text-secondary);
  font-size: 0.9rem;
  font-family: monospace;
}

.title {
  font-size: 2.5rem;
  font-weight: 100;
  font-family: 'Raleway', sans-serif;
  color: var(--text-primary);
  text-align: center;
  padding: 1rem 0;
  margin: 0;
  background-color: var(--bg-primary);
  position: sticky;
  top: 0;
  z-index: 1;
  width: 100%;
  letter-spacing: -0.02em;
  line-height: 0.8;
}

.main-content {
  width: 100%;
  height: calc(100vh - 3.5rem);
  /* Subtract title height */
  box-sizing: border-box;
  flex: 1 1 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  overflow-y: auto;
  padding: 1rem;
}

@media (max-width: 600px) {
  .title {
    padding-top: 1rem;
    padding-bottom: 1rem;
  }
}
</style>
