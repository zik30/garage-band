<script setup lang="ts">
import { ref } from 'vue'

defineProps({
  isOpen: Boolean,
})

const emit = defineEmits(['submit'])

const username = ref('')
const avatarPic = ref('')

const submit = () => {
  if (!username.value.trim()) return

  emit('submit', {
    username: username.value,
    avatar_pic: avatarPic.value,
  })
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

      <button @click="submit">
        Continue
      </button>
    </div>
  </div>
</template>

<style scoped>
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