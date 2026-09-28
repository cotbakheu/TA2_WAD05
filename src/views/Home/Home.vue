<script setup lang="ts">
import { onMounted, ref, watch } from 'vue'

type User = {
  id: number
  name: string
  username: string
  email: string
  phone: string
  company: {
    name: string
  }
  address: {
    street: string
    suite: string
    city: string
    zipcode: string
  }
  showDetails: boolean
}

const baseUrl = 'https://jsonplaceholder.typicode.com'
let url = `${baseUrl}/users`

// state variables
let errorMessage = ref('')
let isLoading = ref(false)
let users = ref<User[]>([])
let searchInput = ref('')
let sortOrder = ref<'asc' | 'desc'>('asc')

const totalUsers = 10

async function fetchUsers(): Promise<User[]> {
  isLoading.value = true
  users.value = []
  errorMessage.value = ''
  try {
    const response = await fetch(url)
    if (!response.ok) {
      throw new Error(`Failed to load users. Please try again later.`)
    }
    const users = await response.json()
    if (users.length === 0) {
      throw new Error(`No users found. Please try again with a different search term.`)
    }
    isLoading.value = false
    return users
  } catch (error: unknown) {
    isLoading.value = false
    errorMessage.value = `Failed to load users. Please try again later.`
    if (error instanceof Error && error.message) {
      errorMessage.value = error.message
    }
    return []
  }
}

async function loadUsers() {
  const loadedUsers = await fetchUsers()
  users.value = loadedUsers.map((user: User) => ({
    ...user,
    showDetails: false,
  }))
  sortUsers()
}

function searchUsers(event: Event) {
  event.preventDefault()
  url = `${baseUrl}/users?${searchInput.value ? `name=${encodeURIComponent(searchInput.value)}` : ''}`
  loadUsers()
}

function sortUsersAZ() {
  users.value = users.value.sort((a, b) => a.name.localeCompare(b.name))
}

function sortUsersZA() {
  users.value = users.value.sort((a, b) => b.name.localeCompare(a.name))
}

function sortUsers() {
  if (sortOrder.value === 'asc') {
    sortUsersAZ()
  } else if (sortOrder.value === 'desc') {
    sortUsersZA()
  }
}

function resetSearch() {
  searchInput.value = ''
  url = `${baseUrl}/users`
  loadUsers()
}

watch(sortOrder, () => {
  sortUsers()
})

onMounted(() => {
  loadUsers()
})
</script>

<template>
  <div class="container pt-5 w-50">
    <nav class="navbar navbar-light bg-teal mb-3 rounded">
      <div class="container-fluid">
        <span class="navbar-brand mb-0 h1 text-white">Dashboard Vue — Pertemuan 5</span>
      </div>
    </nav>
    <div class="d-flex align-items-end mb-3 gap-2">
      <button class="btn btn-success" @click="searchUsers">Muat Pengguna</button>
      <div class="flex-grow-1">
        <input type="text" class="form-control" v-model="searchInput" placeholder="Cari nama..." />
      </div>
      <button
        class="btn"
        :class="[sortOrder === 'asc' ? 'btn-primary' : 'btn-outline-primary']"
        @click="sortOrder = 'asc'"
      >
        Urutkan A-Z
      </button>
      <button
        class="btn"
        :class="[sortOrder === 'desc' ? 'btn-primary' : 'btn-outline-primary']"
        @click="sortOrder = 'desc'"
      >
        Urutkan Z-A
      </button>
      <button class="btn btn-secondary" @click="resetSearch()">Muat Ulang</button>
    </div>
    <p>Menampilkan user {{ users.length }} dari {{ totalUsers }}</p>
    <div class="card mb-2" v-for="user in users" :key="user.id">
      <div class="card-body">
        <div class="d-flex justify-content-between align-items-start">
          <div>
            <h5 class="card-title">{{ user.name }}</h5>
            <p class="card-text">{{ user.email }}</p>
          </div>
          <button
            class="btn"
            :class="[{ 'btn-outline-info': !user.showDetails }, { 'btn-info': user.showDetails }]"
            @click="user.showDetails = !user.showDetails"
          >
            {{ user.showDetails ? 'Sembunyikan' : 'Lihat Detail' }}
          </button>
        </div>
        <div v-if="user.showDetails" class="p-2 border-top border-1 border-secondary mt-2">
          <p class="card-text mb-0"><strong>Username:</strong> {{ user.username }}</p>
          <p class="card-text mb-0"><strong>Phone:</strong> {{ user.phone }}</p>
          <p class="card-text mb-0"><strong>Company:</strong> {{ user.company.name }}</p>
          <p class="card-text mb-0">
            <strong>Address:</strong> {{ user.address.street }}, {{ user.address.suite }},
            {{ user.address.city }}, {{ user.address.zipcode }}
          </p>
        </div>
      </div>
    </div>
    <div v-if="isLoading" class="d-flex justify-content-center">
      <div id="spinner" class="spinner-border m-5" role="status">
        <span class="visually-hidden">Loading...</span>
      </div>
    </div>
    <div v-if="errorMessage" class="alert alert-danger" role="alert" id="error-message">
      {{ errorMessage }}
    </div>
  </div>
</template>

<style scoped></style>
