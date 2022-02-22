<template>
  <div id="portfolio" v-if="showView">
    <div id="head">
      <b-row>
        <b-col lg="4" class="pb-2"><b-button id="buttonaddValues" v-b-modal.modal-lg variant="primary">Añadir valores</b-button>
          <b-modal id="modal-lg" size="lg" title="Añadir valores" hide-footer>
            <b-form @submit="onSubmit" @reset="onReset">
              <b-form-group
                  id="input-group-1"
                  label="ISIN/Nombre:"
                  label-for="input-1"
              >
                <b-form-input
                    v-model="form.nombre_ISIN"
                    id="inputsymbol"
                    :state="ISINState"
                    v-on:keyup="keyListener"
                    list="my-list-id"
                    required></b-form-input>
                <datalist id="my-list-id">
                  <option v-for="(value, index) in values" v-bind:key="index">{{ value.symbol + " - " + value.name}}</option>
                </datalist>
                <b-form-invalid-feedback>
                  Introduce un Nombre/ISIN válido
                </b-form-invalid-feedback>
              </b-form-group>

              <b-form-group
                  id="input-group-3"
                  label="Cantidad:"
                  label-for="input-3"
              >
                <b-form-input
                    v-model="form.quantity"
                    id="inputQuantity" type="number"
                    step="1"
                    min="1"
                    :state="quantityState"
                    required></b-form-input>
                <b-form-invalid-feedback>
                  Introduce una cantidad mayor que 0
                </b-form-invalid-feedback>
              </b-form-group>
              <b-form-group
                  id="input-group-4"
                  label="Precio de compra:"
                  label-for="input-4"
              >
                <b-form-input
                    v-model="form.buyPrice"
                    id="inputBuyPrice"
                    type="number"
                    step="0.01"
                    min="0.01"
                    :state="buyPriceState"
                    required></b-form-input>
                <b-form-invalid-feedback>
                  Introduce un precio de compra mayor que 0
                </b-form-invalid-feedback>
              </b-form-group>
              <b-form-group
                  id="input-group-6"
                  label="Fecha de la compra:"
                  label-for="input-6"
              >
                <b-form-datepicker
                    v-model="form.date"
                    id="datepicker"
                    class="mb-2"
                    :state="dateState"
                    required></b-form-datepicker>
                <b-form-invalid-feedback>
                  Introduce una fecha válida
                </b-form-invalid-feedback>
              </b-form-group>
              <br>
              <div>
                <b-button class="formButton" type="reset">Reset</b-button>
                <b-button class="formButton" variant="primary" type="submit">Enviar</b-button>
              </div>
            </b-form>
          </b-modal>
        </b-col>
        <b-col lg="4" class="pb-2"><h1>Portfolio</h1></b-col>
        <b-col lg="4" class="pb-2"></b-col>
      </b-row>
    </div>
    <br>
    <b-table-simple striped hover outlined responsive class="table">
      <b-thead>
      <b-tr>
        <b-th scope="col">Nombre</b-th>
        <b-th scope="col">Ticker/ISIN</b-th>
        <b-th scope="col">Cantidad</b-th>
        <b-th scope="col">GyP hoy</b-th>
        <b-th scope="col">% hoy</b-th>
        <b-th scope="col">Último precio</b-th>
        <b-th scope="col">Valor mercado</b-th>
        <b-th scope="col">Valor mercado (EUR)</b-th>
        <b-th scope="col">Precio medio de compra</b-th>
        <b-th scope="col">GyP total</b-th>
      </b-tr>
      </b-thead>
      <b-tbody>
      <b-tr v-for="(item, index) in this.info" v-bind:key="index" >
        <b-td v-if="infoFinances[index].quoteResponse.result[0].hasOwnProperty('longName')">{{ infoFinances[index].quoteResponse.result[0].longName}}</b-td>
        <!--Nombre-->
        <b-td v-else>{{ infoFinances[index].quoteResponse.result[0].shortName}}</b-td>
        <!--Ticker/ISIN-->
        <b-td>{{ index }}</b-td>
        <!--Cantidad-->
        <b-td>{{ item.quantity }}</b-td>
        <!--GyP-->
        <b-td class="gypgreen" v-if="(item.quantity *  infoFinances[index].quoteResponse.result[0].regularMarketChange * 0.87) > 0">{{ (item.quantity *  infoFinances[index].quoteResponse.result[0].regularMarketChange * 0.87).toFixed(2) + " €"}}</b-td>
        <b-td v-else-if="(item.quantity *  infoFinances[index].quoteResponse.result[0].regularMarketChange * 0.87) == 0">{{ (item.quantity *  infoFinances[index].quoteResponse.result[0].regularMarketChange * 0.87).toFixed(2) + " €"}}</b-td>
        <b-td class="gypred" v-else>{{ (item.quantity *  infoFinances[index].quoteResponse.result[0].regularMarketChange * 0.87).toFixed(2) + " €"}}</b-td>
        <!--% hoy-->
        <b-td class="gypgreen" v-if="(infoFinances[index].quoteResponse.result[0].regularMarketChangePercent > 0)">{{ (infoFinances[index].quoteResponse.result[0].regularMarketChangePercent).toFixed(2) }}%</b-td>
        <b-td v-else-if="(infoFinances[index].quoteResponse.result[0].regularMarketChangePercent == 0)">{{ (infoFinances[index].quoteResponse.result[0].regularMarketChangePercent).toFixed(2) }}%</b-td>
        <b-td class="gypred" v-else>{{ (infoFinances[index].quoteResponse.result[0].regularMarketChangePercent).toFixed(2) }}%</b-td>
        <!--Último precio-->
        <b-td v-if="infoFinances[index].quoteResponse.result[0].currency === 'USD'">{{ (infoFinances[index].quoteResponse.result[0].regularMarketPrice).toFixed(2) + " $"}}</b-td>
        <b-td v-else-if="infoFinances[index].quoteResponse.result[0].currency === 'EUR'">{{ (infoFinances[index].quoteResponse.result[0].regularMarketPrice).toFixed(2) + " €"}}</b-td>
        <!--Valor mercado-->
        <b-td v-if="infoFinances[index].quoteResponse.result[0].currency === 'USD'">{{ (infoFinances[index].quoteResponse.result[0].regularMarketPrice * item.quantity).toFixed(2) + " $"}}</b-td>
        <b-td v-else-if="infoFinances[index].quoteResponse.result[0].currency === 'EUR'">{{ (infoFinances[index].quoteResponse.result[0].regularMarketPrice * item.quantity).toFixed(2) + " €"}}</b-td>
        <!--Valor mercado(EUR)-->
        <b-td>{{ ((infoFinances[index].quoteResponse.result[0].regularMarketPrice * item.quantity) * 0.87).toFixed(2) + " €"}}</b-td>
        <!--Precio medio de compra-->
        <b-td v-if="infoFinances[index].quoteResponse.result[0].currency === 'USD'">{{ item.buyPrice.toFixed(2) + " $"}}</b-td>
        <b-td v-else-if="infoFinances[index].quoteResponse.result[0].currency === 'EUR'">{{ item.buyPrice.toFixed(2) + " €"}}</b-td>
        <!--GyP total-->
         <b-td class="gypgreen" v-if="gyp[index] > 0">{{gp[index] + " €"}}</b-td>
        <b-td v-else-if="gyp[index] == 0">{{ gp[index] + " €"}}</b-td>
        <b-td class="gypred" v-else>{{ gp[index] + " €"}}</b-td> 
      </b-tr>
      </b-tbody>
    </b-table-simple>
    <br>
    <br>
    <div v-if="showPieChart" class="control-section">
      <div align='center'>
        <ejs-accumulationchart style='display:block' :load='load' align='center' id='chartcontainer' :title="title"
                               :legendSettings='legendSettings' :tooltip='tooltip'>
          <e-accumulation-series-collection>
            <e-accumulation-series :dataSource='pieChartData' xName='x' yName='y' startAngle=60 :dataLabel='dataLabel' innerRadius='0%' name='Sectores' > </e-accumulation-series>

          </e-accumulation-series-collection>
        </ejs-accumulationchart>
      </div>
    </div>
  </div>
  <div v-else class="loading" >
    <loading :active="true"
             :can-cancel="false"
             :is-full-page="false"/>
  </div>
</template>

<script>
import axios from "axios";
import Vue from "vue";
import Loading from 'vue-loading-overlay';
import 'vue-loading-overlay/dist/vue-loading.css';
import { AccumulationChartPlugin, AccumulationTooltip, PieSeries, AccumulationLegend, AccumulationDataLabel } from "@syncfusion/ej2-vue-charts";
import API_KEY from "../../../constants/constants";


Vue.use(AccumulationChartPlugin);

export default Vue.extend({
  name: "ShowPortfolio",
  components: {
    Loading
  },
  data(){
      return{
        info : null,
        infoTransactions: null,
        gyp: {},
        showPieChart: false,
        form: {
          nombre_ISIN: '',
          quantity: 0,
          buyPrice: 0,
          date: null
        },
        infoFinances: {},
        showView : false,
        transactionDate: '',
        modal: {
          title: '',
          message: '',
          variant: '',
        },
        values: [],
        renderFinished: false,
        pieChartData: [],

        dataLabel: {
          visible: true, position: 'Outside',
          connectorStyle: { length: '20px', type: 'Curve' }, name: 'text',
        },

        legendSettings: {
          visible: false,
        },

        tooltip: { enable: true, format: '${point.x} : <b>${point.y}%</b>' },

        title: "Sectores de la cartera"
      }
  },
  provide: {
    accumulationchart: [AccumulationLegend, PieSeries, AccumulationDataLabel, AccumulationTooltip]
  },
  computed: {

    ISINState() {
      return this.form.nombre_ISIN.length > 0
    },
    quantityState() {
      return this.form.quantity > 0
    },
    buyPriceState() {
      return this.form.buyPrice > 0
    },
    dateState() {
      let currentTime = new Date()
      return this.form.date != null && this.form.date <= currentTime.toDateString()
    }
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
    getData(){
      let hashClient = this.$cookies.get("Session")
      this.info = JSON.parse(localStorage.getItem("info"))
      this.infoFinances = JSON.parse(localStorage.getItem("infoFinances"))
      this.infoTransactions = JSON.parse(localStorage.getItem("infoTransactions"))

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
              this.totalGyP(hashClient);
            })
            .catch(err => {
              this.showWarningModal(err.response.data);
            })
      }
      else {
        this.showView = true
        this.getPieChart()
      }
    },
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

      if(sectors.size !== 0) this.showPieChart = true;

    },
    load: function(args) {
      let selectedTheme = location.hash.split('/')[1];
      selectedTheme = selectedTheme ? selectedTheme : 'Material';
      args.chart.theme = (selectedTheme.charAt(0).toUpperCase() +
          selectedTheme.slice(1)).replace(/-dark/i, 'Dark').replace(/contrast/i, 'Contrast');
    },
    async onSubmit(event) {
      event.preventDefault()

      let hashClient = this.$cookies.get("Session")
      let deHashedClient = window.atob(hashClient).split(" ")
      let newForm = {}
      let infoStock = this.form.nombre_ISIN.split(" - ")

      localStorage.removeItem("info")
      localStorage.removeItem("infoFinances")
      localStorage.removeItem("infoTransactions")

      let op = {
        method: 'GET',
        url: 'https://stock-data-yahoo-finance-alternative.p.rapidapi.com/ws/insights/v1/finance/insights',
        params: {symbol: infoStock[0]},
        headers: {
          'x-rapidapi-host': 'stock-data-yahoo-finance-alternative.p.rapidapi.com',
          'x-rapidapi-key': API_KEY
        }
      };

      let axiosResponse = await axios.request(op)
      let sector = (axiosResponse.data.finance.result.companySnapshot !== undefined) ? axiosResponse.data.finance.result.companySnapshot.sectorInfo : "N/A"

      newForm.idClient = deHashedClient[0]
      newForm.stockID = infoStock[0]
      newForm.stockName = infoStock[1]
      newForm.stockSector = sector
      newForm.buyPrice = this.form.buyPrice
      newForm.quantity = this.form.quantity
      newForm.date = this.form.date

      console.log(newForm)

      axios
          .post('https://finanzyou-back.herokuapp.com/client/addTransaction', newForm)
          .then(response => {
            console.log(response)
            window.location.href = "/client/portfolio"
          })
          .catch(err => {
            this.showWarningModal(err.response.data)
          })

    },
    onReset(event) {
      event.preventDefault()
      this.resetForm();
    },

    resetForm() {
        this.form.nombre_ISIN= ''
        this.form.quantity= 0
        this.form.buyPrice= 0
        this.form.date= null
    },

    showWarningModal(message) {
      this.$bvModal.show("modal")
      this.modal.message = message
      this.modal.title = "¡Operación Fallida!"
      this.modal.variant = 'warning'
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

    keyListener:function(){ //Cuando escribes en el input para añadir valores llama a autocomplete
      let elem = document.getElementById("inputsymbol").value
      this.autoComplete(elem);
    },

    async autoComplete(toComplete){
      this.values=[];
      let options = {
        method: 'GET',
        url: 'https://stock-data-yahoo-finance-alternative.p.rapidapi.com/v6/finance/autocomplete',
        params: {query: toComplete, lang: 'en'},
        headers: {
          'x-rapidapi-host': 'stock-data-yahoo-finance-alternative.p.rapidapi.com',
          'x-rapidapi-key': API_KEY
        }
      };

      let response = await axios.request(options)
      for (let value in response.data.ResultSet.Result) {
        this.values.push({
          symbol: response.data.ResultSet.Result[value].symbol,
          name: response.data.ResultSet.Result[value].name,
        });
      }
    },

    totalGyP(hashClient){//Peticion a showTransactions para asi calcular las GyP totales
      let aux; //variable auxiliar para hacer cuentas
      let gp; //pyg por cada transaccion
      axios
            .get('https://finanzyou-back.herokuapp.com/client/showTransactions/' + hashClient)
            .then(response => {
              this.infoTransactions = response.data;
              localStorage.setItem("infoFinances", JSON.stringify(this.info))
            })
            .catch(err => {
              this.showWarningModal(err.response.data);
            })

      for(let item in this.infoTransactions){
          //Aux = precio actual - precio de compra
          aux = this.infoFinances[item.stockID].quoteResponse.result[0].regularMarketPrice - item.buyPrice;
          //gp = aux * cantidad
          gp = aux * this.quantity;
          
          if(this.gyp[item.stockID]==null){
            this.gyp[item.stockID]= gp;
          }
          //en gyp[STOCK] se van sumando todas las transacciones de STOCK hasta que tienes el Gyp total
          else{ 
            this.gyp[item.stockID]+=gp;
          }
      }
    }
  }
});
</script>

<style scoped>

#portfolio {
  margin: 20px;
}

#head{
  padding: 20px;
}

#buttonaddValues{
  margin-top: 10px;
}

.formButton {
  margin: 10px;
}

.gypgreen{
  color: green;
}

.gypred{
  color: red;
}

</style>