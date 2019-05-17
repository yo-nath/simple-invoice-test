<template>
  <v-app id="app">
    <div class="loading-overlay" v-if="loading">
      <v-progress-circular
        indeterminate
        color="white"
        :size="80"
        :width="8"
      ></v-progress-circular>
    </div>
    <v-navigation-drawer v-model="drawer" clipped fixed app>
      <navigation-side />
    </v-navigation-drawer>
    <v-toolbar app fixed clipped-left>
      <v-toolbar-side-icon @click.stop="drawer = !drawer"></v-toolbar-side-icon>
      <v-toolbar-title>Simple Invoice Apps</v-toolbar-title>
    </v-toolbar>
    <v-content>
      <router-view />
    </v-content>
  </v-app>
</template>

<script>
  import NavigationSide from './components/NavigationSide';
  import { mapState } from 'vuex';

  export default {
    components: {
      NavigationSide
    },
    data: () => ({
      drawer: false
    }),
    props: {
      source: String
    },
    computed: {
      ...mapState({
        loading: state => state.loading
      })
    }
  }
</script>

<style>
.loading-overlay {
  width: 100%;
  height: 100%;
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  background-color: rgba(85, 85, 85, 0.7);
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
}
</style>
