<template>
  <div class="photos-list">
    <PhotoSummary
      v-for="photo in photos"
      :key="photo._id"
      :photo="photo"
      @vote="handleVote"
    />
  </div>
</template>

<script setup>
import { defineProps, defineEmits } from 'vue'
import PhotoSummary from '../layout/PhotoSummary.vue'

defineProps({
  photos: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['vote'])

const handleVote = (photoId) => {
  if (photoId) {
    console.log('PhotosList received vote event for photo:', photoId)
    emit('vote', photoId)
  } else {
    console.error('Received undefined photoId in PhotosList')
  }
}
</script>

<style scoped>
.photos-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 1rem;
}
</style>
