<template>
  <q-layout view="hHh Lpr lff">
    <q-header elevated>
      <q-toolbar>
        <q-toolbar-title>CityInfo</q-toolbar-title>
      </q-toolbar>
    </q-header>

    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import EssentialLink from "components/EssentialLink.vue";

export default {
  name: "MainLayout",
  components: { EssentialLink },
  data() {
    return {
      leftDrawerOpen: false,
      // essentialLinks: linksData,
    };
  },
};
</script>
