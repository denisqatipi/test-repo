<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import { useTransformStore } from '../stores/transform'
import MessageIn from '../components/MessageIn.vue'
import MessageOut from '../components/MessageOut.vue'
import XmlTreeViewer from '../components/xml/XmlTreeViewer.vue'
import JsonTreeViewer from '../components/json/JsonTreeViewer.vue'
import TransformView from '../components/transform/TransformView.vue'
import AppLogo from '../components/AppLogo.vue'

const transformStore = useTransformStore()
const currentStep = ref(transformStore.currentStep)
const totalSteps = 3
const showXmlViewer = ref(false)
const showJsonViewer = ref(false)
const uploadedFile = ref<File | null>(null)
const xmlData = ref<any>(null)
const jsonData = ref<any>(null)

const steps = [
  { 
    number: 1, 
    title: 'Message In',
    description: 'Select your input message type and file',
    icon: 'pi-file-import'
  },
  { 
    number: 2, 
    title: 'Message Out',
    description: 'Configure your output message format',
    icon: 'pi-file-export'
  },
  { 
    number: 3, 
    title: 'Transform',
    description: 'Map fields and transform your message',
    icon: 'pi-sync'
  }
]

// Watch for transform store changes
watch(() => transformStore.currentStep, (newStep) => {
  currentStep.value = newStep
})

// Initialize state from store
onMounted(() => {
  // Reset all state when component mounts
  transformStore.reset()
  resetLocalState()
})

const resetLocalState = () => {
  showXmlViewer.value = false
  showJsonViewer.value = false
  uploadedFile.value = null
  xmlData.value = null
  jsonData.value = null
  currentStep.value = 1
}

const handleFileUpload = (file: File, data: any) => {
  uploadedFile.value = file
  xmlData.value = data
  showXmlViewer.value = true
  transformStore.setMessageIn(file, data, 'XML')
}

const handleJsonFileUpload = (file: File, data: any) => {
  uploadedFile.value = file
  jsonData.value = data
  showJsonViewer.value = true
  transformStore.setMessageOut(file, data, 'JSON')
}

const handleXmlSave = () => {
  showXmlViewer.value = false
  currentStep.value = 2
  transformStore.setCurrentStep(2)
}

const handleJsonSave = () => {
  showJsonViewer.value = false
  currentStep.value = 3
  transformStore.setCurrentStep(3)
}

const navigateToStep = (stepNumber: number) => {
  if (stepNumber <= currentStep.value) {
    if (stepNumber === 1) {
      showJsonViewer.value = false
      if (transformStore.messageIn.file) {
        showXmlViewer.value = true
      }
    } else if (stepNumber === 2) {
      showXmlViewer.value = false
      if (transformStore.messageOut.file) {
        showJsonViewer.value = true
      }
    } else if (stepNumber === 3) {
      showXmlViewer.value = false
      showJsonViewer.value = false
    }
    currentStep.value = stepNumber
    transformStore.setCurrentStep(stepNumber)
  }
}

// Watch for store resets
watch(() => transformStore.messageIn.file, (newFile) => {
  if (!newFile) {
    showXmlViewer.value = false
    uploadedFile.value = null
    xmlData.value = null
  }
})

watch(() => transformStore.messageOut.file, (newFile) => {
  if (!newFile) {
    showJsonViewer.value = false
    jsonData.value = null
  }
})
</script>