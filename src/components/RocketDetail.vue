<template>
  <div>
    <h1 class="text-center">
      ROCKET DETAIL
    </h1>
    <div v-if="!loading">
      <div class="container">
        <a
          href="/"
          type="button"
          class="btn btn-dark mb-3"
        >
          Back
        </a>
        <form @submit.prevent="handleSubmit">
          <div class="row mb-3">
            <div class="col">
              <!-- Display preview if available, otherwise show the original image -->
              <img
                v-if="imagePreview"
                :src="imagePreview"
                alt="Rocket image"
                class="img-fluid mb-3"
              >
              <img
                v-else
                :src="formData.flickr_images"
                alt="Rocket image"
                class="img-fluid mb-3"
              >
            </div>

            <div class="row col">
              <div
                v-if="path !== 'detail'"
                class="col-md-12 mb-3"
              >
                <label
                  for="imageUrl"
                  class="form-label"
                >
                  Image URL
                </label>
                <input
                  id="imageUrl"
                  type="file"
                  accept="image/*"
                  class="form-control"
                  required
                  @change="handleImageUpload"
                >
              </div>

              <div class="col-md-12 mb-3">
                <label
                  for="name"
                  class="form-label"
                >
                  Rocket Name
                </label>
                <input
                  id="name"
                  :value="path === 'detail' ? formData.name : null"
                  type="text"
                  class="form-control"
                  :readonly="path === 'detail'"
                  required
                  @input="path !== 'detail' ? formData.name = $event.target.value : null"
                >
              </div>

              <div class="col-md-12 mb-3">
                <label
                  for="description"
                  class="form-label"
                >
                  Rocket Description
                </label>
                <textarea
                  id="description"
                  :value="path === 'detail' ? formData.description : null"
                  class="form-control"
                  rows="5"
                  :readonly="path === 'detail'"
                  required
                  @input="path !== 'detail' ? formData.description = $event.target.value : null"
                />
              </div>

              <div class="col-md-12 mb-3">
                <label
                  for="const"
                  class="form-label"
                >
                  Cost per launch
                </label>
                <input
                  id="cost"
                  :value="path === 'detail' ? formData.cost_per_launch : null"
                  type="number"
                  class="form-control"
                  :readonly="path === 'detail'"
                  required
                  @input="path !== 'detail' ? formData.cost_per_launch = $event.target.value : null"
                >
              </div>

              <div class="col-md-12 mb-3">
                <label
                  for="country"
                  class="form-label"
                >
                  Country
                </label>
                <input
                  id="country"
                  :value="path === 'detail' ? formData.country : null"
                  type="text"
                  class="form-control"
                  :readonly="path === 'detail'"
                  required
                  @input="path !== 'detail' ? formData.country = $event.target.value : null"
                >
              </div>

              <div class="col-md-12 mb-3">
                <label
                  for="first_flight"
                  class="form-label"
                >
                  First flight
                </label>
                <input
                  id="first_flight"
                  :value="path === 'detail' ? formData.first_flight : null"
                  type="date"
                  class="form-control"
                  :readonly="path === 'detail'"
                  required
                  @input="path !== 'detail' ? formData.first_flight = $event.target.value : null"
                >
              </div>

              <div
                v-if="path !== 'detail'"
                class="col-md-12"
              >
                <button
                  type="submit"
                  class="btn btn-primary w-100 mt-3"
                >
                  Save
                </button>
              </div>
            </div>
          </div>
        </form>
      </div>
    </div>
    <div v-else>
      <Loader />
    </div>
  </div>
</template>

<script setup lang="ts">
  import { onMounted, ref } from 'vue'
  import { useRoute, useRouter } from 'vue-router'
  import axios from 'axios'

  const route = useRoute()
  const router = useRouter();
  const path = ref(route.path.split('/')[1])
  const loading = ref(true)
  const imagePreview = ref('')
  const formData = ref({
    flickr_images: '',
    name: '',
    description: '',
    cost_per_launch: '',
    country: '',
    first_flight: ''
  })

  const handleImageUpload = (event) => {
    const file = event.target.files[0]
    if (file) {
      const reader = new FileReader()
      reader.onload = (e) => {
        imagePreview.value = e.target.result
        formData.value.flickr_images = e.target.result
      }
      reader.readAsDataURL(file)
    }
  }

  const generateId = (): string => {
    return 'rocket-' + Date.now().toString(36) + Math.random().toString(36).substring(2);
  };

  const saveRocketToLocalStorage = (rocketData: any) => {
    const existingRockets = JSON.parse(localStorage.getItem('spacex-rockets') || '[]');

    const existingIndex = existingRockets.findIndex((r: any) => r.id === rocketData.id);

    if (existingIndex >= 0) {
      existingRockets[existingIndex] = rocketData;
    } else {
      existingRockets.push(rocketData);
    }

    localStorage.setItem('spacex-rockets', JSON.stringify(existingRockets));
  };

  onMounted(async () => {
    if (path.value === 'detail'){
      const id = route.params.id.toString()
      try {
        const response = await axios.get(`https://api.spacexdata.com/v4/rockets/${id}`)
        formData.value = {
          ...response.data,
          flickr_images: response.data.flickr_images[0]
        }

      } catch (error) {
        console.error('Error fetching rocket details:', error)
      } finally {
        loading.value = false
      }
    } else {
      loading.value = false
    }
  })

  const handleSubmit = async () => {
    const rocketData = {
      ...formData.value,
      id: path.value === 'edit' ? route.params.id : generateId(),
      flickr_images: [
        formData.value.flickr_images
      ]
    }
    saveRocketToLocalStorage(rocketData);

    router.push('/')
    console.log('Form submitted', formData.value)
  }
</script>
