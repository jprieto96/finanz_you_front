h4
<template>
  <div class="container detailPage" v-if="showView">
    <a :href="infoStock.website" target="_blank">
      <div class="titleStock">
        <b-img
          v-if="
            this.img !== null &&
            this.img.hasOwnProperty('branding') &&
            this.img.branding.logo_url !== null
          "
          class="logo"
          :src="img.branding.logo_url + '?apikey=' + apiKeyForImage"
          fluid
          alt="Responsive image"
        ></b-img>
        <h1 v-if="img !== null">{{ this.img.name }}</h1>
        <h1 v-else>{{ this.id }}</h1>
      </div>
    </a>
    <p
      class="stockDescription"
      v-if="this.infoStock.hasOwnProperty('longBusinessSummary')"
    >
      {{ this.infoStock.longBusinessSummary }}
    </p>
    <p class="stockDescription" v-else>Descripción no disponible</p>
    <br />
    <hr />
    <br />
    <div class="container">
      <div class="row">
        <div
          class="control-section col-lg-6 col-md-6 col-sm-12"
          v-if="showPieChart"
        >
          <div align="center">
            <ejs-accumulationchart
              style="display: block"
              :load="load"
              align="center"
              id="chartcontainer"
              :title="'% ' + this.id + ' sobre tu cartera'"
              :legendSettings="legendSettings"
              :tooltip="tooltip"
            >
              <e-accumulation-series-collection>
                <e-accumulation-series
                  :dataSource="pieChartData"
                  xName="x"
                  yName="y"
                  startAngle="60"
                  :dataLabel="dataLabel"
                  innerRadius="0%"
                  name="% cartera"
                >
                </e-accumulation-series>
              </e-accumulation-series-collection>
            </ejs-accumulationchart>
          </div>
        </div>
        <br />
        <div
          class="recommendation-section col-lg-6 col-md-6 col-sm-12"
          align="center"
        >
          <div id="chart" v-if="showRecommendationGraph">
            <h5><b>Tendencia de recomendaciones</b></h5>
            <apexchart
              type="bar"
              height="350"
              :options="chartOptions"
              :series="series"
            ></apexchart>
          </div>
          <div v-else><h5><b>No existen recomendaciones para este valor</b></h5></div>
        </div>
      </div>
      <br />
      <hr v-if="this.news.length > 0" />
      <br />
      <div class="row">
        <div v-if="this.news.length > 0">
          <h4><b>{{ this.news.length }} últimas noticias</b></h4>
          <br />
          <b-list-group>
            <b-list-group-item
              v-for="item in news"
              v-bind:key="item"
              :href="item.article_url"
              target="_blank"
              class="align-items-start"
            >
              <div class="itemNews">
                <img :src="item.image_url" :alt="id" class="imgNews" />
                <div class="newsText">
                  <p>
                    <b>{{ item.title }}</b>
                  </p>
                  <p>{{ item.description }}</p>
                  <a class="linkNews" :href="item.article_url" target="_blank"
                    ><p>Ver noticia completa...</p></a
                  >
                </div>
              </div>
            </b-list-group-item>
          </b-list-group>
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
import {
  AccumulationChartPlugin,
  AccumulationTooltip,
  PieSeries,
  AccumulationLegend,
  AccumulationDataLabel,
} from "@syncfusion/ej2-vue-charts";
import Vue from "vue";
import axios from "axios";
import VueApexCharts from "vue-apexcharts";

Vue.use(AccumulationChartPlugin);

export default {
  name: "Stock",
  components: {
    apexchart: VueApexCharts,
  },
  data() {
    return {
      apiKey: process.env.VUE_APP_APIKEY,
      apiKeyForImage: process.env.VUE_APP_APIKEY_FOR_STOCK_INFO,
      backURL: process.env.VUE_APP_BACK_URL,
      errorMSG: process.env.VUE_APP_ERROR_MSG,
      infoStock: {},
      infoRecommendationStock: {},
      id: this.$route.params.id,
      showView: false,
      showPieChart: false,
      showRecommendationGraph: false,
      pieChartData: [],
      news: [],
      img: null,
      modal: {
        title: "",
        message: "",
        variant: "",
      },
      dataLabel: {
        visible: true,
        position: "Outside",
        connectorStyle: { length: "20px", type: "Curve" },
        name: "text",
      },

      legendSettings: {
        visible: false,
      },

      tooltip: { enable: true, format: "${point.x} : <b>${point.y}%</b>" },

      series: [
        {
          name: "Venta-Fuerte",
          data: [],
        },
        {
          name: "Venta",
          data: [],
        },
        {
          name: "Mantener",
          data: [],
        },
        {
          name: "Compra",
          data: [],
        },
        {
          name: "Compra-Fuerte",
          data: [],
        },
      ],

      chartOptions: {
        chart: {
          type: "bar",
          height: 350,
          stacked: true,
          toolbar: {
            show: true,
          },
          zoom: {
            enabled: false,
          },
        },
        responsive: [
          {
            breakpoint: 480,
            options: {
              legend: {
                position: "bottom",
                offsetX: -10,
                offsetY: 0,
              },
            },
          },
        ],
        plotOptions: {
          bar: {
            horizontal: false,
            borderRadius: 10,
          },
        },
        xaxis: {
          type: "string",
          categories: [],
        },
        legend: {
          position: "right",
          offsetY: 100,
        },
        fill: {
          opacity: 1,
        },
      },
    };
  },
  provide: {
    accumulationchart: [
      AccumulationLegend,
      PieSeries,
      AccumulationDataLabel,
      AccumulationTooltip,
    ],
  },
  created() {
    this.getData();
    this.getPieChart();
  },
  methods: {
    getPieChart() {
      let info = JSON.parse(localStorage.getItem("info"));
      let infoFinances = JSON.parse(localStorage.getItem("infoFinances"));
      let marketValueStockDetail = 0;
      if (infoFinances[this.id].quoteResponse.result[0].currency === "USD") {
        marketValueStockDetail =
          infoFinances[this.id].quoteResponse.result[0].regularMarketPrice *
          info[this.id].quantity *
          0.87;
      } else {
        marketValueStockDetail =
          infoFinances[this.id].quoteResponse.result[0].regularMarketPrice *
          info[this.id].quantity;
      }

      let totalMarketValue = 0;
      for (let stock in infoFinances) {
        if (infoFinances[stock].quoteResponse.result[0].currency === "USD") {
          totalMarketValue +=
            infoFinances[stock].quoteResponse.result[0].regularMarketPrice *
            info[stock].quantity *
            0.87;
        } else {
          totalMarketValue +=
            infoFinances[stock].quoteResponse.result[0].regularMarketPrice *
            info[stock].quantity;
        }
      }

      let percentageOfParticularStock = (
        (marketValueStockDetail / totalMarketValue) *
        100
      ).toFixed(2);
      let restPercentage = (100 - percentageOfParticularStock).toFixed(2);
      if (percentageOfParticularStock > 0) {
        this.pieChartData.push({
          x: this.id,
          y: percentageOfParticularStock,
          text: this.id,
        });
      }
      if (restPercentage > 0) {
        this.pieChartData.push({
          x: "Resto de la cartera",
          y: restPercentage,
          text: "Resto de la cartera",
        });
      }
      this.showPieChart = true;
    },
    async getData() {
      try {
        let options = {
          method: "GET",
          url:
            "https://stock-data-yahoo-finance-alternative.p.rapidapi.com/v11/finance/quoteSummary/" +
            this.id,
          params: {
            modules:
              "assetProfile,financialData,recommendationTrend,incomeStatementHistory",
          },
          headers: {
            "x-rapidapi-host":
              "stock-data-yahoo-finance-alternative.p.rapidapi.com",
            "x-rapidapi-key": this.apiKey,
          },
        };
        let responseStockInfo = await axios.request(options);
        this.infoStock =
          responseStockInfo.data.quoteSummary.result[0].assetProfile;
        this.infoRecommendationStock =
          responseStockInfo.data.quoteSummary.result[0];
      } catch (err) {
        console.log(err);
        document.querySelector(".errorDiv").style.display = "block";
        return;
      }

      try {
        let responseForPic = await axios.get(
          "https://api.polygon.io/v3/reference/tickers/" +
            this.id +
            "?apikey=" +
            this.apiKeyForImage
        );
        let responseNews = await axios.get(
          "https://api.polygon.io/v2/reference/news?ticker=" +
            this.id +
            "&limit=5&apiKey=" +
            this.apiKeyForImage
        );

        if (responseForPic.data.status !== "NOT_FOUND") {
          this.img = responseForPic.data.results;
        }
        if (responseNews.data.status !== "NOT_FOUND") {
          this.news = responseNews.data.results;
        }
        this.getRecommendationData();
        this.showView = true;
      } catch (err) {
        this.getRecommendationData();
        this.showView = true;
        console.log(err);
      }
    },
    showWarningModal(message) {
      this.$bvModal.show("modal");
      this.modal.message = message;
      this.modal.title = "¡Operación Fallida!";
      this.modal.variant = "warning";
    },
    load: function (args) {
      let selectedTheme = location.hash.split("/")[1];
      selectedTheme = selectedTheme ? selectedTheme : "Material";
      args.chart.theme = (
        selectedTheme.charAt(0).toUpperCase() + selectedTheme.slice(1)
      )
        .replace(/-dark/i, "Dark")
        .replace(/contrast/i, "Contrast");
    },
    getLast4Months() {
      var monthNames = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ];

      var today = new Date();
      var last4Months = [];

      for (let i = 0; i < 4; i++) {
        if (today.getMonth() - i < 0) {
          last4Months.unshift(monthNames[today.getMonth() - i + 12]);
        } else {
          last4Months.unshift(monthNames[today.getMonth() - i]);
        }
      }
      return last4Months;
    },

    async getRecommendationData() {
      this.chartOptions.xaxis.categories = this.getLast4Months();

      try {
        //series[0].data son los valores de strongSell
        for (let i = 0; i < 4; i++) {
          this.series[0].data.unshift(
            this.infoRecommendationStock.recommendationTrend.trend[i].strongSell
          );
        }

        //series[1].data son los valores de sell
        for (let i = 0; i < 4; i++) {
          this.series[1].data.unshift(
            this.infoRecommendationStock.recommendationTrend.trend[i].sell
          );
        }

        //series[2].data son los valores de hold
        for (let i = 0; i < 4; i++) {
          this.series[2].data.unshift(
            this.infoRecommendationStock.recommendationTrend.trend[i].hold
          );
        }

        //series[3].data son los valores de buy
        for (let i = 0; i < 4; i++) {
          this.series[3].data.unshift(
            this.infoRecommendationStock.recommendationTrend.trend[i].buy
          );
        }

        //series[4].data son los valores de strongBuy
        for (let i = 0; i < 4; i++) {
          this.series[4].data.unshift(
            this.infoRecommendationStock.recommendationTrend.trend[i].strongBuy
          );
        }
        this.showRecommendationGraph = true;
      } catch (err) {
        console.log(err);
      }
    },
  },
};
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
  text-decoration: none;
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
.newsCard {
  display: flex;
  height: 150px;
  margin: 25px;
}

.imgNews {
  height: 150px;
  width: 200px;
  margin: 20px;
}

.newsText {
  font-size: 15px;
  margin-top: 20px;
}

.linkNews {
  color: blue;
}

.itemNews {
  display: flex;
}

</style>
