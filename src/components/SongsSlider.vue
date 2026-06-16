<script setup>
import { ref, onMounted } from 'vue'
import { supabase } from '../lib/supabase.js'

  const songs = ref([])

  const currentVoterId = Number(localStorage.getItem('voter_id'))
  const currentUsername = localStorage.getItem('username')
  const currentAvatar = localStorage.getItem('avatar_pic')



const loadSongs = async () => {
  const { data, error } = await supabase
    .from('songs')
    .select(`
      *,
      votes!votes_song_fk (
        voter_id,
        voters!votes_voter_fk (
          username,
          avatar_pic
        )
      )
    `)

  if (error) {
    console.error(error)
    return
  }

  songs.value = data || []
}

  const hasVoted = (song) => {

    song.votes?.map((vote) => {
      console.log(vote.voter_id+ " and "+ currentVoterId);
      
    })
  
  return song.votes?.some(
    vote => vote.voter_id === currentVoterId
  )
}

const toggleVote = async (song) => {
  if (!currentVoterId) {
    alert('Profile not found')
    return
  }

  const existingVote = song.votes?.find(
    vote => vote.voter_id === currentVoterId
  )

  if (existingVote) {
    // optimistic remove
    song.votes = song.votes.filter(
      vote => vote.voter_id !== currentVoterId
    )

    const { error } = await supabase
      .from('votes')
      .delete()
      .eq('song_id', song.id)
      .eq('voter_id', currentVoterId)

    if (error) {
      console.error(error)
      await loadSongs()
      return
    }

    return
  }

  if (!song.votes) {
    song.votes = []
  }

  // optimistic add
  song.votes.push({
    voter_id: currentVoterId,
    voters: {
      username: currentUsername,
      avatar_pic: currentAvatar,
    },
  })

  const { error } = await supabase
    .from('votes')
    .insert({
      song_id: song.id,
      voter_id: currentVoterId,
    })

  if (error) {
    console.error(error)
    await loadSongs()
  }
}

onMounted(loadSongs)
</script>

<template>
  <div class="songs-grid">
    <div
      v-for="song in songs"
      :key="song.id"
      class="song-card"
    >
      <img
        :src="song.song_picture"
        :alt="song.song_name"
        class="cover"
      />

      <div class="content">
        <h2>{{ song.song_name }}</h2>

        <p>{{ song.song_artist }}</p>

        <audio
          :src="song.song_url"
          controls
        />

        <button
          class="vote-btn"
          :class="{ voted: hasVoted(song) }"
          @click="toggleVote(song)"
        >
          {{ hasVoted(song) ? ' Remove Vote' : ' Vote' }}
          ({{ song.votes?.length || 0 }})
        </button>

        <div
          v-if="song.votes?.length"
          class="voters"
        >
          <img
            v-for="vote in song.votes"
            :key="vote.voter_id"
            :src="vote.voters?.avatar_pic"
            :alt="vote.voters?.username"
            :title="vote.voters?.username"
            class="avatar"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.songs-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(340px, 1fr));
  gap: 24px;

  max-width: 1200px;
  margin: 0 auto;
  padding: 24px;
}

.song-card {
  background: var(--bg);
  border: 1px solid var(--border);
  border-radius: 16px;
  overflow: hidden;

  box-shadow: var(--shadow);
}

.cover {
  width: 100%;
  height: 280px;
  object-fit: cover;
}

.content {
  padding: 16px;
}

.content h2 {
  margin-bottom: 6px;
}

.content p {
  margin-bottom: 16px;
}

audio {
  width: 100%;
}

.vote-btn {
  width: 100%;
  margin-top: 16px;

  border: none;
  border-radius: 10px;

  padding: 12px;

  cursor: pointer;

  background: var(--accent-bg);
  color: var(--accent);

  font: inherit;
  font-weight: 600;

  transition: 0.2s;
}

.vote-btn:hover {
  transform: translateY(-1px);
}

.vote-btn.voted {
  background: var(--accent);
  color: white;
}

.voters {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;

  margin-top: 16px;
}

.avatar {
  width: 42px;
  height: 42px;

  border-radius: 50%;
  object-fit: cover;

  border: 2px solid var(--border);
}
</style>