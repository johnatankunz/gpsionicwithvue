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
        <h3>{{ latitude }}</h3>
        <h3>{{ longitude }}</h3>
        <ion-button @click="buscaSalvaCidade()">Qual cidade?</ion-button>
        <h3>{{ cidade }}</h3>
      </div>

      <ion-row>
        <ion-col>
          <ul v-for="(cidade, index) in list" :key="index"> 
            <h1> {{cidade}} </h1>
            <p> {{ latitude}}, {{longitude}} </p>
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
      list: []
    };
  },
  created: async function() {
    await this.localStorage.create();
  },
  mounted: async function() {
    this.cidade = await this.localStorage.get(this.cidade);
  },
  ionViewWillEnter() {
    this.printCurrentPosition();
  },
  methods: {
    printCurrentPosition: async function() {
      const coordinates = await Geolocation.getCurrentPosition();
      console.log("Current position:", coordinates);
      this.latitude = coordinates.coords.latitude;
      this.longitude = coordinates.coords.longitude;
    },
    buscaSalvaCidade: async function() {
      const ACCESS_KEY = "33e72439933d8f0e72bff0671cae589f";
      const options = {
        url: `http://api.positionstack.com/v1/reverse?access_key=${ACCESS_KEY}&query=${this.latitude},${this.longitude}`
      };
      const response = await Http.get(options);
      console.log(response);
      this.cidade = response.data.data[0].locality + ', ' + response.data.data[0].region_code;
      this.localStorage.set('cidade', this.cidade);
      this.localStorage.set('latitude', this.latitude);
      this.localStorage.set('longitude', this.longitude);

      console.log(this.cidade);
      this.list.push(this.cidade);
      
    }
  },
});
</script>

<style scoped>
.container {
  text-align: center;
}
</style>
