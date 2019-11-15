<template>
  <v-app>
    <v-content>
      <NutriScore/>
    </v-content>
  </v-app>
</template>

<script>
import NutriScore from './components/NutriScore';

export default {
  name: 'App',

  components: {
    NutriScore
  },

  data: () => ({
    //
  }),
};
</script>
