<template>
    <v-toolbar app color="indigo darken-4" dark>
      <v-btn to="/" icon>
        <v-icon large>home</v-icon>
      </v-btn>
      <v-toolbar-title class="headline" @click="goHome">
            <span class="text-uppercase">CLAV</span>
            <span class="font-weight-light"> : calculador de dependências entre processos</span>
      </v-toolbar-title>
    </v-toolbar>
</template>

<script>
export default {
    methods: {
        goHome: function(){
            this.$router.push('/');
        }
    }
}
</script>

