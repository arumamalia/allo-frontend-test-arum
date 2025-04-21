<template>
  <h1 class="text-center">
    ROCKET LIST
  </h1>
  <div class="container mb-3">
    <div class="d-flex justify-content-between">
      <a
        href="/create"
        class="col-md-2 btn btn-primary"
      >
        Add
      </a>
      <select
        class="form-select w-25"
        aria-label="Default select example"
        @change="onChange($event)"
      >
        <option
          value=""
          selected
        >
          Filter by All Engines
        </option>
        <option value="merlin">
          Merlin
        </option>
        <option value="raptor">
          Raptor
        </option>
      </select>
    </div>
  </div>
  <div
    v-if="!loading"
    class="container"
  >
    <div class="row row-cols-1 row-cols-md-4 g-4">
      <div
        v-for="(list, index) in rocketList"
        :key="index"
        class="col"
      >
        <div
          class="card h-100"
          style="width: 18rem;"
        >
          <img
            :src="list.flickr_images[0]"
            class="card-img-top img-fluid"
            alt="Rocket image"
            style="height: 200px; object-fit: cover;"
          >
          <div class="card-body d-flex flex-column">
            <h5 class="card-title">
              {{ list.name }}
            </h5>
            <p class="card-text flex-grow-1">
              {{ list.description }}
            </p>
            <a
              :href="'/detail/'+list.id"
              class="btn btn-primary mt-auto"
            >
              Detail
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div v-else>
    <Loader />
  </div>
</template>

<script setup lang="ts">
  import {onMounted, ref} from 'vue'
  import axios from 'axios'
  import * as bootstrap from 'bootstrap'

  const rocketList = ref()
  const loading = ref(false)

  onMounted(async () => {
    fetchRocket()

    const modal = document.getElementById('errorModal');

    modal.addEventListener('hidden.bs.modal', () => {
      fetchRocket()
    });
  })

  const getFromLocalStorage = () => {
    const existingRockets = JSON.parse(localStorage.getItem('spacex-rockets') || '[]');

    return existingRockets

  }

  const fetchRocket = async (engineType = '') => {
    const dataLocalStorage = getFromLocalStorage()

    try {
      loading.value = true
      const payload = {
        "query": {},
        "options": {}
      }

      if (engineType) {
        payload.query = {
          "engines.type": engineType
        }
      }

      const response = await axios.post('https://api.spacexdata.com/v4/rockets/query', payload);
      rocketList.value = response.data.docs
      if (!engineType) {
        rocketList.value = [...dataLocalStorage, ...response.data.docs]
      }
    } catch (error) {
      console.error('Error fetching rocket list:', error)
      showModal()
    } finally {
      loading.value = false
    }
  }

  const showModal = () => {
    const modalInstance = new bootstrap.Modal(document.querySelector('#errorModal'));
    if (modalInstance) modalInstance.show();
  };

  const onChange = (event) => {
    fetchRocket(event.target.value)
  }
</script>
