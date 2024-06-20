<template>
  <div class="p-8">
    <h2 class="text-3xl font-bold text-center mb-6">Filtrer les lancements</h2>
    <div class="mb-6 text-center">
      <select v-model="filter" @change="filterLaunches" class="p-2 border border-gray-300 rounded">
        <option value="all">Tous les lancements</option>
        <option value="success">Lancements réussis</option>
        <option value="failure">Lancements échoués</option>
      </select>
    </div>
    <div v-if="loading" class="text-center">
      <p>Chargement...</p>
    </div>
    <ul v-else class="space-y-6">
      <li v-for="launch in limitedLaunches" :key="launch.id" class="p-6 bg-white rounded-lg shadow-md">
        <div class="flex flex-col md:flex-row md:items-center md:justify-between">
          <div class="mb-4 md:mb-0">
            <h2 class="text-xl font-bold">{{ launch.name }}</h2>
            <p class="text-gray-500">{{ formatDate(launch.date_unix) }}</p>
            <p class="mt-2">{{ launch.details }}</p>
          </div>
          <div class="mb-4 md:mb-0 md:flex-shrink-0 md:ml-4">
            <img :src="launch.links.patch.small" alt="Mission Patch" class="w-24 h-24 object-cover rounded">
          </div>
        </div>
        <div class="mt-4">
          <p v-if="launch.landpadDetails.name != 'N/A'"><strong>Lieu de lancement:</strong> {{ launch.landpadDetails.name }}, {{ launch.landpadDetails.locality }}, {{ launch.landpadDetails.region }}</p>
          <p v-else><strong>Lieu de lancement:</strong> Iconnus</p>
          <p v-if="launch.payloadDetails.names.length != 0"><strong>Payloads:</strong> {{ launch.payloadDetails.names.join(', ') }}</p>
          <p v-else><strong>Payloads:</strong> Iconnus</p>
          <p v-if="launch.payloadDetails.customers.length != 0"><strong>Clients:</strong> {{ launch.payloadDetails.customers.join(', ') }}</p>
          <p v-else><strong>Clients:</strong> Iconnus</p>
        </div>
        <div class="mt-4 flex space-x-4">
          <a v-if="launch.links.article" :href="launch.links.article" target="_blank" class="text-blue-500 hover:underline">Article</a>
          <button v-if="launch.links.webcast" @click.prevent="showVideo(launch.links.webcast)" class="text-blue-500 hover:underline">Vidéo</button>
        </div>
      </li>
    </ul>
    <div v-if="showModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center">
      <div class="bg-white p-4 rounded-lg shadow-lg w-full max-w-5xl h-4/5">
        <button @click="closeModal" class="text-gray-500 float-right">&times;</button>
        <iframe :src="videoUrl" frameborder="0" allowfullscreen class="w-full h-full"></iframe>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      filter: 'all',
      launches: [],
      filteredLaunches: [],
      loading: false,
      showModal: false,
      videoUrl: ''
    };
  },
  mounted() {
    this.fetchLaunches();
  },
  computed: {
    limitedLaunches() {
      return this.filteredLaunches.slice(-10).reverse();
    }
  },
  methods: {
    async fetchLaunches() {
      this.loading = true;
      try {
        // Recup les lancement
        const response = await axios.get('https://api.spacexdata.com/v5/launches', { timeout: 10000 });
        this.launches = response.data;
        await this.fetchAdditionalDetails();
        this.filterLaunches();
      } catch (error) {
        console.error("Erreur lors de la récupération des lancements :", error);
      } finally {
        this.loading = false;
      }
    },
    async fetchAdditionalDetails() {
      for (const launch of this.launches) {
        try {
          // Fetch landpad details
          if (launch.cores[0].landpad) {
            const landpadResponse = await axios.get(`https://api.spacexdata.com/v4/landpads/${launch.cores[0].landpad}`, { timeout: 10000 });
            launch.landpadDetails = landpadResponse.data;
          } else {
            launch.landpadDetails = { name: 'N/A', locality: 'N/A', region: 'N/A' };
          }

          // Fetch payload details
          launch.payloadDetails = { names: [], customers: [] };
          for (const payloadId of launch.payloads) {
            const payloadResponse = await axios.get(`https://api.spacexdata.com/v4/payloads/${payloadId}`, { timeout: 10000 });
            launch.payloadDetails.names.push(payloadResponse.data.name);
            console.log(launch.payloadDetails.names)
            launch.payloadDetails.customers.push(...payloadResponse.data.customers);
          }
        } catch (error) {
          console.error(`Erreur lors de la récupération des détails supplémentaires pour le lancement ${launch.id} :`, error);
        }
      }
    },
    filterLaunches() {
      // Permet de filtrer la data 
      if (this.filter === 'all') {
        this.filteredLaunches = this.launches;
      } else if (this.filter === 'success') {
        this.filteredLaunches = this.launches.filter(launch => launch.success === true);
      } else if (this.filter === 'failure') {
        this.filteredLaunches = this.launches.filter(launch => launch.success === false);
      }
    },
    formatDate(unixTimestamp) {
      const date = new Date(unixTimestamp * 1000);
      return date.toLocaleDateString();
    },
    showVideo(url) {
      this.videoUrl = url.replace("watch?v=", "embed/");
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
      this.videoUrl = '';
    }
  },
  watch: {
    filter() {
      this.filterLaunches();
    }
  }
};
</script>

<style scoped>
a {
  cursor: pointer;
}
</style>
