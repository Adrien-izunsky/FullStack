<template>
  <div class="wrapper">
    <form @submit.prevent="handleSubmit">
      <!-- Choix de la personne -->
      <div class="input-group">
        <label for="person">Choisir une personne :</label>
        <select v-model="person" required>
          <option v-for="individu in individus" :key="individu.matricule" :value="individu.matricule">
            {{ individu.nom }} {{ individu.prenom }}
          </option>
        </select>
      </div>

      <!-- Choix du projet -->
      <div class="input-group">
        <label for="project">Choisir un projet :</label>
        <select v-model="projet" required>
          <option v-for="item in projets" :key="item.id" :value="item.id">
            {{ item.nom }}
          </option>
        </select>
      </div>

      <!-- Entrée du rôle -->
      <div class="input-group">
        <label for="role">Fonction :</label>
        <input type="text" v-model="fonction" required />
      </div>

      <!-- Curseur de pourcentage -->
      <div class="input-group">
        <label for="participation">Niveau de participation :</label>
        <input
          type="range"
          v-model="participation"
          min="0"
          max="100"
          step="1"
          @input="afficherPourcentage"
          :style="{ background: calculerFondCurseur() }"
        />
        <div class="pourcentage-valeur">
          {{ participation }}%
        </div>
      </div>

      <!-- Bouton d'envoi -->
      <div class="bouton-zone">
        <button type="submit">Soumettre</button>
      </div>
    </form>

    <ErreurMessage v-if="messageErreur" :message="messageErreur" />
    <p v-if="messageSucces" class="succes">{{ messageSucces }}</p>
  </div>
</template>

<script>
import axios from 'axios';
import ErreurMessage from './ErreurMessage.vue';

export default {
  components: { ErreurMessage },
  data() {
    return {
      individus: [],
      projets: [],
      person: null,
      projet: null,
      fonction: '',
      participation: 50,
      messageErreur: null,
      messageSucces: null,
    };
  },
  mounted() {
    this.chargerIndividus();
    this.chargerProjets();
  },
  methods: {
    async chargerIndividus() {
      try {
        const { data } = await axios.get("http://localhost:8989/api/personnes");
        this.individus = data;
      } catch (err) {
        console.error("Erreur de chargement des individus :", err);
        this.messageErreur = "Impossible de récupérer la liste des personnes.";
      }
    },

    async chargerProjets() {
      try {
        const { data } = await axios.get("http://localhost:8989/api/projets");
        this.projets = data;
      } catch (err) {
        console.error("Erreur de chargement des projets :", err);
        this.messageErreur = "Impossible de récupérer la liste des projets.";
      }
    },

    afficherPourcentage() {
      console.log(`Participation actuelle : ${this.participation}%`);
    },

    async handleSubmit() {
      const donnees = {
        matricule: this.person,
        codeProjet: this.projet,
        role: this.fonction,
        pourcentage: this.participation / 100,
      };

      try {
        await axios.post("/api/gestion/participation", donnees, {
          headers: { "Content-Type": "application/json" },
        });
        this.messageSucces = "Données enregistrées avec succès !";
        this.messageErreur = null;
      } catch (err) {
        this.messageErreur = err.response?.data?.message || "Erreur lors de l'envoi des données.";
        this.messageSucces = null;
      }
    },

    calculerFondCurseur() {
      return `linear-gradient(to right, green ${this.participation}%, #e0e0e0 ${this.participation}%)`;
    },
  },
};
</script>

<style scoped>
.wrapper {
  padding: 25px;
  border: 1px solid #ddd;
  border-radius: 10px;
  background-color: #f0f0f0;
  max-width: 800px;
  margin: 0 auto;
}

.input-group {
  margin-bottom: 15px;
  display: flex;
  flex-direction: column;
}

.input-group label {
  margin-bottom: 5px;
  font-weight: bold;
}

input, select {
  padding: 8px 10px;
  border-radius: 5px;
  border: 1px solid #ccc;
}

input[type="range"] {
  height: 8px;
  appearance: none;
  border-radius: 5px;
}

input[type="range"]::-webkit-slider-thumb {
  appearance: none;
  width: 18px;
  height: 18px;
  background-color: green;
  border-radius: 50%;
}

.pourcentage-valeur {
  margin-top: 5px;
  font-size: 16px;
  color: #333;
}

.bouton-zone {
  display: flex;
  justify-content: center;
}

button {
  padding: 10px 20px;
  background-color: green;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: darkgreen;
}

.succes {
  color: green;
  margin-top: 10px;
}

.error {
  color: red;
  margin-top: 10px;
}
</style>
