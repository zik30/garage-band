<script setup>
import { ref, onMounted } from 'vue'
  import UserProfileModal from './Modal.vue'
import { supabase } from '../lib/supabase'
import SongsSlider from './SongsSlider.vue'


const showModal = ref(false)

onMounted(() => {
  const username = localStorage.getItem('username')
  const avatar = localStorage.getItem('avatar_pic')

  if (!username || !avatar) {
    showModal.value = true
  }
})

  const saveProfile = async ({ username, avatar_pic }) => {
    const { data, error } = await supabase
      .from('voters')
      .insert({
        username,
        avatar_pic,
      })
      .select()
      .single()

    if (error) {
      console.error(error)
      return
    }

    localStorage.setItem('username', data.username)
    localStorage.setItem('avatar_pic', data.avatar_pic)
    localStorage.setItem('voter_id', data.id)

    showModal.value = false
  }
</script>

<template>
  <section id="center">
    <UserProfileModal
      :is-open="showModal"
      @submit="saveProfile"
    />

    <SongsSlider/>
  </section>


  <section id="spacer"></section>
</template>
