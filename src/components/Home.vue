<template>
  <div class="p-6 bg-white rounded-lg shadow-md" v-if="launch">
    <h1 class="text-3xl font-bold text-center mb-4">Prochain Lancement</h1>
    <div class="text-center">
      <p class="text-lg"><strong>Nom de la mission :</strong> {{ launch.name }}</p>
      <p class="text-lg"><strong>Date de lancement :</strong> {{ new Date(launch.date_utc).toLocaleString() }}</p>
      <p class="text-lg"><strong>Décompte :</strong> {{ countdown }} secondes</p>
    </div>
  </div>
  <div v-else class="text-center p-6 bg-white rounded-lg shadow-md">
    <p>Chargement...</p>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      launch: null,
      countdown: 0,
      targetTime: null,
      intervalId: null,
    };
  },
  mounted() {
    this.fetchNextLaunch();
  },
  methods: {
    async fetchNextLaunch() {
      try {
        // Recup le prochain lancement
        const response = await axios.get('https://api.spacexdata.com/v4/launches/next');
        this.launch = response.data;
        // Lance le décompte
        this.startCountdown();
      } catch (error) {
        console.error("Erreur lors de la récupération du prochain lancement :", error);
      }
    },
    startCountdown() {
      if (this.launch && this.launch.date_unix) {
        // Défini l'heure cible à partir de la date Unix 
        this.targetTime = this.launch.date_unix * 1000; // convertir en millisecondes

        // Test verrification compteur avec date aujourd'hui avec h+1 
        // const now = new Date();
        // h+1
        // const oneHourLater = new Date(now.getTime() + 60 * 60 * 1000);
        // Unix en secondes
        // const unixTimestamp = Math.floor(oneHourLater.getTime() / 1000);
        // this.targetTime = unixTimestamp * 1000

        // Maj compteur toutes les secondes
        this.intervalId = setInterval(this.updateCountdown, 1000);
      }
    },
    updateCountdown() {
      const now = new Date().getTime();
      const remainingTime = (this.targetTime - now) / 1000; // temps restant

      // Maj décompte
      this.countdown = Math.max(0, Math.floor(remainingTime));

      // Arrêt quand zéro
      if (this.countdown <= 0) {
        clearInterval(this.intervalId);
      }
    },
  },
};
</script>

<style scoped>

</style>
