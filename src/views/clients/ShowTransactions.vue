<template>
  <div id="transactions" v-if="showView">
    <div id="head">
      <h1>Movimientos</h1>
    </div>
    <br>
    <b-table-simple striped hover outlined responsive class="table">
      <b-thead>
      <b-tr>
        <b-th scope="col">Nombre</b-th>
        <b-th scope="col">Ticker/ISIN</b-th>
        <b-th scope="col">Cantidad</b-th>
        <b-th scope="col">Precio de compra</b-th>
        <b-th scope="col">Fecha de compra</b-th>

      </b-tr>
      </b-thead>
      <b-tbody>
      <b-tr v-for="(item, index) in this.info" v-bind:key="index" >
        <a :href="'/stock/' + item.stockID">
          <b-td>{{ item.stockName }}</b-td>
        </a>
        <b-td>{{ item.stockID }}</b-td>
        <b-td>{{ item.quantity }}</b-td>
        <b-td v-if="item.currency === 'USD'">{{ item.buyPrice.toFixed(2) + " $" }}</b-td>
        <b-td v-else-if="item.currency === 'EUR'">{{ item.buyPrice.toFixed(2) + " €" }}</b-td>
        <b-td>{{ getDate(item.date) }}</b-td>
      </b-tr>
      </b-tbody>
    </b-table-simple>
    <br>
    <p class="empty_msg" v-if="showEmptyMsg">No hay ningún movimiento</p>
  </div>
</template>

<script>
import axios from "axios";
import CONSTANT from "../../../constants/constants";


export default {
  name: "ShowTransactions",
  data(){
      return{
        info : null,
        showView : false,
        showEmptyMsg: false,
        modal: {
          title: '',
          message: '',
          variant: '',
        },
        values: []
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
      let promises = []

      this.info = JSON.parse(localStorage.getItem("infoTransactions"))

      if(this.info === null) {
        promises.push(axios
            .get( CONSTANT.BACK_URL + 'client/showTransactions/' + hashClient)
            .then(response => {
              this.info = response.data;
              localStorage.setItem("infoTransactions", JSON.stringify(this.info))
              this.showEmptyMsg = this.info.length == 0
            })
            .catch((err) => {
              this.showWarningModal(err.response.data)
              this.showEmptyMsg = true
            }))

        Promise.all(promises)
            .then(() => {
              this.showView = true;
            })
            .catch(() => {
              this.showWarningModal(CONSTANT.ERROR_MSG)
            })
      }
      else {
        this.showView = this.info.length != 0
        this.showEmptyMsg = this.info.length == 0
      }


    },
    getDate(date) {
      let arrayAux = date.split(' ')
      return arrayAux[0] + " " + arrayAux[1] + " " + arrayAux[2]
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

#transactions {
  margin: 20px;
}

#head{
  padding: 20px;
}

a {
  color: black;
}

</style>