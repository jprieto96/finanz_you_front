<template>
  <div class="detailPage" v-if="showView">
    <a :href="infoStock.website" target="_blank">
      <div class="titleStock">
        <b-img
          v-if="img !== null && this.img.branding.logo_url !== null"
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
          v-if="this.news.length > 0"
          class="news col-lg-6 col-md-6 col-sm-12"
        >
          <h4><b>3 últimas noticias</b></h4>
          <br />
          <b-card
            v-for="item in news"
            v-bind:key="item"
            no-body
            class="overflow-auto newsCard text-center mb-3 card-item"
          >
            <b-row no-gutters>
              <b-col md="3">
                <b-card-img
                  :src="item.image_url"
                  :alt="id"
                  class="rounded-0"
                ></b-card-img>
              </b-col>
              <b-col md="9">
                <b-card-body :title="item.title">
                  <b-card-text>
                    {{ item.description }}
                  </b-card-text>
                </b-card-body>
              </b-col>
            </b-row>
          </b-card>
        </div>
        <div class="col-lg-6 col-md-6 col-sm-12">
          <div class="control-section" v-if="showPieChart">
            <div align="center">
              <ejs-accumulationchart
                style="display: inline-block"
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
          <div class="recommendation-section" align="center">
            <div id="chart">
              <apexchart
                type="bar"
                height="350"
                :options="chartOptions"
                :series="series"
              ></apexchart>
            </div>
          </div>
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
      id: this.$route.params.id,
      showView: false,
      showPieChart: false,
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

      months: [],

      series: [
        {
          name: "Venta-Fuerte",
          data: [44, 55, 41, 67, 22, 43],
        },
        {
          name: "Venta",
          data: [13, 23, 20, 8, 13, 27],
        },
        {
          name: "Mantener",
          data: [11, 17, 15, 15, 21, 14],
        },
        {
          name: "Compra",
          data: [21, 7, 25, 13, 22, 8],
        },
        {
          name: "Compra-Fuerte",
          data: [5, 7, 11, 12, 17, 3],
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
          type: "datetime",
          categories: [
            "01/01/2011 GMT",
            "01/02/2011 GMT",
            "01/03/2011 GMT",
            "01/04/2011 GMT",
          ],
        },
        legend: {
          position: "right",
          offsetY: 40,
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
    this.getRecommendationData();
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
      console.log(this.showPieChart);
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
              "assetProfile,financialData,recommendationTrend,incomeStatementHistory, ",
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
            "&limit=3&apiKey=" +
            this.apiKeyForImage
        );
        if (responseForPic.data.status !== "NOT_FOUND") {
          this.img = responseForPic.data.results;
        }
        this.news = responseNews.data.results;
        this.showView = true;
      } catch (err) {
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
        if((today.getMonth() - i) <0){
            last4Months.push(
            monthNames[today.getMonth() - i + 12]
          );
        }
        else{
          last4Months.push(
            monthNames[today.getMonth() - i]
          );
        }
      }
      console.log(last4Months)
      return last4Months;
    },

    async getRecommendationData() {
      this.months = this.getLast4Months();
      console.log(this.months);
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