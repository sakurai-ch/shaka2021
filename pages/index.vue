<template>
  <v-row 
    justify="center" 
    align="center"
  >
    <v-col 
      cols="12" 
      sm="8" 
      md="6"
    >
      <v-card class="logo py-4 d-flex justify-center link">
        <NuxtLink 
          to="/register" 
          tag="div"
          style="cursor:pointer;"
        >
          選手登録
        </NuxtLink>
      </v-card>
      <v-card class="logo py-4 d-flex justify-center link">
        <NuxtLink 
          to="/flight" 
          tag="div"
          style="cursor:pointer;"
        >
          フライト記録
        </NuxtLink>
      </v-card>
      <v-card class="logo py-4 d-flex justify-center link">
        <NuxtLink 
          to="/results" 
          tag="div"
          style="cursor:pointer;"
        >
          結果表
        </NuxtLink>
      </v-card>
      <v-card class="logo py-4 d-flex justify-center link">
        <NuxtLink 
          to="/results?edit=true" 
          tag="div"
          style="cursor:pointer;"
        >
          結果編集
        </NuxtLink>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
export default {
  data(){
    return {
      header: {
        title: "Home"
      }
    }
  },
  mounted() {
    this.updateHeader();
  },
  methods: {
    updateHeader() {
      this.$nuxt.$emit('updateHeader', this.header.title );
    }
  }
}
</script>