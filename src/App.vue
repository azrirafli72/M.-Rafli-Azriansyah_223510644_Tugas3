<script setup>
import { ref, onMounted, computed, watch } from 'vue'

const todoList = ref([])
const nama = ref('')
const konten = ref('')
const kategori = ref(null)

const todo_asc = computed(() => todoList.value.sort((a, b) => {
  return b.createdAt - a.createdAt
}))

const tambahTodo = () => {
  if (konten.value.trim() === '' || kategori.value === null) {
    return
  }
  todoList.value.push({
    content: konten.value,
    category: kategori.value,
    done: false,
    createdAt: new Date().getTime()
  })
  konten.value = ''
  kategori.value = null
}

watch(todoList, newVal => {
  localStorage.setItem('todoList', JSON.stringify(newVal))
}, { deep: true })

watch(nama, (newVal) => {
  localStorage.setItem('nama', newVal)
})

const hapusTodo = todo => {
  todoList.value = todoList.value.filter(t => t !== todo)
}

onMounted(() => {
  nama.value = localStorage.getItem('nama') || ''
  todoList.value = JSON.parse(localStorage.getItem('todoList')) || []
})
</script>

<template>
  <div class="container">
    <main class="app">
      <!-- Untuk Bagian Sambutan pada Pengunjung Web -->
      <section class="sambutan">
        <h2 class="judul text-align-center">
          Selamat Datang di Aplikasi, <input type="text" placeholder="Tulis Nama Anda" v-model="nama" />
        </h2>
      </section>
      <!-- Untuk Bagian Proses Pembuatan To-do List Baru -->
      <section class="buat-todo">
        <h3>TAMBAH TODO LIST BARU</h3>

        <form @submit.prevent="tambahTodo">
          <h4>Apa yang terdapat pada Tugas / Todo List Anda?</h4>
          <input type="text" placeholder="Contoh: Baca Al-quran, Kuliah, Menyapu Halaman, dll" v-model="konten" />

          <h4>Pilih Kategori Tugas / Todo List Anda</h4>
          <div class="pilihan">

            <label>
              <input type="radio" name="category" value="Bisnis Pekerjaan" v-model="kategori" />
              <span class="ceklisBulat bisnis"></span>
              <div>Bisnis Pekerjaan</div>
            </label>

            <label>
              <input type="radio" name="category" value="Hobby / Pribadi" v-model="kategori" />
              <span class="ceklisBulat hobipribadi"></span>
              <div>Hobi untuk Kebutuhan Pribadi</div>
            </label>
          </div>

          <input type="submit" class="btn btn primary" value="Tambahkan ke List" />
        </form>
      </section>

      <br>

      <section class="todo-list">
        <h3>Daftar TODO LIST Anda. Dapat Melakukan Proses CRUD</h3>
        <div class="list">
          <div v-for="todo in todo_asc" :class="`todo-item ${todo.done && 'done'}`">
            <div class="todo-content">
              <input type="text" v-model="todo.content">
            </div>

            <label>
              <input type="checkbox" v-model="todo.done" />
              <span :class="`btn btn-warning ${todo.category}`">Tandai Sebagai Selesai</span>
            </label>

            <div class="aksi">
              <button class="btn btn-danger" @click="hapusTodo(todo)">Hapus</button>
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>