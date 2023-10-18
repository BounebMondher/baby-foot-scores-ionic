<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Players</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Players</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-item v-for="player of players" :key="player['id']">
        {{ player['name'] }}
        <ion-button class="delete-player" color="danger" style="right: 20px;position: absolute;" @click="deletePlayer(player['id'])">Delete</ion-button>
      </ion-item>

      <ion-button id="open-players-modal" expand="block">Add new player</ion-button>
      <ion-modal ref="modal" trigger="open-players-modal" @willDismiss="onWillDismiss">
        <ion-header>
          <ion-toolbar>
            <ion-buttons slot="start">
              <ion-button @click="cancel()">Cancel</ion-button>
            </ion-buttons>
            <ion-title>Choose name</ion-title>
            <ion-buttons slot="end">
              <ion-button :strong="true" @click="confirm()">Confirm</ion-button>
            </ion-buttons>
          </ion-toolbar>
        </ion-header>
        <ion-content class="ion-padding">
          <ion-item>
            <ion-input v-model="name" @ionInput="nameChanged($event.target.value)" label="Name" placeholder="Name">
            </ion-input>
          </ion-item>
          <div style="color:red">{{ displayError }}</div>
        </ion-content>
      </ion-modal>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonPage,
  IonButtons,
  IonButton,
  IonModal,
  IonHeader,
  IonContent,
  IonToolbar,
  IonTitle,
  IonItem,
  IonInput,
  IonLabel,
} from '@ionic/vue';
import { OverlayEventDetail } from '@ionic/core/components';
import { defineComponent, ref } from 'vue';
import axios from "axios";

axios.defaults.baseURL = 'http://localhost';

export default defineComponent({
  components: {
    IonPage,
    IonButtons,
    IonButton,
    IonModal,
    IonHeader,
    IonContent,
    IonToolbar,
    IonTitle,
    IonItem,
    IonInput,
    IonLabel,
  },
  data() {
    return {
      name: '',
      displayError: '',
      players: []
    };
  },
  mounted() {
    this.refreshPlayers();
  },
  methods: {
    cancel() {
      this.$refs.modal.$el.dismiss(null, 'cancel');
    },
    confirm() {
      if(this.name=='')
      {
        this.displayError = 'please input a valid name';
      }
      else
        this.$refs.modal.$el.dismiss(null, 'confirm');
    },
    onWillDismiss(ev: CustomEvent<OverlayEventDetail>) {
      if (ev.detail.role === 'confirm') {
        this.addPlayer(this.name);
        this.name="";
      }
    },
    refreshPlayers() {
      axios.get('api/get-players').then(response => {
        this.players = response.data;
      });
    },
    addPlayer(name:string) {
      axios.post('api/add-player', {'name' : name}).then(response => {
        this.refreshPlayers();
      })
    },
    deletePlayer(id: Number) {
      if (confirm("Do you wish to delete this player ?"))
      axios.delete('api/delete-player/'+id).then(response => {
        this.refreshPlayers();
      })
    },
    nameChanged(name:string | number | null | undefined) {
      if(name == '')
      {
        this.displayError = 'please input a valid name';
      }
      else
        this.displayError = "";
    }
  },
});
</script>

