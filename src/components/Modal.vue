<script setup>
import { ref } from 'vue'
import { supabase } from '../lib/supabase'

defineProps({
  isOpen: Boolean,
})

const emit = defineEmits(['submit'])

const username = ref('')
const avatarPic = ref('')
const errorMessage = ref('')
const loading = ref(false)

const submit = async () => {
  if (loading.value) return

  errorMessage.value = ''

  if (!username.value.trim()) {
    errorMessage.value = 'Введите имя пользователя'
    return
  }

  loading.value = true

  try {
    const { data: existingUser, error } = await supabase
      .from('voters')
      .select('*')
      .eq('username', username.value.trim())
      .maybeSingle()

    if (error) throw error

    let user

    // Пользователь существует
    if (existingUser) {
      user = existingUser

      // Обновляем аватар только если введён новый
      if (
        avatarPic.value.trim() &&
        avatarPic.value.trim() !== existingUser.avatar_pic
      ) {
        const { data: updatedUser, error: updateError } = await supabase
          .from('voters')
          .update({
            avatar_pic: avatarPic.value.trim(),
          })
          .eq('id', existingUser.id)
          .select()
          .single()

        if (updateError) throw updateError

        user = updatedUser
      }
    } else {
      // Создаём нового пользователя
      const { data: newUser, error: insertError } = await supabase
        .from('voters')
        .insert({
          username: username.value.trim(),
          avatar_pic: avatarPic.value.trim() || null,
        })
        .select()
        .single()

      if (insertError) throw insertError

      user = newUser
    }

    // Сохраняем в localStorage
    localStorage.setItem('voter_id', String(user.id))
    localStorage.setItem('username', user.username)
    localStorage.setItem('avatar_pic', user.avatar_pic || '')
    localStorage.setItem('is_new', 'true')

    emit('submit', user)
  } catch (error) {
    console.error(error)
    errorMessage.value =
      error?.message || 'Произошла ошибка. Попробуйте ещё раз.'
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div
    v-if="isOpen"
    class="modal-overlay"
  >
    <div class="modal">
      <h2>Welcome 👋</h2>

      <p class="description">
        Please tell us who you are before voting.
      </p>

      <input
        v-model="username"
        type="text"
        placeholder="Username"
      />

      <input
        v-model="avatarPic"
        type="url"
        placeholder="Avatar URL"
      />

      <div
        v-if="avatarPic"
        class="avatar-preview"
      >
        <img
          :src="avatarPic"
          alt="Avatar"
        >
      </div>
<p
  v-if="errorMessage"
  class="error"
>
  {{ errorMessage }}
</p>
      <button
  :disabled="loading"
  @click="submit"
>
  {{ loading ? 'Loading...' : 'Continue' }}
</button>
    </div>
  </div>
</template>

<style scoped>
button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.error {
  color: #ef4444;
  font-size: 14px;
}

.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,.45);
  backdrop-filter: blur(8px);

  display: flex;
  align-items: center;
  justify-content: center;

  z-index: 999;
}

.modal {
  width: 420px;
  max-width: calc(100vw - 32px);

  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 16px;

  padding: 24px;
  box-shadow: var(--shadow);

  display: flex;
  flex-direction: column;
  gap: 16px;
}

.description {
  color: var(--text);
}

input {
  height: 48px;

  background: var(--bg);
  color: var(--text-h);

  border: 1px solid var(--border);
  border-radius: 10px;

  padding: 0 14px;
  font: inherit;
}

input:focus {
  outline: none;
  border-color: var(--accent);
}

button {
  height: 48px;

  background: var(--accent);
  color: white;

  border: none;
  border-radius: 10px;

  cursor: pointer;
  font: inherit;
}

.avatar-preview {
  display: flex;
  justify-content: center;
}

.avatar-preview img {
  width: 80px;
  height: 80px;

  border-radius: 50%;
  object-fit: cover;

  border: 2px solid var(--border);
}
</style>