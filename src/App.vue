<template>
  <v-app>
    <MainPageHeader/>

    <v-content>
      <router-view/>
    </v-content>

    <PageFooter/>
  </v-app>
</template>

<script>
import PageFooter from "@/components/PageFooter.vue"; // @ is an alias to /src
import MainPageHeader from "@/components/MainPageHeader.vue"; // @ is an alias to /src

export default {
  name: 'App',
  components: {
    PageFooter, MainPageHeader
  },
  data: () => ({
      
    })
}
</script>
