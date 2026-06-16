<script setup>
import { ref, onMounted } from 'vue'
import UserProfileModal from './Modal.vue'
import SongsSlider from './SongsSlider.vue'

const showModal = ref(false)

onMounted(() => {
  const username = localStorage.getItem('username')
  const isNew = localStorage.getItem('is_new')

  if (!username || isNew !== 'true') {
    localStorage.removeItem('voter_id')
    localStorage.removeItem('username')
    localStorage.removeItem('avatar_pic')

    showModal.value = true
  }
})

const saveProfile = () => {
  showModal.value = false
}
</script>

<template>
  <section id="center">
    <UserProfileModal
      :is-open="showModal"
      @submit="saveProfile"
    />

    <SongsSlider />
  </section>

  <section id="spacer"></section>
</template>