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
  </div>
</template>

<script>
import { AccumulationChartPlugin, AccumulationTooltip, PieSeries, AccumulationLegend, AccumulationDataLabel } from "@syncfusion/ej2-vue-charts";
import Vue from "vue";
import axios from "axios";
import API_KEY from "../../../constants/constants";

Vue.use(AccumulationChartPlugin);

export default Vue.extend({
  name: "ClientHome",
  data() {
    return {
      showView: false,
      showPieChart: false,
      pieChartData: [],
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
    getPieChart() {
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
        var options = {
          method: 'GET',
          url: 'https://stock-data-yahoo-finance-alternative.p.rapidapi.com/v6/finance/quote',
          params: {symbols: index},
          headers: {
            'x-rapidapi-host': 'stock-data-yahoo-finance-alternative.p.rapidapi.com',
            'x-rapidapi-key': API_KEY
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
            this.getPieChart()
            this.showView = true
          })
          .catch(err => {
            console.log(err)
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
            .get('https://finanzyou-back.herokuapp.com/client/showPortfolio/' + hashClient)
            .then(response => {
              this.info = response.data;
              localStorage.setItem("info", JSON.stringify(this.info))
              this.financialData();
            })
            .catch(err => {
              this.showWarningModal(err.response.data);
            })
      }
      else {
        this.getPieChart()
        this.showView = true
      }
    }
  }
});
</script>

<style scoped>

.container {
  padding: 20px;
}

</style>