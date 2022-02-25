<template>
  <div class="container" v-if="showView">
    <a :href="this.infoStock.results.homepage_url" target="_blank">
      <div class="titleStock">
        <b-img v-if="this.infoStock.results.branding.logo_url !== null" class="logo" :src="this.infoStock.results.branding.logo_url + '?apikey=' + this.apiKey" fluid alt="Responsive image"></b-img>
        <h1>{{ this.infoStock.results.name }}</h1>
      </div>
    </a>
    <p class="stockDescription" v-if="this.infoStock.results.hasOwnProperty('description')">
      {{ this.infoStock.results.description }}
    </p>
    <p class="stockDescription" v-else>Descripción no disponible</p>
    <br>
    <hr>
    <br>
    <div class="news">
      <h4><b>5 últimas noticias</b></h4>
      <br>
      <b-card v-for="item in news.results" v-bind:key="item" no-body class="overflow-auto newsCard text-center mb-3 card-item">
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
import axios from "axios";

export default {
  name: "Stock",
  data() {
    return {
      apiKey: process.env.VUE_APP_APIKEY_FOR_STOCK_INFO,
      backURL: process.env.VUE_APP_BACK_URL,
      errorMSG: process.env.VUE_APP_ERROR_MSG,
      infoStock: {},
      id: this.$route.params.id,
      showView: false,
      news: {},
      modal: {
        title: '',
        message: '',
        variant: '',
      }
    }
  },
  created() {
    this.getData()
  },
  methods: {
    async getData() {
      try {
        let responseInfo = await axios.get("https://api.polygon.io/v3/reference/tickers/" + this.id + "?apikey=" + this.apiKey)
        let responseNews = await axios.get("https://api.polygon.io/v2/reference/news?ticker=" + this.id + "&limit=5&apiKey=" + this.apiKey)
        if(responseInfo === null || responseNews === null) this.showWarningModal(this.errorMSG)
        else {
          this.infoStock = responseInfo.data
          this.news = responseNews.data
          this.showView = true
        }
      }
      catch (err) {
        console.error(err)
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

.container {
  display: block;
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
  padding: 35px;
}

hr {
  border: none;
  height: 1px;
  color: #333;
  background-color: #333;
}

.newsCard {
  max-width: 540px;
  max-height: 160px;
}

</style>