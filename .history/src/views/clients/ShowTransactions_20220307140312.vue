<template>
  <div id="transactions" v-if="showView">
    <div id="head">
      <h1>Movimientos</h1>
    </div>
    <div><b-button
        id="deleteValue"
        variant="danger"
        pill
        @click="deleteOption = !deleteOption"
    >Eliminar valor</b-button></div>
    <br>
    <b-table-simple striped hover outlined responsive class="table">
      <b-thead>
      <b-tr>
        <b-th scope="col">Nombre</b-th>
        <b-th scope="col">Ticker/ISIN</b-th>
        <b-th scope="col">Cantidad</b-th>
        <b-th scope="col">Precio de compra</b-th>
        <b-th scope="col">Fecha de compra</b-th>
        <th v-if="deleteOption" scope="col">Eliminar</th>
      </b-tr>
      </b-thead>
      <b-tbody>
      <b-tr v-for="(item, index) in this.info" v-bind:key="index" >
        <a :href="'/stock/' + item.stockID" target="_blank">
          <b-td>{{ item.stockName }}</b-td>
        </a>
        <b-td>{{ item.stockID }}</b-td>
        <b-td>{{ item.quantity }}</b-td>
        <b-td v-if="item.currency === 'USD'">{{ item.buyPrice.toFixed(2) + " $" }}</b-td>
        <b-td v-else-if="item.currency === 'EUR'">{{ item.buyPrice.toFixed(2) + " €" }}</b-td>
        <b-td>{{ getDate(item.date) }}</b-td>
        <b-td v-if="deleteOption"><b-form-checkbox v-model="checked" v-bind:value="item" name="checkbox-button"></b-form-checkbox></b-td>
      </b-tr>
      </b-tbody>
    </b-table-simple>
    <br>
    <div v-if="checked && !showEmptyMsg"><b-button
        v-b-modal.modal-delete-confirmed
        id="deleteConfirmedValue"
        pill variant="outline-danger"
        @click="deleteConfirmed=true"
    >Borrar {{ checked.idStock }}</b-button></div>
    <div v-if="deleteConfirmed">
      <b-modal id="modal-delete-confirmed" title="Confirmación para eliminar" cancel-title="Cancelar" ok-title="Estoy seguro/a" v-on:ok="onSubmit">
        <p class="my-4">¿Está seguro/a que quiere eliminar {{ checked.quantity }} uds de  {{ checked.stockName }} por {{ checked.buyPrice.toFixed(2) + "$" }} a fecha {{ getDate(checked.date) }}?</p>
      </b-modal>
      <p class="empty_msg" v-if="showEmptyMsg">No hay ningún movimiento</p>
      <b-modal id="modal-1" title="BootstrapVue">
        <p class="my-4">Hello from modal!</p>
      </b-modal>
    </div>

    <ModalMessage
        :message="modal.message"
        :title="modal.title"
        :variant="modal.variant"
        class="custom-modal"
    ></ModalMessage>
  </div>
</template>

<script>
import axios from "axios";
import ModalMessage from "@/components/Modal";

export default {
  name: "ShowTransactions",
  // eslint-disable-next-line vue/no-unused-components
  components: {ModalMessage},
  data(){
      return {
        backURL: process.env.VUE_APP_BACK_URL,
        errorMSG: process.env.VUE_APP_ERROR_MSG,
        info : null,
        showView : false,
        showEmptyMsg: false,
        modal: {
          title: '',
          message: '',
          variant: '',
        },
        values: [],
        deleteOption: false,
        checked: false,
        deleteConfirmed: false,
        deleteDone: false
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
            .get( this.backURL + 'client/showTransactions/' + hashClient)
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
              this.showWarningModal(this.errorMSG)
            })
      }
      else {
        this.showView = this.info.length != 0
        this.showEmptyMsg = this.info.length == 0
      }
    },
    getDate(date) {
      let arrayAux = date.split(' ')
      return arrayAux[0] + " " + arrayAux[1] + " " + arrayAux[2].split(',')[0]
    },
    showSuccessModal() {
      this.$bvModal.show("modal")
      this.modal.title = "¡Operación Exitosa!"
      this.modal.message = "Se ha eliminado correctamente"
      this.modal.variant = 'success'
    },
    showWarningModal(message) {
      this.$bvModal.show("modal")
      this.modal.message = message
      this.modal.title = "¡Operación Fallida!"
      this.modal.variant = 'warning'
    },
    onSubmit() {
      let transaction = {};
      transaction.id = this.checked.id;

      axios
          .post( this.backURL + 'client/deleteTransaction/', transaction)
          .then(() => {
            //Aviso de que se ha borrado correctamente.
            localStorage.clear()
            this.showSuccessModal()
            this.getData();
          })
          .catch((err) => {
            //Aviso de que no se ha podido borrar.
            this.showWarningModal("Error al eliminar")
            console.log(err)
          })
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