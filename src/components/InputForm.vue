<template>
  <div class="w-full max-w-4xl mx-auto p-6">
    <!-- Sélecteur de mode -->
    <div class="mb-8">
      <div class="flex items-center justify-center space-x-4">
        <span :class="{ 'text-green-400 font-medium': !isManualMode, 'text-gray-300': isManualMode }">
          Upload de fichier
        </span>
        <button 
          @click="toggleMode"
          class="relative inline-flex h-6 w-11 items-center rounded-full transition-colors"
          :class="isManualMode ? 'bg-green-400' : 'bg-gray-600'"
        >
          <span
            class="inline-block h-4 w-4 transform rounded-full bg-white transition-transform"
            :class="isManualMode ? 'translate-x-6' : 'translate-x-1'"
          />
        </button>
        <span :class="{ 'text-green-400 font-medium': isManualMode, 'text-gray-300': !isManualMode }">
          Saisie manuelle
        </span>
      </div>
    </div>

    <!-- Zone de contenu dynamique -->
    <div class="mt-6">
      <Transition
        enter-active-class="transition-opacity duration-300"
        leave-active-class="transition-opacity duration-300"
        enter-from-class="opacity-0"
        leave-to-class="opacity-0"
      >
        <FileUploader v-if="!isManualMode" @file-selected="handleFileSelected" />
        <TextEditor v-else v-model="manualText" />
      </Transition>
    </div>

    <!-- Ajout du message d'erreur -->
    <div v-if="errorMessage" class="mt-4 text-red-400 text-center">
      {{ errorMessage }}
    </div>

    <!-- Ajout du message de succès -->
    <div v-if="successMessage" class="mt-4 text-green-400 text-center">
      {{ successMessage }}
    </div>

    <!-- Bouton de soumission -->
    <div class="mt-8 flex justify-center">
      <button
        @click="handleSubmit"
        :disabled="isLoading || !isValid"
        class="group relative px-8 py-3 bg-black/40 border-2 border-green-500 rounded-lg 
               hover:bg-green-500/20 transition-all duration-300 overflow-hidden
               disabled:opacity-50 disabled:cursor-not-allowed"
      >
        <span class="relative z-10 text-green-400 font-medium text-lg group-hover:text-green-300 
                     transition-colors flex items-center gap-2"
        >
          <span>{{ isLoading ? 'Traitement...' : 'Soumettre' }}</span>
          <span v-if="!isLoading" class="matrix-chars"></span>
        </span>
        <div class="absolute inset-0 flex justify-center items-center opacity-0 group-hover:opacity-100 
                    transition-opacity duration-300"
        >
          <div class="matrix-glow"></div>
        </div>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import axios from 'axios'
import FileUploader from './FileUploader.vue'
import TextEditor from './TextEditor.vue'

const isManualMode = ref(false)
const manualText = ref('')
const isLoading = ref(false)
const errorMessage = ref('')
const successMessage = ref('')

const toggleMode = () => {
  isManualMode.value = !isManualMode.value
}

const handleFileSelected = async (file) => {
  try {
    const content = await readFileContent(file)
    manualText.value = content
  } catch (error) {
    errorMessage.value = "Erreur lors de la lecture du fichier"
    console.error('Erreur de lecture:', error)
  }
}

// Nouvelle fonction utilitaire pour lire le contenu du fichier
const readFileContent = (file) => {
  return new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = (e) => resolve(e.target.result)
    reader.onerror = (e) => reject(e)
    reader.readAsText(file)
  })
}

const handleSubmit = async () => {
  try {
    isLoading.value = true
    errorMessage.value = ''
    successMessage.value = ''

    // Validation des données
    if (!manualText.value.trim()) {
      throw new Error('Le contenu ne peut pas être vide')
    }

    // Préparation des données
    const payload = {
      content: manualText.value.trim(),
      mode: isManualMode.value ? 'manual' : 'file'
    }

    // Soumission vers l'API de traitement et d'envoi automatique des messages
    const response = await axios.post( `${import.meta.env.VITE_API_URL}/api/submit`, payload, {
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${import.meta.env.VITE_API_KEY}`
      }
    })

    // Traitement de la réponse
    if (response.data) {
      // Réinitialisation après succès
      manualText.value = ''
      localStorage.removeItem('editor-content')
      successMessage.value = response.data.message || 'Le contenu a été soumis avec succès !'
    }

  } catch (error) {
    errorMessage.value = error.response?.data?.message || error.message || 'Une erreur est survenue'
    console.error('Erreur de soumission:', error)
  } finally {
    isLoading.value = false
  }
}

// Ajout d'une propriété calculée pour la validation
const isValid = computed(() => {
  return manualText.value.trim().length > 0
})
</script>

<style scoped>
.matrix-chars::after {
  content: '01';
  animation: matrix-chars 1s steps(2) infinite;
}

@keyframes matrix-chars {
  0% { content: '01'; }
  50% { content: '10'; }
  100% { content: '01'; }
}

.matrix-glow {
  width: 100%;
  height: 100%;
  background: radial-gradient(circle, rgba(0,255,0,0.2) 0%, transparent 70%);
  animation: glow 2s ease-in-out infinite;
}

@keyframes glow {
  0% { transform: scale(1); opacity: 0.5; }
  50% { transform: scale(1.2); opacity: 0.8; }
  100% { transform: scale(1); opacity: 0.5; }
}
</style> 