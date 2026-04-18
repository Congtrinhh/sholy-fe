<script setup lang="ts">
import { computed, ref } from 'vue'

interface Url {
  long_url: string
  short_url?: string
  expire_in_number?: number
  expire?: Date
}

const request = ref<Url>({
  long_url: '',
})

const result = ref<Url>({ long_url: 'abc', short_url: 'alias', expire: new Date() })

async function submitUrl() {
  clearErrorMessage()
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
    // handle error by showing it
    const { error } = await response.json()
    errorMessage.value = error

    throw new Error(`HTTP error! Status: ${response.status}`) // Check if request was successful
  }

  const url = await response.json()
  result.value = url
  isVisibleResult.value = true
}

const resultShortUrl = computed(() => {
  const serverBaseUrl = import.meta.env.VITE_API_BASE_URL
  return `${serverBaseUrl}/${result.value?.short_url}`
})

const isVisibleResult = ref(false)

function copyShortUrl() {
  if (isVisibleTick.value) return

  // 1. Get the text from the div
  const text = document.getElementById('resultShortUrl')?.innerText ?? ''

  // 2. Use the Clipboard API to write text
  navigator.clipboard
    .writeText(text)
    .then(() => {
      isVisibleTick.value = true
      setTimeout(() => {
        isVisibleTick.value = false
      }, 3000)
    })
    .catch((err) => {
      console.error('Failed to copy: ', err)
    })
}

const isVisibleTick = ref(false)

const errorMessage = ref('')

function clearErrorMessage() {
  errorMessage.value = ''
}
const expiryDateResult = computed(() => {
  return new Date(result.value.expire as Date).toLocaleString('vi-vn', {
    day: '2-digit',
    month: '2-digit',
    year: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
    hour12: false,
  })
})
</script>

<template>
  <main>
    <section id="main">
      <div class="form-wrapper wrapper">
        <div class="title">Create short link with Sholy</div>

        <div class="form-parent">
          <div class="form-group">
            <div class="label">Long url</div>
            <input
              type="text"
              name="long-url"
              id="longUrl"
              @input="clearErrorMessage"
              v-model="request.long_url"
              placeholder="https://portal.azure.com/#allservices/category/All"
            />
          </div>
          <div class="form-group">
            <div class="label">Short url (Optional)</div>
            <input
              type="text"
              name="short-url"
              @input="clearErrorMessage"
              id="shortUrl"
              v-model="request.short_url"
              placeholder="azure-portal"
            />
          </div>
          <div class="form-group">
            <div class="label">Expiry dates from now (Optional)</div>
            <input
              type="text"
              name="expire"
              @input="clearErrorMessage"
              id="expire"
              placeholder="7"
              v-model="request.expire_in_number"
            />
          </div>
          <div class="form-group">
            <button @click="submitUrl">Get link</button>
          </div>
          <div class="error-message">{{ errorMessage }}</div>
        </div>
      </div>

      <div class="result-wrapper wrapper" v-if="isVisibleResult">
        <div class="short-link-result">
          <span>Created successfully:</span>
          <div class="value" id="resultShortUrl">{{ resultShortUrl }}</div>
          <div class="copy-button" @click="copyShortUrl">
            <span v-if="isVisibleTick" class="copied">Copied</span>
            <i v-else class="fa-regular fa-clone"></i>
          </div>
        </div>
        <div class="expiry-date">
          Expiry date:
          {{ expiryDateResult }}
        </div>
      </div>
    </section>
  </main>
</template>

<style>
#main {
  flex-direction: column;
  gap: 40px;
  /* width: 60%; */
  max-width: 600px;
  display: flex;
  margin: 0 16px;
}

main {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100vw;
  height: 100vh;
  background: #f5e9d8;
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
  padding: 12px;
  border-radius: 4px;
  border: 2px dashed #333;
  /* border: 1px solid #333; */
}

button {
  background-color: #e76f2e;
  color: #fff;
  padding: 12px;
  border-radius: 6px;
  cursor: pointer;
  border: 2px solid purple;
  margin-top: 12px;
  font-weight: 700;
}

.wrapper {
  padding: 24px;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  gap: 20px;
  background: white;
  box-shadow: rgba(99, 99, 99, 0.2) 0px 2px 8px 0px;
}

.copy-button {
  cursor: pointer;
  margin-right: 6px;
}

.short-link-result {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-direction: column;
}

.copied {
  display: inline-block;
  background: #333;
  color: #fff;
  border-radius: 4px;
  font-size: 0.9rem;
  padding: 0 4px;
}

.error-message {
  color: red;
  margin-top: 6px;
}

#resultShortUrl {
  max-width: calc(100vw - 42px);
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

@media screen and (min-width: 576px) {
  #main {
    width: 60%;
    max-width: 600px;
  }

  .short-link-result {
    flex-direction: row;
  }

  #resultShortUrl {
    max-width: unset;
  }
}
</style>
