<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import HelloWorld from './components/HelloWorld.vue'
import { ref } from 'vue'
const version = import.meta.env.VITE_APP_VERSION
const isPlaying = ref(false)

function handlePlaying(value: boolean) {
  isPlaying.value = value
}
</script>

<template>
  <header>
    <img
      v-if="!isPlaying"
      alt="Kobold loading"
      class="logo"
      src="@/assets/loading.gif"
      width="125"
      height="125"
    />
    <img
      v-if="isPlaying"
      alt="Kobold playing"
      class="logo"
      src="@/assets/pog.gif"
      width="125"
      height="125"
    />

    <div class="wrapper">
      <HelloWorld msg="How the Kobold Doot works!" @is-playing="handlePlaying" />

      <nav>
        <RouterLink to="/">Home</RouterLink>
        <RouterLink to="/about">About</RouterLink>
      </nav>
    </div>
  </header>

  <RouterView />
  <div>
    <span>Version: {{ version }}</span>
  </div>
</template>

<style scoped>
header {
  line-height: 1.5;
  max-height: 100vh;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 2rem;
}

nav a.router-link-exact-active {
  color: var(--color-text);
}

nav a.router-link-exact-active:hover {
  background-color: transparent;
}

nav a {
  display: inline-block;
  padding: 0 1rem;
  border-left: 1px solid var(--color-border);
}

nav a:first-of-type {
  border: 0;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }

  nav {
    text-align: left;
    margin-left: -1rem;
    font-size: 1rem;

    padding: 1rem 0;
    margin-top: 1rem;
  }
}
</style>
