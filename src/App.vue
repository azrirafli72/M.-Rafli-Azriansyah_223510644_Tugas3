<script setup>
  import { ref, onMounted, computed, watch } from 'vue';

  const todoList = ref([]);
  const nama = ref('');
  const konten = ref('');
  const kategori = ref(null);
  const currentPage = ref(1);
  const itemsPerPage = 2; // Jumlah item per halaman

  const todo_asc = computed(() => todoList.value.sort((a, b) => b.createdAt - a.createdAt));

  // Hitung indeks untuk pagination
  const startIndex = computed(() => (currentPage.value - 1) * itemsPerPage);
  const endIndex = computed(() => currentPage.value * itemsPerPage);

  // Data todo list yang sudah dipaginasi
  const paginatedTodoList = computed(() => todo_asc.value.slice(startIndex.value, endIndex.value));

  // Total halaman yang ada
  const totalPages = computed(() => Math.ceil(todo_asc.value.length / itemsPerPage));

  // Menambahkan todo baru
  const tambahTodo = () => {
    if (konten.value.trim() === '' || kategori.value === null) {
      return;
    }
    todoList.value.push({
      content: konten.value,
      category: kategori.value,
      done: false,
      createdAt: new Date().getTime(),
    });
    konten.value = '';
    kategori.value = null;
  };

  // Menghapus todo dari list
  const hapusTodo = (todo) => {
    todoList.value = todoList.value.filter((t) => t !== todo);
  };

  // Tandai todo sebagai selesai
  const tandaiSelesai = (todo) => {
    todo.done = !todo.done;
  };

  // Simpan data todo list ke localStorage
  watch(todoList, (newVal) => {
    localStorage.setItem('todoList', JSON.stringify(newVal));
  }, { deep: true });

  // Memuat data dari localStorage saat komponen dimuat
  onMounted(() => {
    nama.value = localStorage.getItem('nama') || '';
    todoList.value = JSON.parse(localStorage.getItem('todoList')) || [];
  });
</script>

<template>
  <div class="container">
    <main class="app">
      <!-- Bagian Sambutan -->
      <section class="sambutan">
        <h2 class="judul text-align-center">
          Selamat Datang di Aplikasi, <input type="text" placeholder="Tulis Nama Anda" v-model="nama" />
        </h2>
      </section>

      <!-- Form Tambah Todo -->
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

          <input type="submit" class="btn btn-primary" value="Tambahkan ke List" />
        </form>
      </section>

      <br />

      <!-- Daftar Todo List -->
      <section class="todo-list">
        <h3>Daftar TODO LIST Anda. Dapat Melakukan Proses CRUD</h3>
        <div class="list-container">
          <div class="list">
            <div v-for="todo in paginatedTodoList" :class="`todo-item ${todo.done && 'done'}`" :key="todo.createdAt">
              <div class="card">
                <div class="card-body">
                  <div :class="{ 'todo-content': true, 'done': todo.done }">
                    <input type="text" v-model="todo.content" />
                  </div>

                  <div class="aksi">
                    <label>
                      <input type="checkbox" v-model="todo.done" @change="tandaiSelesai(todo)" />
                      <span :class="['btn', 'btn-warning', todo.category]" @click="tandaiSelesai(todo)">
                        {{ todo.done ? 'Batalkan Selesai' : 'Tandai Selesai' }}
                      </span>
                    </label>
                    <button class="btn btn-danger" @click="hapusTodo(todo)">Hapus</button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Pagination -->
          <div class="pagination">
            <button :disabled="currentPage === 1" @click="currentPage--">Prev</button>
            <span>Halaman {{ currentPage }} dari {{ totalPages }}</span>
            <button :disabled="currentPage === totalPages" @click="currentPage++">Next</button>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
.container {
  max-width: 900px;
  margin: 40px auto;
  padding: 20px;
  background-color: #f9f9f9;
  border: 1px solid #ddd;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 10px;
  display: flex;
  justify-content: center; /* Mengatur tata letak horizontal tengah */
  align-items: center; /* Mengatur tata letak vertikal tengah */
}

.card {
  width: 100%;
  background-color: #fff;
  border: 1px solid #ddd;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  padding: 20px;
  transition: all 0.3s ease;
}

.card:hover {
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
}

.card-body {
  display: flex;
  justify-content: space-between; /* Menyusun konten dan tombol ke kanan */
  align-items: center;
  width: 100%;
  flex-direction: column; /* Stack the todo content and actions vertically */
  align-items: stretch; /* Stretch the todo content and actions to fill the available space */
}

.todo-content {
  margin-bottom: 10px;
  flex: 1; /* Konten todo list mengisi sisa ruang yang tersedia */
  overflow-wrap: break-word; /* Allow long text to wrap to the next line */
  word-break: break-all; /* Break long words to the next line */
  white-space: normal; /* Allow text to wrap to the next line */
  text-align: left; /* Left-align the text */
  padding: 10px; /* Add some padding to the text */
  border: 1px solid #ddd; /* Add a border to the text */
  border-radius: 2px; /* Add a border radius to the text */
}

.todo-content input {
  width: 100%; /* Lebar input mengisi seluruh ruang yang tersedia */
  box-sizing: border-box; /* Menyertakan padding dan border dalam perhitungan lebar */
}

.todo-item.done .todo-content {
  text-decoration: line-through; /* Tandai konten yang selesai dengan garis melintang */
}

.aksi {
  display: flex;
  align-items: center;
  margin-top: 10px; /* Add some space between the todo content and the actions */
  flex-shrink: 0; /* Prevent the actions from shrinking */
}

.aksi label {
  margin-right: 10px;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.pagination button {
  margin: 0 5px;
  padding: 8px 16px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.pagination button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.aksi label span {
  text-decoration: none; /* Remove line-through on button text */
}
</style>
