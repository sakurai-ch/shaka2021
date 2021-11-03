<template>
  <v-app dark>
    <v-app-bar
      app
    >
      <v-toolbar-title> {{ title }} </v-toolbar-title>
      <v-spacer />
      <NuxtLink 
          to="/" 
          tag="div"
          style="cursor:pointer;"
          v-if='this.title != "フライト記録"'
        >
          ⌂
      </NuxtLink>
    </v-app-bar>
    <v-main>
      <v-container>
        <Nuxt />
      </v-container>
    </v-main>
    
    <v-footer
      app
    >
      <span>&copy; {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
data() {
    return {
      title: ""
    }
  },
  created() {
    this.setListener();
  },
  methods: {
    setListener() {
      this.$nuxt.$on('updateHeader', this.setHeader)
    },
    setHeader(title) {
      this.title = title
    },
  }
}
</script>
