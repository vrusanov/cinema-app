<template>
  <v-app dark>
    <v-main>
      <v-container>
        <Nuxt keep-alive />
      </v-container>
    </v-main>
    <v-footer
      :absolute="!fixed"
      app
    >
      <span>&copy; cinemApp by Vasyl Rusanov {{ new Date().getFullYear() }}</span>
    </v-footer>
  </v-app>
</template>

<script>
export default {
  data () {
    return {
      fixed: false,
    }
  }
}
</script>
