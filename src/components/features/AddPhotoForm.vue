<template>
  <Form
    class="add-photo-form"
    @submit="handleSubmit"
    v-slot="{ errors }">
    <h2>Add photo</h2>
    <Message v-if="isSuccess" severity="success" :closable="false">Success! Your photo has been submitted</Message>
    <Message v-if="isError" severity="error" :closable="false">Oops… something went wrong…</Message>
    <div class="form-content">
      <div class="form-fields">
        <!-- title -->
        <Field
          class="p-field"
          name="title"
          v-slot="{ field }"
          rules="required|min:10|max:60"
          >
          <label>Title</label>
          <InputText
            v-bind="field"
            type="text"
            v-model="form.title"
          />
          <span class="error-text">{{ errors.title }}</span>
        </Field>

        <!-- author -->
        <Field
          class="p-field"
          name="author"
          v-slot="{ field }"
          rules="required|min:3|max:30">
          <label>Author</label>
          <InputText
            v-model="form.author"
            v-bind="field"
            type="text"
          />
          <span class="error-text">{{ errors.author }}</span>
        </Field>

        <!-- category -->
        <Field
          class="p-field"
          name="category"
          v-slot="{ field }"
          rules="required">
          <label>Category</label>
          <Dropdown
            v-model="form.category"
            v-bind="field"
            type="text"
            :options="categories"
            optionLabel="name"
            placeholder="Select a Category"
          />
        </Field>

        <!-- description -->
        <Field
          class="p-field"
          name="description"
          v-slot="{ field }"
          rules="required|max:100">
          <label>Description</label>
          <Textarea
            v-model="form.description"
            rows="5"
            v-bind="field"
            type="text"
          />
          <span class="error-text">{{ errors.description }}</span>
        </Field>

        <Button
          class="submit-button"
          type="submit"
          label="Add"
          icon="pi pi-plus"
        />
      </div>

      <Field class="p-col" v-slot="{ field }" rules="required|ext:png,jpg" name="image">
        <span class="error-text">{{ errors.image }}</span>
        <ImageUpload
          v-bind="field"
          @choose="onFileChosen" />
      </Field>
    </div>
  </Form>
</template>

<script>
import { mapState } from 'vuex'
import ImageUpload from '@/components/shared/ImageUpload.vue'
import InputText from 'primevue/inputtext'
import Dropdown from 'primevue/dropdown'
import Textarea from 'primevue/textarea'
import Button from 'primevue/button'
import Message from 'primevue/message'
import axios from 'axios'
import { apiUrl } from '@/config'
import { Form, Field, defineRule } from 'vee-validate'
// Import reguł walidacji
import { required, min, max, ext } from '@vee-validate/rules'

export default {
  name: 'AddPhotoForm',
  components: {
    ImageUpload,
    InputText,
    Dropdown,
    Textarea,
    Button,
    Message,
    Form,
    Field
  },
  setup () {
    // Rejestracja reguł
    defineRule('required', (value) => required(value) || 'This field is so so required...')
    defineRule('min', (value, params) => min(value, params) || `It should be longer than ${params}`)
    defineRule('max', (value, params) => max(value, params) || `It should be shorten than ${params}`)
    defineRule('ext', (value, params) => ext(value, params) || `You should use one of these extensions: ${params}`)
  },
  computed: {
    ...mapState({
      categories: state => state.categories.categories
    })
  },
  data () {
    return {
      form: {
        title: '',
        author: '',
        description: '',
        category: null,
        file: null
      },
      isSuccess: false,
      isError: false
    }
  },
  mounted () {
    this.$store.dispatch('categories/fetchCategories')
  },
  methods: {
    // Validator function
    isRequired (value) {
      return value ? true : 'This field is required'
    },
    async handleSubmit (e) {
      if (e && e.preventDefault) {
        e.preventDefault()
      }

      this.isSuccess = false
      this.isError = false

      if (!this.form.category) {
        console.error('Category is not selected')
        this.isError = true
        return
      }

      try {
        const formData = new FormData()
        formData.append('title', this.form.title)
        formData.append('author', this.form.author)
        formData.append('description', this.form.description)
        formData.append('category', this.form.category.name)
        if (this.form.file) {
          formData.append('file', this.form.file)
        }
        await axios.post(`${apiUrl}/photos`, formData, { 'Content-Type': 'multipart/form-data' })
        this.isSuccess = true
        const category = this.form.category.name
        this.resetForm()
        this.$router.push(`/photos/${category}`)
      } catch (error) {
        console.error('Error submitting form:', error)
        this.isError = true
      }
    },
    onFileChosen (file) {
      this.form.file = file
    },
    resetForm () {
      this.form = {
        title: '',
        author: '',
        description: '',
        category: '',
        file: null
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.add-photo-form {
  max-width: 800px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

h2 {
  font-size: 24px;
  color: #333;
  margin-bottom: 20px;
}

.form-content {
  display: flex;
  gap: 2rem;
}

.form-fields {
  flex: 1;
}

.p-field {
  margin-bottom: 1rem;
}

.p-field label {
  display: block;
  margin-bottom: 0.5rem;
  font-size: 14px;
  color: #666;
}

:deep(.p-inputtext),
:deep(.p-dropdown),
:deep(.p-textarea) {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  font-size: 14px;
}

:deep(.p-dropdown) {
  .p-dropdown-label {
    padding: 8px;
  }
}

:deep(.p-textarea) {
  min-height: 100px;
}

.submit-button {
  background-color: #4CAF50;
  border: none;
  color: white;
  padding: 10px 20px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin-top: 10px;
  cursor: pointer;
  border-radius: 4px;
}

.image-upload-container {
  flex: 1;
  display: flex;
  justify-content: center;
  align-items: flex-start;
}

:deep(.image-upload) {
  border: 2px dashed #ccc;
  border-radius: 4px;
  width: 100%;
  height: 300px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f9f9f9;
}

.error-text {
  color: red;
  font-size: 12px;
  margin-top: 4px;
}
</style>
