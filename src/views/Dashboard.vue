<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { useAuthStore } from '../stores/auth'
import { useChannelStore } from '../stores/channel'
import { useStatsStore } from '../stores/stats'
import { useTransformStore } from '../stores/transform'
import AppLogo from '../components/AppLogo.vue'
import ChannelUploadDialog from '../components/ChannelUploadDialog.vue'
import TransformResultDialog from '../components/TransformResultDialog.vue'

const router = useRouter()
const authStore = useAuthStore()
const channelStore = useChannelStore()
const statsStore = useStatsStore()
const transformStore = useTransformStore()

const stats = ref([
  { label: 'Total Transformations', value: '0', icon: 'pi-sync' },
  { label: 'This Month', value: '0', icon: 'pi-calendar' },
  { label: 'Success Rate', value: '0%', icon: 'pi-check-circle' },
  { label: 'Processing', value: '0', icon: 'pi-spinner' }
])

const loading = ref(true)
const showUploadDialog = ref(false)
const showResultDialog = ref(false)
const selectedChannel = ref<any>(null)
const transformedData = ref<any>(null)
const transformError = ref('')

onMounted(async () => {
  try {
    await Promise.all([
      channelStore.fetchChannels(),
      statsStore.fetchStats()
    ])

    // Update stats with real data
    stats.value = [
      { label: 'Total Transformations', value: statsStore.stats.totalTransformations.toString(), icon: 'pi-sync' },
      { label: 'This Month', value: statsStore.stats.transformationsThisMonth.toString(), icon: 'pi-calendar' },
      { label: 'Success Rate', value: `${statsStore.stats.successRate}%`, icon: 'pi-check-circle' },
      { label: 'Processing', value: statsStore.stats.processing.toString(), icon: 'pi-spinner' }
    ]
  } finally {
    loading.value = false
  }
})

const navigateToTransform = () => {
  transformStore.reset()
  router.push('/transform')
}

const useChannel = (channel: any) => {
  selectedChannel.value = channel
  showUploadDialog.value = true
  transformError.value = ''
  transformedData.value = null
}

const handleFileUpload = async (file: File) => {
  try {
    const result = await channelStore.transformWithChannel(selectedChannel.value.id, file)
    transformedData.value = result.data
    showUploadDialog.value = false
    showResultDialog.value = true
    
    // Refresh stats after successful transformation
    await statsStore.fetchStats()
  } catch (error: any) {
    transformError.value = error.message || 'Error during transformation'
    showUploadDialog.value = false
    showResultDialog.value = true
  }
}

const downloadResult = () => {
  if (!transformedData.value) return

  const blob = new Blob([JSON.stringify(transformedData.value, null, 2)], { type: 'application/json' })
  const url = window.URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `transformed_${selectedChannel.value.name}.json`
  a.click()
  window.URL.revokeObjectURL(url)
}

const closeResultDialog = () => {
  showResultDialog.value = false
  transformedData.value = null
  transformError.value = ''
}

const logout = () => {
  authStore.logout()
  router.push('/login')
}
</script>