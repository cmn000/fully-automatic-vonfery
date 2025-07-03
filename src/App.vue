<template>

  <v-app>

  <v-navigation-drawer app v-model="drawer">
    <v-list>
      <v-list-item 
      prepend-icon="mdi-home"
      to="/">首页</v-list-item>
      <v-list-item 
      prepend-icon="mdi-database-check"
      to="/data">数据</v-list-item>
      <v-list-item>
      </v-list-item>
    </v-list>
  </v-navigation-drawer>
    <v-main>

      <router-view />
      
    </v-main>
  </v-app>
</template>
<script setup>
import { ref } from 'vue';

const drawer = ref(true);
</script>
