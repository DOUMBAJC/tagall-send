<template>
  <div class="w-full">
    <div class="flex flex-wrap items-center gap-1 sm:gap-2 mb-2 p-2 bg-black/50 rounded-t-lg border border-gray-600">
      <button
        v-for="tool in tools"
        :key="tool.label"
        @click="applyFormat(tool.format)"
        class="p-1.5 sm:p-2 hover:bg-black/70 rounded transition-colors"
        :title="tool.label"
      >
        <span class="text-gray-300 text-sm sm:text-base">{{ tool.icon }}</span>
      </button>
    </div>

    <div class="relative">
      <textarea
        ref="textareaRef"
        v-model="localContent"
        @input="autoResize"
        :placeholder="'Commencez à écrire ici...'"
        class="w-full min-h-[150px] sm:min-h-[200px] p-3 sm:p-4 bg-black/30 text-gray-200 
               border border-gray-600 rounded-b-lg focus:ring-2 focus:ring-green-500 
               focus:border-transparent outline-none resize-none placeholder-gray-500
               text-sm sm:text-base"
        :class="{ 'border-red-500': hasError }"
      ></textarea>

      <div class="absolute bottom-1 sm:bottom-2 right-1 sm:right-2 text-xs sm:text-sm text-gray-400">
        {{ characterCount }} caractères
      </div>
    </div>

    <div v-if="hasError" class="mt-2 text-xs sm:text-sm text-red-400">
      {{ errorMessage }}
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  },
  maxLength: {
    type: Number,
    default: 250000
  }
})

const emit = defineEmits(['update:modelValue'])

const textareaRef = ref(null)
const localContent = ref(props.modelValue)
const hasError = ref(false)
const errorMessage = ref('')

// Outils de formatage basiques
const tools = [
  { label: 'Gras', icon: 'B', format: '**' },
  { label: 'Italique', icon: 'I', format: '_' },
  { label: 'Liste', icon: '•', format: '- ' }
]

// Compteur de caractères
const characterCount = computed(() => localContent.value.length)

// Auto-redimensionnement du textarea
const autoResize = () => {
  const textarea = textareaRef.value
  textarea.style.height = 'auto'
  textarea.style.height = textarea.scrollHeight + 'px'
}

// Application du formatage
const applyFormat = (format) => {
  const textarea = textareaRef.value
  const start = textarea.selectionStart
  const end = textarea.selectionEnd
  const text = localContent.value

  if (format === '- ') {
    // Pour les listes, ajouter au début de la ligne
    const beforeCursor = text.substring(0, start)
    const afterCursor = text.substring(end)
    const newLine = '\n- '
    localContent.value = beforeCursor + newLine + afterCursor
  } else {
    // Pour le gras et l'italique
    const selectedText = text.substring(start, end)
    const newText = format + selectedText + format
    localContent.value = text.substring(0, start) + newText + text.substring(end)
  }
}

// Validation
const validate = () => {
  if (characterCount.value > props.maxLength) {
    hasError.value = true
    errorMessage.value = `Le texte dépasse la limite de ${props.maxLength} caractères`
    return false
  }
  hasError.value = false
  errorMessage.value = ''
  return true
}

// Sauvegarde automatique dans le localStorage
const saveToLocalStorage = () => {
  localStorage.setItem('editor-content', localContent.value)
}

// Surveillance des changements
watch(localContent, (newValue) => {
  emit('update:modelValue', newValue)
  validate()
  saveToLocalStorage()
}, { debounce: 300 })

// Initialisation
onMounted(() => {
  // Restaurer le contenu sauvegardé
  const savedContent = localStorage.getItem('editor-content')
  if (savedContent) {
    localContent.value = savedContent
  }
  
  autoResize()
})
</script> 