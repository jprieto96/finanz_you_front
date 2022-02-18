<template>
  <div></div>
</template>

<script>
export default {
  name: "ClientLogOut",
  created() {
    if(this.$cookies.get("Session") == null) {
      window.location.href = '/login'
    }
    else {
      this.$cookies.remove("Session")
      window.location.href = '/'
    }
  }
}
</script>

<style scoped>

</style>