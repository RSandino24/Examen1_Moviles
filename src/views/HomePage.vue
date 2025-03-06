<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>GitHub Repository Finder</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div class="container">
        <ion-card class="card-centered">
          <ion-card-header>
            <ion-select
              placeholder="Select a Language"
              v-model="selectedLanguage"
              @ionChange="fetchRandomRepo"
            >
              <ion-select-option v-for="language in languages" :key="language" :value="language">
                {{ language }}
              </ion-select-option>
            </ion-select>
          </ion-card-header>

          <div v-if="state === 'empty'" class="state-message">
            Please select a language
          </div>

          <div v-if="state === 'loading'" class="state-message">
            Loading, please wait...
          </div>

          <div v-if="state === 'error'" class="state-message error">
            Error fetching repositories
            <ion-button expand="block" color="danger" @click="fetchRandomRepo">Click to retry</ion-button>
          </div>

          <div v-if="state === 'success'">
            <ion-card class="repo-card">
              <ion-card-header class="repo-header">
                <ion-card-title>{{ repository.name }}</ion-card-title>
                <ion-icon
                  slot="end"
                  :icon="openOutline"
                  @click="openRepo(repository.html_url)"
                ></ion-icon>
              </ion-card-header>
              <ion-card-content>
                {{ repository.description || 'No description available' }}
              </ion-card-content>
              <div class="repo-info">
                <ion-chip><ion-label>🟡 {{ repository.language }}</ion-label></ion-chip>
                <ion-chip><ion-label>⭐ {{ repository.stars }}</ion-label></ion-chip>
                <ion-chip><ion-label>🔗 {{ repository.forks }}</ion-label></ion-chip>
                <ion-chip><ion-label>⚠️ {{ repository.issues }}</ion-label></ion-chip>
              </div>
            </ion-card>

            <ion-button expand="block" @click="fetchRandomRepo">Refresh</ion-button>
          </div>
        </ion-card>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref } from 'vue';
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonCard, IonCardHeader, IonCardTitle,
  IonCardContent, IonButton, IonSelect, IonSelectOption, IonChip, IonLabel, IonIcon
} from '@ionic/vue';
import { openOutline } from 'ionicons/icons';

const GITHUB_TOKEN = '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';

const selectedLanguage = ref('');
const state = ref('empty');
const repository = ref({});

const languages = ['JavaScript', 'Python', 'Java', 'C++', 'Go', 'PHP', 'Ruby'];

const fetchRandomRepo = async () => {
  if (!selectedLanguage.value) {
    state.value = 'empty';
    return;
  }

  state.value = 'loading';

  try {
    const response = await fetch(
      `https://api.github.com/search/repositories?q=language:${selectedLanguage.value}&sort=stars&order=desc`,
      {
        headers: {
          Authorization: `token ${GITHUB_TOKEN}`
        }
      }
    );

    const data = await response.json();

    if (data.items && data.items.length > 0) {
      const randomIndex = Math.floor(Math.random() * data.items.length);
      const repo = data.items[randomIndex];
      repository.value = {
        name: repo.name,
        description: repo.description,
        language: repo.language,
        stars: repo.stargazers_count,
        forks: repo.forks_count,
        issues: repo.open_issues_count,
        html_url: repo.html_url,
      };

      state.value = 'success';
    } else {
      throw new Error('No repositories found');
    }
  } catch (error) {
    state.value = 'error';
  }
};

const openRepo = (url) => {
  window.open(url, '_blank');
};
</script>

<style scoped>
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 90vh;
}
.card-centered {
  width: 90%;
  max-width: 400px;
}
.state-message {
  text-align: center;
  padding: 15px;
  color: #555;
}
.state-message.error {
  color: #fff;
  background: #f44336;
  padding: 15px;
  border-radius: 5px;
  margin-top: 10px;
}
.repo-card {
  margin-top: 15px;
}
.repo-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.repo-info {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  margin-top: 10px;
}
ion-chip {
  font-size: 12px;
}
</style>
