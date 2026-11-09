<template>
  <v-app>
    <v-app-bar density="compact" color="primary" title="CAPS Parser" />
    <v-main>
      <CapsParser />
    </v-main>
  </v-app>
</template>

<script setup>
import CapsParser from './components/CapsParser.vue'
</script>
