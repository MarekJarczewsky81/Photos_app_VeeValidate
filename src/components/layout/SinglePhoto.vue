<template>
  <Dialog
    header="Photo Details"
    :visible="true"
    :modal="true"
    :closable="true"
    :dismissableMask="true"
    @update:visible="closeDialog"
    :style="{ width: '90%', maxWidth: '800px' }"
  >
    <div v-if="photosRequest.pending" class="loader-overlay">
      <ProgressSpinner />
    </div>
    <div v-else class="single-photo-content">
      <ImageItem :src="photo.src" :alt="photo.title" class="single-photo__image" />
      <div class="photo-details">
        <h3>{{ photo.title }}</h3>
        <p class="author">by {{ photo.author }}</p>
        <Tag :value="photo.category" class="p-mb-2" />
        <p class="description">{{ photo.description }}</p>
        <div class="votes">
          <span>{{ photo.votes }}</span>
          <Button icon="pi pi-star" class="p-button-text p-button-rounded" />
        </div>
      </div>
    </div>

    <template #footer>
      <Button label="Close" icon="pi pi-times" @click="closeDialog" class="p-button-text" />
    </template>
  </Dialog>
</template>

<script>
import { mapState, mapActions } from 'vuex'
import Dialog from 'primevue/dialog'
import Button from 'primevue/button'
import ProgressSpinner from 'primevue/progressspinner'
import ImageItem from '@/components/layout/ImageItem.vue'
import Tag from 'primevue/tag'

export default {
  name: 'SinglePhoto',
  components: { Dialog, Button, ProgressSpinner, ImageItem, Tag },
  computed: {
    ...mapState('photos', {
      photo: state => state.singlePhoto,
      photosRequest: state => state.photosRequest
    }),
    photoId () {
      return this.$route.params.photoId
    }
  },
  watch: {
    photoId: 'fetchPhoto'
  },
  mounted () {
    this.fetchPhoto()
  },
  methods: {
    ...mapActions('photos', ['fetchSinglePhoto']),
    fetchPhoto () {
      this.fetchSinglePhoto(this.photoId)
    },
    closeDialog () {
      this.$router.push('/')
    }
  }
}
</script>

<style lang="scss" scoped>
::v-deep .p-dialog {
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

::v-deep .p-dialog-header {
  padding: 0.5rem 1rem;
  border-bottom: 1px solid #eee;
}

.single-photo-content {
  display: flex;
  flex-direction: column;
}

.single-photo__image {
  width: 100%;
  height: auto;
  object-fit: cover;
  max-height: 500px;
}

.photo-details {
  padding: 1rem;
}

h3 {
  margin: 0 0 0.5rem;
  font-size: 1.5rem;
}

.author {
  color: #666;
  margin-bottom: 0.5rem;
}

.description {
  margin: 1rem 0;
}

.votes {
  display: flex;
  align-items: center;
  justify-content: flex-end;

  span {
    margin-right: 0.5rem;
    font-weight: bold;
  }
}

::v-deep .p-tag {
  font-size: 0.8rem;
}

.loader-overlay {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 300px;
}
</style>
