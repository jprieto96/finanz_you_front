<template>
  <div class="container" v-if="showView">
    <h1>{{ id }}</h1>
    <p v-if="this.infoStock.finance.result.reports.length > 0 && this.infoStock.finance.result.reports[0].hasOwnProperty('summary')">
      {{ this.infoStock.finance.result.reports[0].summary }}
    </p>
    <p v-else>Summary N/A</p>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "Stock",
  data() {
    return {
      apiKey: process.env.VUE_APP_APIKEY,
      backURL: process.env.VUE_APP_BACK_URL,
      errorMSG: process.env.VUE_APP_ERROR_MSG,
      infoStock: {},
      id: this.$route.params.id,
      showView: false
    }
  },
  created() {
    this.getData()
  },
  methods: {
    async getData() {
      let options = {
        method: 'GET',
        url: 'https://stock-data-yahoo-finance-alternative.p.rapidapi.com/ws/insights/v1/finance/insights',
        params: {symbol: this.id},
        headers: {
          'x-rapidapi-host': 'stock-data-yahoo-finance-alternative.p.rapidapi.com',
          'x-rapidapi-key': this.apiKey
        }
      };

      let response = await axios.request(options)
      if(response === null) this.showWarningModal(this.errorMSG)
      else {
        this.infoStock = response.data
        this.showView = true
      }
    },
    showWarningModal(message) {
      this.$bvModal.show("modal")
      this.modal.message = message
      this.modal.title = "¡Operación Fallida!"
      this.modal.variant = 'warning'
    }
  }
}
</script>

<style scoped>

.container {
  padding: 15px;
}

</style>