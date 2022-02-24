<template>
  <div class="container" v-if="showView">
    <h1>{{ id }}</h1>
    <p v-if="this.infoStock.finance.result.reports.length > 0 && this.infoStock.finance.result.reports[0].hasOwnProperty('summary')">
      {{ this.infoStock.finance.result.reports[0].summary }}
    </p>
    <p v-else>Summary N/A</p>
    <br>
    <br>
    <div class="control-section">
      <div>
        <ejs-stockchart
            style="display:block"
            id="stockchartcontainerdefault"
            :primaryXAxis="primaryXAxis"
            :primaryYAxis="primaryYAxis"
            :chartArea="chartArea"
            :tooltip="tooltip"
            :crosshair="crosshair"
            :tooltipRender="tooltipRender"
            :title="title"
            :border="border"
            :enablePeroiSelector="enablePeroiSelector"
            :theme="theme"
        >
          <e-stockchart-series-collection>
            <e-stockchart-series :dataSource="seriesData" type="Candle"  volume='volume' xName='x' low='low' high='high' open='open' close='close'></e-stockchart-series>
          </e-stockchart-series-collection>
        </ejs-stockchart>
      </div>

    </div>
  </div>
</template>

<script>
import CONSTANT from "../../../constants/constants";
import axios from "axios";
import Vue from "vue";
import {
  StockChartPlugin,
  DateTime,
  CandleSeries,
  Tooltip,
  Crosshair,
  RangeTooltip,
  LineSeries,
  SplineSeries,
  HiloOpenCloseSeries,
  HiloSeries,
  RangeAreaSeries,
  Trendlines,
  EmaIndicator,
  RsiIndicator,
  BollingerBands,
  TmaIndicator,
  MomentumIndicator,
  SmaIndicator,
  AtrIndicator,
  AccumulationDistributionIndicator,
  MacdIndicator,
  StochasticIndicator,
  Export
} from "@syncfusion/ej2-vue-charts";
import {chartData} from "@/views/stocks/datasource";

Vue.use(StockChartPlugin);

let selectedTheme = location.hash.split("/")[1];
selectedTheme = selectedTheme ? selectedTheme : "Material";
let theme = (selectedTheme.charAt(0).toUpperCase() + selectedTheme.slice(1)).replace(/-dark/i, "Dark");

export default {
  name: "Stock",
  data() {
    return {
      infoStock: {},
      id: this.$route.params.id,
      showView: false,
      seriesData: chartData,
      theme: theme,
      //Initializing Primary X Axis
      primaryXAxis: {
        valueType: "DateTime",
        majorGridLines: {color: "transparent"},
        crosshairTooltip: {enable: true}
      },
      //Initializing Primary Y Axis
      primaryYAxis: {
        lineStyle: { color: "transparent" },
        majorTickLines: { color: "transparent", width: 0 }
      },
      crosshair: {
        enable: true,
      },
      title: this.$route.params.id + ' Stock Price',
      chartArea: {
        border: {
          width: 0
        }
      },
      border : {width : 0},
      enablePeroiSelector: true,
      tooltip: { enable: true }
    }
  },
  provide: {
    stockChart: [
      DateTime,
      Tooltip,
      Crosshair,
      RangeTooltip,
      LineSeries,
      SplineSeries,
      CandleSeries,
      HiloOpenCloseSeries,
      HiloSeries,
      RangeAreaSeries,
      Trendlines,
      EmaIndicator,
      RsiIndicator,
      BollingerBands,
      TmaIndicator,
      MomentumIndicator,
      SmaIndicator,
      AtrIndicator,
      AccumulationDistributionIndicator,
      MacdIndicator,
      StochasticIndicator,
      Export
    ]
  },
  created() {
    this.getData()
    this.getChart()
  },
  methods: {
    async getData() {
      let options = {
        method: 'GET',
        url: 'https://stock-data-yahoo-finance-alternative.p.rapidapi.com/ws/insights/v1/finance/insights',
        params: {symbol: this.id},
        headers: {
          'x-rapidapi-host': 'stock-data-yahoo-finance-alternative.p.rapidapi.com',
          'x-rapidapi-key': CONSTANT.API_KEY
        }
      };

      let response = await axios.request(options)
      if(response === null) this.showWarningModal(CONSTANT.ERROR_MSG)
      else {
        this.infoStock = response.data
        this.showView = true
      }
    },
    tooltipRender: function(args){
      if (args.text.split('<br/>')[4]) {
        let target = parseInt(args.text.split('<br/>')[4].split('<b>')[1].split('</b>')[0]);
        let value  = (target / 100000000).toFixed(1) + 'B';
        args.text = args.text.replace(args.text.split('<br/>')[4].split('<b>')[1].split('</b>')[0], value);
      }
    },
    async getChart() { //This call returns the quotes of the value for the last 3 days
      let options = {
        method: 'GET',
        url: 'https://stock-data-yahoo-finance-alternative.p.rapidapi.com/v8/finance/chart/' + this.id,
        params: {range: '2d'},
        headers: {
          'x-rapidapi-host': 'stock-data-yahoo-finance-alternative.p.rapidapi.com',
          'x-rapidapi-key': CONSTANT.API_KEY
        }
      };

      let response = await axios.request(options)
      if (response === null) this.showWarningModal(CONSTANT.ERROR_MSG)
      else {
        console.log(response.data)
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