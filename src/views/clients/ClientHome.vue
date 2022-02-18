<template>
  <div class="main_container">
  </div>
</template>

<script>
export default {
  name: "ClientHome",
  created() {
    if(this.$cookies.get("Session") == null) {
      window.location.href = '/login'
    }
  }
}
</script>

<style scoped>

</style>