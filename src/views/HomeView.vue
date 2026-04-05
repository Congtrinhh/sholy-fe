<script setup lang="ts">
import { computed, ref } from 'vue'

interface Url {
  long_url: string
  short_url?: string
  expire_in_number?: number
  expire?: Date
}

const request = ref<Url>({
  long_url: 'https://expressjs.com/en/starter/generator.html',
})

const result = ref<Url>({ long_url: 'abc', short_url: 'alias', expire: new Date() })

async function submitUrl() {
  isVisibleResult.value = false

  const baseUrl = import.meta.env.VITE_API_BASE_URL
  const apiUrl = `${baseUrl}/urls`
  const response = await fetch(apiUrl, {
    method: 'POST',
    body: JSON.stringify(request.value),
    headers: {
      'Content-Type': 'application/json', // Tell the server we are sending JSON
    },
  })

  if (!response.ok) {
    throw new Error(`HTTP error! Status: ${response.status}`) // Check if request was successful
  }

  const url = await response.json()
  result.value = url
  isVisibleResult.value = true
}

const resultShortUrl = computed(() => {
  const serverBaseUrl = import.meta.env.VITE_API_BASE_URL
  return `${serverBaseUrl}/urls/${result.value?.short_url}`
})

const isVisibleResult = ref(false)
</script>

<template>
  <main>
    <section id="main">
      <div class="form-wrapper wrapper">
        <div class="title">Create short link with Sho.ly</div>

        <div class="form-parent">
          <div class="form-group">
            <div class="label">Long url</div>
            <input
              type="text"
              name="long-url"
              id="longUrl"
              v-model="request.long_url"
              placeholder="https://portal.azure.com/#allservices/category/All"
            />
          </div>
          <div class="form-group">
            <div class="label">Short url (Optional)</div>
            <input
              type="text"
              name="short-url"
              id="shortUrl"
              v-model="request.short_url"
              placeholder="https://sho.ly/abc => abc"
            />
          </div>
          <div class="form-group">
            <div class="label">Expiry date from now (Optional)</div>
            <input
              type="text"
              name="expire"
              id="expire"
              placeholder="7"
              v-model="request.expire_in_number"
            />
          </div>
          <div class="form-group">
            <button @click="submitUrl">Get link</button>
          </div>
        </div>
      </div>

      <div class="result-wrapper wrapper" v-if="isVisibleResult">
        <div class="short-link-result">
          <span>Created successfully:</span>
          <div class="value">{{ resultShortUrl }}</div>
          <div class="copy-button"><i class="fa-regular fa-clone"></i></div>
        </div>
        <div class="expiry-date">Expiry date: {{ result.expire }}</div>
      </div>
    </section>
  </main>
</template>

<style>
main {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100vw;
  height: 100vh;
  background: #f5e9d8;
}

#main {
  display: flex;
  flex-direction: column;
  width: 60%;
  max-width: 600px;
  gap: 40px;
}

.form-wrapper {
}

.title {
  font-size: 1.6rem;
  font-weight: bold;
  text-align: center;
}

.form-parent {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

input {
  padding: 8px 12px;
  border-radius: 4px;
  /* border: 2px dashed #333; */
  border: 1px solid #333;
}

button {
  background-color: #e76f2e;
  color: #fff;
  padding: 8px;
  border-radius: 6px;
  cursor: pointer;
}

.wrapper {
  padding: 24px;
  border: 1px solid;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  background: white;
}

.copy-button {
  cursor: pointer;
}

.short-link-result {
  display: flex;
  align-items: center;
  gap: 8px;
}
</style>
