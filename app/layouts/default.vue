<template>
  <v-app>
    <v-navigation-drawer
      v-model="drawer"
      theme="light"
      temporary
    >
      <v-list>
        <v-list-item
          prepend-avatar="https://i.pinimg.com/originals/83/b0/7f/83b07fe79c4bdcc8e6c27f974c597842.jpg"
          title="Marnie Ocampo"
          subtitle="ocampomarnie05@gmail.com"
        />
      </v-list>

      <v-divider />

      <v-list density="compact" nav>
        <v-list-item
          prepend-icon="mdi-view-dashboard"
          title="Dashboard"
          to="/"
          exact
          @click="drawer = true"
        />

        <v-list-item
          prepend-icon="mdi-qrcode"
          title="QrScanner"
          to="/qr-scanner"
          @click="drawer = true"
        />

        <v-list-item
          prepend-icon="mdi-weather-cloudy"
          title="Weather App"
          to="/weather"
          @click="drawer = true"
        />
      </v-list>
    </v-navigation-drawer>

    <v-app-bar>
      <template #prepend>
        <v-app-bar-nav-icon
          @click="drawer = !drawer"
        />
      </template>

      <v-app-bar-title>
        Application Bar
      </v-app-bar-title>
    </v-app-bar>

    <v-main>
      <v-container fluid>
        <slot />
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from 'vue'

const drawer = ref(true)
</script>
