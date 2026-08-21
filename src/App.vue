<template>
  <DialogHost>
    <header>
      <DarkModeToggle />
      <Header addonName="Stremio Addon Manager"
        addonLogo="logo.png" />
    </header>
    <main>
      <Summary type="workflow" />
      <Configuration />
      <Summary type="spotlights" />
      <ThankYou />
      <FAQ />
    </main>
    <footer>
      <Footer />
    </footer>
  </DialogHost>
</template>

<script setup>
import './assets/main.css'
import Header from './components/Header.vue'
import DarkModeToggle from './components/DarkModeToggle.vue'
import Summary from './components/Summary.vue'
import Configuration from './components/Configuration.vue'
import ThankYou from './components/ThankYou.vue'
import FAQ from './components/FAQ.vue'
import Footer from './components/Footer.vue'
import DialogHost from './components/DialogHost.vue'
</script>
