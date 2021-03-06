<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useI18n } from 'vue-i18n'
import { defineProps } from 'vue'

const props = defineProps({
  guild: {
    type: String,
    required: true,
  },
})

const router = useRouter()
const { t } = useI18n()

const guildIcon = ref('')
const guildName = ref('')
const error = ref('')

interface Score {
  username?: string
  discriminator?: string
  avatar?: string
  displayColor?: string
  status?: 'online' | 'idle' | 'dnd'
  user?: string
  guild?: string
  exp?: number
  level?: number
  total_exp?: number
  cooldown_till?: Date
  percentage?: number
}

const scores = ref<Score[]>([])

function compare(a: Score, b: Score) {
  if (a.total_exp! > b.total_exp!)
    return -1
  if (a.total_exp! < b.total_exp!)
    return 1

  return 0
}

// fetch(`http://127.0.0.1:7777/v1/${props.guild}`)
fetch(`https://gideon-api.koen02.nl/v1/${props.guild}`)
  .then(async(response) => {
    if (response.ok) {
      const data = await response.json()
      guildIcon.value = data.guildInfo.icon
      guildName.value = data.guildInfo.name
      for (const score of data.scores)
        score.percentage = Math.round(((score.exp || 0) / Math.floor(5 * Math.pow(score.level || 0, 2) + 50 * (score.level || 0) + 100)) * 100)

      const scoresSorted = data.scores
      scoresSorted.sort(compare)
      scores.value = scoresSorted
    }
    else {
      error.value = `Could not find any guild with ID: ${props.guild}`
    }
  })
  .catch(() => error.value = 'Error while fetching data')
</script>

<template>
  <div>
    <img
      v-if="guildIcon"
      :src="guildIcon"
      alt="Gideon Avatar"
      class="inline rounded-full h-20 w-20 mb-3"
    />
    <carbon-user-avatar v-else class="h-20 w-20 mb-3" />
    <p class="font-bold text-2xl font-custom">{{ guildName }}</p>

    <p v-if="error" class="text-xl font-custom my-15">{{ error }}</p>

    <div v-if="scores.length > 0" class="my-15 flex flex-col gap-5 items-center">
      <div
        v-for="(score, index) in scores"
        :key="score.user"
        class="max-w-3xl border dark:border-gray-700 shadow-lg hover:shadow-xl transition duration-500 rounded-lg w-full "
      >
        <div class="flex flex-col sm:flex-row">
          <div class="flex items-center justify-center">
            <span class="sm:ml-10 font-bold text-xl sm:text-2xl font-custom">#{{ index + 1 }}</span>
            <img
              :src="score.avatar"
              :alt="`Avatar of ${score.username}#${score.discriminator}`"
              class="object-contain h-15 my-3 sm:my-6 mx-3 rounded-full"
            />
            <div>
              <span class="text-xl sm:text-2xl font-custom">{{ score.username }}</span>
              <span class="sm:text-lg text-gray-400 font-custom">#{{ score.discriminator }}</span>
            </div>
          </div>
          <div class="flex flex-row items-center flex-grow">
            <div class="flex items-center flex-col mx-5 sm:mx-7 flex-grow">
              <span class="font-custom text-sm sm:text-lg">
                {{ score.exp || 0 }} / {{ Math.floor(5 * Math.pow(score.level || 0, 2) + 50 * (score.level || 0) + 100) }}
                XP [{{ score.total_exp }} Total]
              </span>
              <div
                class="w-full h-6 bg-gray-100 dark:bg-darkbg rounded-lg border border-gray-300 dark:border-gray-700 flex-grow mb-3"
              >
                <div
                  class="h-full text-center text-xs dark:text-white bg-blue-400 rounded-lg border-2 border-gray-100 dark:border-darkbg"
                  :style="`width: ${score.percentage}%`"
                >
                  <span>{{ score.percentage }}%</span>
                </div>
              </div>
            </div>
            <span class="text-xl sm:text-2xl mr-5 sm:mr-10 font-custom flex-shrink-0">Level {{ score.level }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
