<template>
  <div class="container" v-if="showView">
    <div class="control-section" v-if="showPieChart">
      <div align='center'>
        <ejs-accumulationchart style='display:inline-block' :load='load' align='center' id='chartcontainer' :title="title"
                               :legendSettings='legendSettings' :tooltip='tooltip'>
          <e-accumulation-series-collection>
            <e-accumulation-series :dataSource='pieChartData' xName='x' yName='y' startAngle='60' :dataLabel='dataLabel' innerRadius='0%' name='Sectores' > </e-accumulation-series>

          </e-accumulation-series-collection>
        </ejs-accumulationchart>
      </div>
    </div>
    <div class="control-section" v-if="showStocksChart">
      <div align='center'>
        <ejs-accumulationchart style='display:inline-block' :load='load' align='center' id='chartcontainer2' :title="'% activos de tu cartera'"
                               :legendSettings='legendSettings' :tooltip='tooltipStocks'>
          <e-accumulation-series-collection>
            <e-accumulation-series :dataSource='pieChartData' xName='x' yName='y' startAngle='60' :dataLabel='dataLabel' innerRadius='0%' name='% cartera' > </e-accumulation-series>

          </e-accumulation-series-collection>
        </ejs-accumulationchart>
      </div>
    </div>
  </div>
</template>

<script>
import { AccumulationChartPlugin, AccumulationTooltip, PieSeries, AccumulationLegend, AccumulationDataLabel } from "@syncfusion/ej2-vue-charts";
import Vue from "vue";
import axios from "axios";

Vue.use(AccumulationChartPlugin);

export default Vue.extend({
  name: "ClientHome",
  data() {
    return {
      apiKey: process.env.VUE_APP_APIKEY,
      backURL: process.env.VUE_APP_BACK_URL,
      errorMSG: process.env.VUE_APP_ERROR_MSG,
      showView: false,
      showPieChart: false,
      showStocksChart:false,
      pieChartData: [],
      pieChartDataStocks: [],
      infoFinances: {},
      info: null,
      dataLabel: {
        visible: true, position: 'Outside',
        connectorStyle: { length: '20px', type: 'Curve' }, name: 'text',
      },

      legendSettings: {
        visible: false,
      },

      tooltip: { enable: true, format: '${point.x} : <b>${point.y}%</b>' },

      title: "Sectores de tu cartera"
    }
  },
  provide: {
    accumulationchart: [AccumulationLegend, PieSeries, AccumulationDataLabel, AccumulationTooltip]
  },
  created() {
    if(this.$cookies.get("Session") == null) {
      localStorage.clear()
      window.location.href = '/login'
    }
    else {
      this.getData()
    }
  },
  methods: {
    getAllStocksChart() {
      let stockInfo = JSON.parse(localStorage.getItem("info"))[this.id]
      let infoFinances = JSON.parse(localStorage.getItem("infoFinances"))
      let marketValueStockDetail = 0
      if(infoFinances[this.id].quoteResponse.result[0].currency === 'USD') {
        marketValueStockDetail = (infoFinances[this.id].quoteResponse.result[0].regularMarketPrice * stockInfo.quantity) * 0.87
      }
      else {
        marketValueStockDetail = infoFinances[this.id].quoteResponse.result[0].regularMarketPrice * stockInfo.quantity
      }

      let totalMarketValue = 0
      for(let stock in infoFinances) {
        if(infoFinances[stock].quoteResponse.result[0].currency === 'USD') {
          totalMarketValue += (infoFinances[stock].quoteResponse.result[0].regularMarketPrice * stockInfo.quantity) * 0.87
        }
        else {
          totalMarketValue += infoFinances[stock].quoteResponse.result[0].regularMarketPrice * stockInfo.quantity
        }
      }
      
      let percentageOfParticularStock = ((marketValueStockDetail / totalMarketValue) * 100).toFixed(2)
      let restPercentage = (100 - percentageOfParticularStock).toFixed(2)
      this.pieChartData.push({'x': this.id, 'y': percentageOfParticularStock, text: this.id})
      this.pieChartData.push({'x': "Resto de la cartera", 'y': restPercentage, text: "Resto de la cartera"})
      this.showPieChart = true
    },
    getSectorChart() {
      let sectors = new Map()
      for (let index in this.info) {
        if(sectors.has(this.info[index]['sector'])) {
          sectors.set(this.info[index]['sector'], sectors.get(this.info[index]['sector']) + 1)
        }
        else {
          sectors.set(this.info[index]['sector'], 1)
        }
      }

      for (let [key, value] of sectors) {
        this.pieChartData.push({'x': key, 'y': (value/Object.values(this.info).length * 100).toFixed(2), text: key + " " + value})
      }

      if(sectors.size !== 0) this.showPieChart = true

    },
    async financialData(){
      let promises = [];
      for (let index in this.info) {
        let options = {
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
            }).catch(() => {
              this.showWarningModal(this.errorMSG);
            })
        );
        await new Promise(r => setTimeout(r, 300))
      }

      localStorage.setItem("infoFinances", JSON.stringify(this.infoFinances))

      Promise.all(promises)
          .then(() => {
            this.getSectorChart()
            this.showView = true
          })
          .catch(() => {
            this.showWarningModal(this.errorMSG);
          })
    },
    load: function(args) {
      let selectedTheme = location.hash.split('/')[1];
      selectedTheme = selectedTheme ? selectedTheme : 'Material';
      args.chart.theme = (selectedTheme.charAt(0).toUpperCase() +
          selectedTheme.slice(1)).replace(/-dark/i, 'Dark').replace(/contrast/i, 'Contrast');
    },
    getData(){
      let hashClient = this.$cookies.get("Session")
      this.info = JSON.parse(localStorage.getItem("info"))
      this.infoFinances = JSON.parse(localStorage.getItem("infoFinances"))

      if(this.infoFinances === null || this.info === null ||
          (Object.keys(this.infoFinances).length === 0 && Object.keys(this.info).length !== 0 ) ||
          (Object.keys(this.info).length === 0 && Object.keys(this.infoFinances).length !== 0 )) {
        this.infoFinances = {}
        axios
            .get(this.backURL + 'client/showPortfolio/' + hashClient)
            .then(response => {
              this.info = response.data;
              localStorage.setItem("info", JSON.stringify(this.info))
              this.financialData();
            })
            .catch(() => {
              this.showWarningModal(this.backURL);
            })
      }
      else {
        this.getSectorChart()
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
});
</script>

<style scoped>

.container {
  padding: 20px;
}

</style>