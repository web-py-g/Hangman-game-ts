<script setup lang="ts">
import GameHeader from '@/components/GameHeader.vue'
import GameFigure from '@/components/GameFigure.vue'
import GameWrongLetters from '@/components/GameWrongLetters.vue'
import GameWord from '@/components/GameWord.vue'
import GamePopup from '@/components/GamePopup.vue'
import GameNotification from '@/components/GameNotification.vue'
import { computed, onMounted, ref, watch } from 'vue'
import axios from 'axios'

const word = ref('')
const getRandomName = async () => {
  try {
    let { data } = await axios<{ FirstName: string }>(
      'https://api.randomdatatools.ru/?unescaped=false&params=FirstName'
    )

    word.value = data.FirstName.toLowerCase()
  } catch (error) {
    console.error(error)
    word.value = ''
  }
}

onMounted(() => {
  getRandomName()
})

const letters = ref<string[]>([])
const correctLetters = computed(() => letters.value.filter((x) => word.value.includes(x)))
const wrongLetters = computed(() => letters.value.filter((x) => !word.value.includes(x)))
const notification = ref<InstanceType<typeof GameNotification> | null>(null)
const popup = ref<InstanceType<typeof GamePopup> | null>(null)

const isLose = computed(() => wrongLetters.value.length === 6)
const isWin = computed(() => [...word.value].every((x) => correctLetters.value.includes(x)))

watch(wrongLetters, () => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})

watch(correctLetters, () => {
  if (isWin.value) {
    popup.value?.open('win')
  }
})

window.addEventListener('keydown', ({ key }) => {
  if (isWin.value || isLose.value) {
    return
  }

  if (letters.value.includes(key)) {
    notification.value?.open()
    setTimeout(() => {
      notification.value?.close()
    }, 2000)
    return
  }

  if (/[а-яА-ЯёЁ]/.test(key)) {
    letters.value.push(key.toLowerCase())
  }
})

const restart = async () => {
  await getRandomName()
  letters.value = []
  popup.value?.close()
}
</script>

<template>
  <GameHeader />
  <div class="game-container">
    <GameFigure :wrong-letters-count="wrongLetters.length" />

    <GameWrongLetters :wrong-letters="wrongLetters" />

    <GameWord :correct-letters="correctLetters" :word="word" />
  </div>

  <GamePopup @restart="restart" ref="popup" :word="word" />

  <GameNotification ref="notification" />
</template>
