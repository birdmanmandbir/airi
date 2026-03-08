<script setup lang="ts">
import { IconStatusItem, RippleGrid } from '@proj-airi/stage-ui/components'
import { useAnalytics } from '@proj-airi/stage-ui/composables'
import { useProvidersStore } from '@proj-airi/stage-ui/stores/providers'
import { SelectTab } from '@proj-airi/ui'
import { storeToRefs } from 'pinia'
import { computed, ref } from 'vue'

const providersStore = useProvidersStore()
const { trackProviderClick } = useAnalytics()

const {
  allChatProvidersMetadata,
  allAudioSpeechProvidersMetadata,
  allAudioTranscriptionProvidersMetadata,
} = storeToRefs(providersStore)

const providerBlocksConfig = [
  {
    id: 'chat',
    icon: 'i-solar:chat-square-like-bold-duotone',
    title: 'Chat',
    description: 'Text generation model providers. e.g. OpenRouter, OpenAI, Ollama.',
    providersRef: allChatProvidersMetadata,
  },
  {
    id: 'speech',
    icon: 'i-solar:user-speak-rounded-bold-duotone',
    title: 'Speech',
    description: 'Speech (text-to-speech) model providers. e.g. ElevenLabs, Azure Speech.',
    providersRef: allAudioSpeechProvidersMetadata,
  },
  {
    id: 'transcription',
    icon: 'i-solar:microphone-3-bold-duotone',
    title: 'Transcription',
    description: 'Transcription (speech-to-text) model providers. e.g. Whisper.cpp, OpenAI, Azure Speech',
    providersRef: allAudioTranscriptionProvidersMetadata,
  },
]

const activeTab = ref<'chat' | 'speech' | 'transcription'>('chat')

const tabOptions = [
  { label: 'Chat', value: 'chat', icon: 'i-solar:chat-square-like-bold-duotone' },
  { label: 'Speech', value: 'speech', icon: 'i-solar:user-speak-rounded-bold-duotone' },
  { label: 'Transcription', value: 'transcription', icon: 'i-solar:microphone-3-bold-duotone' },
]

const activeBlock = computed(() => {
  const block = providerBlocksConfig.find(b => b.id === activeTab.value)
  if (!block) return null
  return {
    id: block.id,
    icon: block.icon,
    title: block.title,
    description: block.description,
    providers: block.providersRef.value.map((provider, i) => ({
      ...provider,
      renderIndex: i,
    })),
  }
})
</script>

<template>
  <div mb-6 flex flex-col gap-5>
    <div bg="primary-500/10 dark:primary-800/25" rounded-lg p-4>
      <div mb-2 text-xl font-normal text="primary-800 dark:primary-100">
        {{ $t('settings.pages.providers.helpinfo.title') }}
      </div>
      <div text="primary-700 dark:primary-300">
        <i18n-t keypath="settings.pages.providers.helpinfo.description">
          <template #chat>
            <div bg="primary-500/10 dark:primary-800/25" inline-flex items-center gap-1 rounded-lg px-2 py-0.5 translate-y="[0.25lh]">
              <div i-solar:chat-square-like-bold-duotone />
              <strong class="font-normal">Chat</strong>
            </div>
          </template>
        </i18n-t>
      </div>
    </div>

    <SelectTab
      v-model="activeTab"
      :options="tabOptions"
    />

    <RippleGrid
      v-if="activeBlock"
      :sections="[activeBlock]"
      :get-items="block => block.providers"
      :columns="{ default: 1, sm: 2, xl: 3 }"
    >
      <template #item="{ item: provider }">
        <IconStatusItem
          :title="provider.localizedName || 'Unknown'"
          :description="provider.localizedDescription"
          :icon="provider.icon"
          :icon-color="provider.iconColor"
          :icon-image="provider.iconImage"
          :to="`/settings/providers/${provider.category}/${provider.id}`"
          :configured="provider.configured"
          @click="trackProviderClick(provider.id, provider.category)"
        />
      </template>
    </RippleGrid>
  </div>
  <div
    v-motion
    text="neutral-500/5 dark:neutral-600/20" pointer-events-none
    fixed top="[calc(100dvh-15rem)]" bottom-0 right--5 z--1
    :initial="{ scale: 0.9, opacity: 0, y: 20 }"
    :enter="{ scale: 1, opacity: 1, y: 0 }"
    :duration="500"
    size-60
    flex items-center justify-center
  >
    <div text="60" i-solar:box-minimalistic-bold-duotone />
  </div>
</template>

<route lang="yaml">
meta:
  layout: settings
  titleKey: settings.pages.providers.title
  subtitleKey: settings.title
  descriptionKey: settings.pages.providers.description
  icon: i-solar:box-minimalistic-bold-duotone
  settingsEntry: true
  order: 6
  stageTransition:
    name: slide
    pageSpecificAvailable: true
</route>
