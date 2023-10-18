<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Games history</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Games history</ion-title>
        </ion-toolbar>
      </ion-header>
      <ion-grid v-for="game of games" :key="game['id']">
        <ion-row>
          <ion-col>{{ getPlayer(game["player1"]) }} - {{ getPlayer(game["player2"]) }}</ion-col>
          <ion-col>{{ game["score1"] }}</ion-col>
          <ion-col>{{ game["score2"] }}</ion-col>
          <ion-col>{{ getPlayer(game["player3"]) }} - {{ getPlayer(game["player4"]) }}</ion-col>
          <ion-col>
            <ion-button class="delete-player" color="danger" style="right: 20px;position: absolute;" @click="deleteGame(game['id'])">Delete</ion-button>
          </ion-col>
        </ion-row>
      </ion-grid>
      <ion-button id="open-games-modal" expand="block">Add new game</ion-button>
      <ion-button @click="refreshPlayers()" id="refresh-players" color="warning" expand="block">Refresh players</ion-button>
      <ion-modal ref="modal" trigger="open-games-modal" @willDismiss="onWillDismiss">
        <ion-header>
          <ion-toolbar>
            <ion-buttons slot="start">
              <ion-button @click="cancel()">Cancel</ion-button>
            </ion-buttons>
            <ion-title>Choose players</ion-title>
            <ion-buttons slot="end">
              <ion-button :strong="true" @click="confirm()">Confirm</ion-button>
            </ion-buttons>
          </ion-toolbar>
        </ion-header>
        <ion-content class="ion-padding">
          <ion-label>Team 1</ion-label>
          <ion-item>
            <ion-select label="Player 1" :value="player1" @ionChange="player1 = $event.target.value" placeholder="Player 1">
              <ion-select-option value="">Select Player</ion-select-option>
              <ion-select-option v-for="player of players" :key="player['id']" :value="player['id']">{{ player['name'] }}</ion-select-option>
            </ion-select>
          </ion-item>
          <ion-item>
            <ion-select label="Player 2" :value="player2" @ionChange="player2= $event.target.value" placeholder="Player 2">
              <ion-select-option value="">Select Player</ion-select-option>
              <ion-select-option v-for="player of players" :key="player['id']" :value="player['id']">{{ player['name'] }}</ion-select-option>
            </ion-select>
          </ion-item>
          <ion-item>
            <ion-input v-model="score1" type="number" min="0" label="Score team 1" placeholder="Score team 1">
            </ion-input>
          </ion-item>
          <ion-label>Team 2</ion-label>
          <ion-item>
            <ion-select label="Player 3" :value="player3" @ionChange="player3= $event.target.value" placeholder="Player 3">
              <ion-select-option value="">Select Player</ion-select-option>
              <ion-select-option v-for="player of players" :key="player['id']" :value="player['id']">{{ player['name'] }}</ion-select-option>
            </ion-select>
          </ion-item>
          <ion-item>
            <ion-select label="Player 4" :value="player4" @ionChange="player4= $event.target.value" placeholder="Player 4">
              <ion-select-option value="">Select Player</ion-select-option>
              <ion-select-option v-for="player of players" :key="player['id']" :value="player['id']">{{ player['name'] }}</ion-select-option>
            </ion-select>
          </ion-item>
          <ion-item>
            <ion-input v-model="score2" type="number" min="0" label="Score team 2" placeholder="Score team 2">
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
  IonList, 
  IonSelect,
  IonSelectOption,
  IonGrid,
  IonRow,
  IonCol
} from '@ionic/vue';
import { OverlayEventDetail } from '@ionic/core/components';
import { defineComponent, ref } from 'vue';
import axios from "axios";


axios.defaults.baseURL = 'http://localhost'

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
    IonList, 
    IonSelect,
    IonSelectOption,
    IonGrid,
    IonRow,
    IonCol
  },
  mounted() {
    this.refreshPlayers();
    this.refreshGames();
  },
  data() {
    return {
      displayError: '',
      players: [],
      games: [],
      player1:'',
      player2:'',
      player3:'',
      player4:'',
      score1:0,
      score2:0
    };
  },
  methods: {
    cancel() {
      this.$refs.modal.$el.dismiss(null, 'cancel');
    },
    confirm() {
      if(!this.player1 && !this.player2 || !this.player3 && !this.player4)
      {
        this.displayError = 'please select at least 1 player for each team';
      }
      else
        this.$refs.modal.$el.dismiss(null, 'confirm');
    },
    onWillDismiss(ev: CustomEvent<OverlayEventDetail>) {
      if (ev.detail.role === 'confirm') {
        this.addGame();
      }
    },
    refreshPlayers() {
      axios.get('api/get-players').then(response => {
        this.players = response.data;
      });
    },
    refreshGames() {
      axios.get('api/get-games').then(response => {
        this.games = response.data;
      });
    },
    addGame() {
      axios.post('api/add-game', {'player1' : this.player1, 'player2' : this.player2, 'player3' : this.player3, 'player4' : this.player4, 'score1' : this.score1, 'score2' : this.score2 }).then(response => {
        this.refreshPlayers();
        this.refreshGames();
      })
    },
    deleteGame(id: Number) {
      if (confirm("Do you wish to delete this game ?"))
      axios.delete('api/delete-game/'+id).then(response => {
        this.refreshPlayers();
        this.refreshGames();
      })
    },
    getPlayer(id:number) {
      if (!id) {
        return "";
      }
      let player = this.players.filter(a => a['id'] == id);
      return player[0] ? player[0]['name'] : "deleted player";

    }
  },
});
</script>
