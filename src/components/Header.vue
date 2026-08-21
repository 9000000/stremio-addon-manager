<template>
  <div class="hero">
    <div class="logo is-center is-vertical-align">
      <img v-if="addonLogo" :src="addonLogo" alt="Addon logo" />
      <h1 v-else>{{ addonName }}</h1>
      <h3>{{ addonSummary }}</h3>
      <small><em>{{ addonTagline }}</em></small>
    </div>
  </div>
</template>

<script setup>
defineProps({
  addonName: {
    type: String,
    required: true
  },
  addonLogo: {
    type: String,
    required: false
  },
  addonSummary: {
    type: String,
    required: false
  },
  addonTagline: {
    type: String,
    required: false
  }
})
</script>

<style scoped>
.hero {
  padding: 0.5rem 0 0.25rem;
}

.logo {
  flex: 1;
  flex-direction: column;
  margin-top: 0.75rem;
  margin-bottom: 0.25rem;
  text-align: center;
  align-items: center;
}

.logo img {
  width: 100%;
  max-width: 150px;
  height: auto;
  display: block;
  margin: 0 auto;
}

.logo svg {
  width: 100%;
  max-width: 150px;
}

.logo h1 {
  font-size: 1.4rem;
  margin: 0.25rem 0;
  text-align: center;
}

.logo h3 {
  font-size: 1.02rem;
  font-weight: 400;
  text-align: center;
  margin: 0.45rem 0 0.2rem;
  line-height: 1.4;
  color: var(--font-color);
  opacity: 0.9;
}

.logo small {
  font-size: 0.85rem;
  opacity: 0.75;
}
</style>
