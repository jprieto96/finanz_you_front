<template>
  <div class="detailPage" v-if="showView">
    <a :href="infoStock.website" target="_blank">
      <div class="titleStock">
        <b-img v-if="img !== null && this.img.branding.logo_url !== null" class="logo" :src="img.branding.logo_url + '?apikey=' + apiKeyForImage" fluid alt="Responsive image"></b-img>
        <h1 v-if="img !== null">{{ this.img.name }}</h1>
        <h1 v-else>{{ this.id }}</h1>
      </div>
    </a>
    <p class="stockDescription" v-if="this.infoStock.hasOwnProperty('longBusinessSummary')">
      {{ this.infoStock.longBusinessSummary }}
    </p>
    <p class="stockDescription" v-else>Descripción no disponible</p>
    <br>
    <hr>
    <br>
    <div class="mid-section">
      <div v-if="this.news.length > 0" class="news">
        <h4><b>5 últimas noticias</b></h4>
        <br>
        <b-card v-for="item in news" v-bind:key="item" no-body class="overflow-auto newsCard text-center mb-3 card-item">
          <b-row no-gutters>
            <b-col md="3">
              <b-card-img :src="item.image_url" :alt="id" class="rounded-0"></b-card-img>
            </b-col>
            <b-col md="9">
              <b-card-body :title="item.title">
                <b-card-text>
                  {{
                    item.description
                  }}
                </b-card-text>
              </b-card-body>
            </b-col>
          </b-row>
        </b-card>
      </div>
      <div class="control-section" v-if="showPieChart">
        <div align='center'>
          <ejs-accumulationchart style='display:inline-block' :load='load' align='center' id='chartcontainer' :title="'%' + this.id + ' sobre tu cartera'"
                                :legendSettings='legendSettings' :tooltip='tooltip'>
            <e-accumulation-series-collection>
              <e-accumulation-series :dataSource='pieChartData' xName='x' yName='y' startAngle='60' :dataLabel='dataLabel' innerRadius='0%' name='Sectores' > </e-accumulation-series>

            </e-accumulation-series-collection>
          </ejs-accumulationchart>
        </div>
    </div>
    </div>
    <b-modal id="modal-1" title="BootstrapVue">
      <p class="my-4">Hello from modal!</p>
    </b-modal>

    <ModalMessage
        :message="modal.message"
        :title="modal.title"
        :variant="modal.variant"
        class="custom-modal"
    ></ModalMessage>
  </div>
  <div class="errorDiv" v-else>
    <h3>No hay información detallada sobre este activo</h3>
  </div>
</template>

<script>
import { AccumulationChartPlugin, AccumulationTooltip, PieSeries, AccumulationLegend, AccumulationDataLabel } from "@syncfusion/ej2-vue-charts";
import Vue from "vue";
import axios from "axios";

Vue.use(AccumulationChartPlugin);

export default {
  name: "Stock",
  data() {
    return {
      apiKey: process.env.VUE_APP_APIKEY,
      apiKeyForImage: process.env.VUE_APP_APIKEY_FOR_STOCK_INFO,
      backURL: process.env.VUE_APP_BACK_URL,
      errorMSG: process.env.VUE_APP_ERROR_MSG,
      infoStock: {},
      id: this.$route.params.id,
      showView: false,
      showPieChart: false,
      pieChartData: [],
      news: [],
      img: null,
      modal: {
        title: '',
        message: '',
        variant: '',
      },
      dataLabel: {
        visible: true, position: 'Outside',
        connectorStyle: { length: '20px', type: 'Curve' }, name: 'text',
      },

      legendSettings: {
        visible: false,
      },

      tooltip: { enable: true, format: '${point.x} : <b>${point.y}%</b>' }
    }
  },
  provide: {
    accumulationchart: [AccumulationLegend, PieSeries, AccumulationDataLabel, AccumulationTooltip]
  },
  created() {
    this.getData()
    this.getPieChart()
  },
  methods: {
    getPieChart() {
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
      console.log(percentageOfParticularStock)
      console.log(restPercentage)
      /*let sectors = new Map()
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

      if(sectors.size !== 0) this.showPieChart = true*/

    },
    async getData() {
      try {
        let options = {
            method: 'GET',
            url: 'https://stock-data-yahoo-finance-alternative.p.rapidapi.com/v11/finance/quoteSummary/' + this.id,
            params: {
              modules: 'assetProfile,financialData,recommendationTrend,incomeStatementHistory, '
            },
            headers: {
              'x-rapidapi-host': 'stock-data-yahoo-finance-alternative.p.rapidapi.com',
              'x-rapidapi-key': this.apiKey
            }
        };
        let responseStockInfo = await axios.request(options)
        this.infoStock = responseStockInfo.data.quoteSummary.result[0].assetProfile
      }
      catch(err) {
        console.log(err)
        document.querySelector(".errorDiv").style.display="block";
        return
      }

      try {
        let responseForPic = await axios.get("https://api.polygon.io/v3/reference/tickers/" + this.id + "?apikey=" + this.apiKeyForImage)
        let responseNews = await axios.get("https://api.polygon.io/v2/reference/news?ticker=" + this.id + "&limit=5&apiKey=" + this.apiKeyForImage)
        if(responseForPic.data.status !== "NOT_FOUND") {
          this.img = responseForPic.data.results
        }
        this.news = responseNews.data.results
        this.showView = true
      }
      catch(err) {
        this.showView = true
        console.log(err)
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

.titleStock {
  display: flex;
}

.logo {
  height: 50px;
  width: 50px;
  margin-right: 40px;
}

.detailPage {
  text-align: left;
  padding: 30px;
}

.stockDescription {
  padding-top: 25px;
}

a {
  color: black;
  text-decoration:none
}

.errorDiv {
  display: none;
  padding: 35px;
}

hr {
  border: none;
  height: 1px;
  color: #333;
  background-color: #333;
}

.mid-section {
  display: flex;
}

.news {
  width: 50%;
}

.newsCard {
  max-width: 540px;
  max-height: 160px;
}

</style>