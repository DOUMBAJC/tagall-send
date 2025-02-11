<template>
  <div
    @dragover.prevent="handleDragOver"
    @dragleave.prevent="handleDragLeave"
    @drop.prevent="handleDrop"
    @click="triggerFileInput"
    :class="[
      'border-2 border-dashed rounded-lg p-4 sm:p-8',
      'flex flex-col items-center justify-center',
      'cursor-pointer transition-colors min-h-[150px] sm:min-h-[200px]',
      isDragging ? 'border-green-400 bg-black/50' : 'border-gray-400 hover:border-green-400 bg-black/30'
    ]"
  >
    <input
      type="file"
      ref="fileInput"
      class="hidden"
      @change="handleFileChange"
      accept=".vcf"
    >
    
    <svg class="w-8 h-8 sm:w-12 sm:h-12 text-gray-400 mb-2 sm:mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" 
        d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M15 13l-3-3m0 0l-3 3m3-3v12" />
    </svg>

    <p class="text-base sm:text-lg text-gray-200 text-center">
      {{ isDragging ? 'Déposez le fichier ici' : 'Déposez votre fichier ici ou cliquez pour parcourir' }}
    </p>
    
    <p class="text-xs sm:text-sm text-gray-400 mt-2">
      Formats acceptés: .vcf
    </p>
    <p v-if="errorMessage" class="text-red-400 text-sm mt-2">
      {{ errorMessage }}
    </p>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const emit = defineEmits(['file-selected'])
const fileInput = ref(null)
const isDragging = ref(false)
const errorMessage = ref('')

const handleDragOver = () => {
  isDragging.value = true
}

const handleDragLeave = () => {
  isDragging.value = false
}

const handleDrop = (e) => {
  isDragging.value = false
  const file = e.dataTransfer.files[0]
  if (file) {
    validateAndEmitFile(file)
  }
}

const triggerFileInput = () => {
  fileInput.value.click()
}

const handleFileChange = (e) => {
  const file = e.target.files[0]
  if (file) {
    validateAndEmitFile(file)
  }
}

const validateAndEmitFile = (file) => {
  const validTypes = ['.vcf']
  const fileExtension = '.' + file.name.split('.').pop().toLowerCase()
  
  if (!validTypes.includes(fileExtension)) {
    errorMessage.value = 'Format de fichier non supporté. Veuillez utiliser un fichier de contacts (.vcf)'
    return
  }
  
  errorMessage.value = ''
  emit('file-selected', file)
}
</script> 