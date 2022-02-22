<template>
  <div></div>
</template>

<script>
export default {
  name: "ClientLogOut",
  created() {
    if(this.$cookies.get("Session") == null) {
      localStorage.removeItem("info")
      localStorage.removeItem("infoFinances")
      localStorage.removeItem("infoTransactions")
      window.location.href = '/login'
    }
    else {
      this.$cookies.remove("Session")
      localStorage.removeItem("info")
      localStorage.removeItem("infoFinances")
      localStorage.removeItem("infoTransactions")
      window.location.href = '/'
    }
  }
}
</script>

<style scoped>

</style>