<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Qual cidade estou?</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content>
      <div class="container">
        <h2>Coordenadas</h2>
        <h3>{{ latitude }}, {{ longitude }}</h3>
        <ion-button @click="buscaSalvaCidade()">Qual cidade?</ion-button>
      </div>

      <ion-row>
        <ion-col>
          <ul v-for="(cidade, index) in list" :key="index">
            <h1> {{ cidade }} </h1>
            <p> {{ latitude }}, {{ longitude }} </p>
          </ul>
        </ion-col>
      </ion-row>
    </ion-content>
  </ion-page>
</template>

<script>
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar,
  IonButton,
  IonRow,
  IonCol
} from "@ionic/vue";
import { defineComponent } from "vue";
import { Geolocation } from "@capacitor/geolocation";
import { Http } from "@capacitor-community/http";
import { Storage} from "@ionic/storage";

export default defineComponent({
  name: "Home",
  components: {
    IonPage,
    IonHeader,
    IonTitle,
    IonToolbar,
    IonContent,
    IonButton,
    IonRow,
    IonCol
  },

  data() {
    return {
      latitude: 0,
      longitude: 0,
      cidade: '',
      localStorage: new Storage(),
      list: [],
      lastFive: 0
    };
  },
  created: async function() {
    await this.localStorage.create();
  },
  mounted: async function() {
    const list = await this.localStorage.get('list');
    this.list = JSON.parse(list) || [];
    this.lastFive = this.list.slice(-5);
  },
  ionViewWillEnter() {
    this.printCurrentPosition();
  },
  methods: {
    printCurrentPosition: async function() {
      const coordinates = await Geolocation.getCurrentPosition();
      this.latitude = coordinates.coords.latitude;
      this.longitude = coordinates.coords.longitude;
    },
    buscaSalvaCidade: async function() {
      this.lastFive = ++this.lastFive;
      const ACCESS_KEY = "33e72439933d8f0e72bff0671cae589f";
      const options = {
        url: `http://api.positionstack.com/v1/reverse?access_key=${ACCESS_KEY}&query=${this.latitude},${this.longitude}`
      };
      const response = await Http.get(options);
      this.cidade = response.data.data[0].locality + ', ' + response.data.data[0].region_code;
      this.localStorage.set('cidade', this.cidade);
      this.localStorage.set('latitude', this.latitude);
      this.localStorage.set('longitude', this.longitude);

      if (this.lastFive <= 5){
        this.list.push(this.cidade);
        }
        await this.localStorage.set('list', JSON.stringify(this.list));
    }
  },
});
</script>

<style scoped>
.container {
  text-align: center;
}
</style>
