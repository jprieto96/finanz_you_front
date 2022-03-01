<template>
  <div class="main_container">
    <div class="d-flex flex-row justify-content-around">
        <div class="card">
          <div class="card-body">
            <h5 class="card-title">Valor total</h5>
            <p class="card-text">{{ totalValue }}</p>
          </div>
        </div>
        <div class="card">
          <div class="card-body">
            <h5 class="card-title">G&P</h5>
            <p class="card-text">{{ infoFinances }}</p>
          </div>
        </div>
        <div class="card">
          <div class="card-body">
            <h5 class="card-title">Total Activos</h5>
            <p class="card-text">{{ totalAssets }}</p>
          </div>
        </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "ClientSummary",
  created() {
    if (this.$cookies.get("Session") == null) {
      window.location.href = '/login'
    }
    else {
      this.getDBValues();
    }
  },
  data() {
    return {
      apiKey: process.env.VUE_APP_APIKEY,
      backURL: process.env.VUE_APP_BACK_URL,
      totalValue: 0,
      earnsAndLoses: 0,
      totalAssets: 0,
      info: {},
      infoFinances: {},
    }
  },
  methods: {
    getDBValues() {

      this.info = JSON.parse(localStorage.getItem("info"))
      this.infoFinances = JSON.parse(localStorage.getItem("infoFinances"))

      if( this.info ===  {}) {
       //   (Object.keys(this.infoFinances).length === 0 && Object.keys(this.info).length !== 0 ) ||
       //   (Object.keys(this.info).length === 0 && Object.keys(this.infoFinances).length !== 0 )){

        let hashClient = this.$cookies.get("Session")
        this.infoFinances = {}
        axios
            .get(this.backURL + 'client/showPortfolio/' + hashClient)
            .then(response => {
              this.info = response.data;
              localStorage.setItem("info", JSON.stringify(this.info));
              this.financialData();
              this.getTotalAssets();
              this.getTotalValue();
              this.getTotalEarnsAndLoses();
            })
            .catch(err => {
              this.showWarningModal(err.response.data);
            })

      }
      else {
        this.getTotalAssets();
        this.getTotalValue();
        this.getTotalEarnsAndLoses();
      }
    },
    getTotalAssets(){
      this.totalAssets = Object.keys(this.info).length;
      this.earnsAndLoses = this.info;
    },
    getTotalValue(){
      /*let totalValueLocal = 0;
      for (const index in this.info) {
        totalValueLocal += this.infoFinances[index].quoteResponse.result[0].regularMarketPrice
      }
      this.totalValue = totalValueLocal;*/
    },
    getTotalEarnsAndLoses(){

    },
    async financialData(){
      let promises = [];
      for (let index in this.info) {
        var options = {
          method: 'GET',
          url: 'https://stock-data-yahoo-finance-alternative.p.rapidapi.com/v6/finance/quote',
          params: {symbols: index},
          headers: {
            'x-rapidapi-host': 'stock-data-yahoo-finance-alternative.p.rapidapi.com',
            'x-rapidapi-key': this.apiKey
          }
        };

        promises.push(axios.request(options).then(response => {
              this.infoFinances[index] = response.data;
              return response.data;
            }).catch(err => {
              this.showWarningModal(err);
            })
        );
        await new Promise(r => setTimeout(r, 300))
      }

      localStorage.setItem("infoFinances", JSON.stringify(this.infoFinances))

      Promise.all(promises)
          .then(() => {
            this.getTotalValue();
          })
          .catch(err => {
            console.log(err)
          })
    },
  }
}
</script>

<style scoped>

.main_container{
  margin-top: 15px;
}
.card-title{
  font-weight: bold;
}

.card{
  min-width: 200px;
}

</style>