<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { io } from 'socket.io-client'

const { t } = useI18n()

const socket = io('127.0.0.1:7777')
socket.on('connect', () => {})

const guild = ref('')

const router = useRouter()
const go = () => {
  if (guild.value)
    router.push(`/dash/${encodeURIComponent(guild.value)}`)
}

</script>

<template>
  <div class="">
    <img src="/avatar.webp" alt="Gideon Avatar" class="inline rounded-full h-20 w-20 mb-3">
    <p>
      <a rel="noreferrer" href="https://github.com/antfu/vitesse" target="_blank">
        {{ t('intro.name') }}
      </a>
    </p>
    <p>
      <em class="text-sm opacity-75">{{ t('intro.desc') }}</em>
    </p>

    <div class="py-4"></div>

    <input
      id="input"
      v-model="guild"
      :placeholder="t('intro.whats-your-guild')"
      :aria-label="t('intro.whats-your-guild')"
      type="text"
      autocomplete="false"
      class="px-4 py-2 text-sm text-center bg-transparent border border-gray-200 rounded outline-none active:outline-none dark:border-gray-700"
      style="width: 250px"
      @keydown.enter="go"
    >
    <label class="hidden" for="input">{{ t('intro.whats-your-guild') }}</label>

    <div>
      <button
        class="m-3 text-sm btn"
        :disabled="!guild"
        @click="go"
      >
        {{ t('button.go') }}
      </button>
    </div>
  </div>
</template>
