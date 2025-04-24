<script setup lang="ts">
import { ref, computed } from 'vue'
import { useFetch } from '#app'

// Оптимізація заголовка сторінки
useHead({
  title: 'Список продуктів'
})

// Завантаження даних з API
interface Product {
  id: number
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
}

const { data, pending: isLoading, error } = useFetch<{ products: Product[] }>('https://dummyjson.com/products')
const products = computed(() => data.value?.products || [])

// Стан для пошуку, сортування та пагінації
const filterText = ref('')
const currentPage = ref(1)
const rowsPerPage = ref(5)
const sortKey = ref('')
const sortDirection = ref<'asc' | 'desc'>('asc')

// Кастомна функція сортування
function customOrderBy<T>(
  array: T[],
  key: keyof T,
  direction: 'asc' | 'desc'
): T[] {
  return array.slice().sort((a, b) => {
    const valueA = a[key]
    const valueB = b[key]

    if (valueA < valueB) return direction === 'asc' ? -1 : 1
    if (valueA > valueB) return direction === 'asc' ? 1 : -1
    return 0
  })
}

// Фільтрація
const filteredData = computed(() =>
  products.value.filter((row) =>
    Object.values(row).some((value) =>
      String(value).toLowerCase().includes(filterText.value.toLowerCase())
    )
  )
)

// Сортування
const sortedData = computed(() =>
  customOrderBy(filteredData.value, sortKey.value as keyof Product, sortDirection.value)
)

// Пагінація
const paginatedData = computed(() =>
  sortedData.value.slice(
    (currentPage.value - 1) * rowsPerPage.value,
    currentPage.value * rowsPerPage.value
  )
)

// Зміна сортування
const changeSort = (key: string) => {
  if (sortKey.value === key) {
    sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc'
  } else {
    sortKey.value = key
    sortDirection.value = 'asc'
  }
}
</script>

<template>
  <div class="container">
    <h1 class="title">Список продуктів</h1>

    <!-- Пошук -->
    <div class="search-wrap">
      <input
        v-model="filterText"
        type="text"
        placeholder="🔍 Пошук продуктів..."
        class="search-input"
      >
    </div>

    <!-- Таблиця -->
    <div class="table-container">
      <n-table v-if="!isLoading" class="custom-table">
        <thead>
          <tr>
            <th class="sortable" @click="changeSort('title')">
              Назва
              <span v-if="sortKey === 'title'">({{ sortDirection }})</span>
            </th>
            <th>Опис</th>
            <th class="sortable" @click="changeSort('price')">
              Ціна
              <span v-if="sortKey === 'price'">({{ sortDirection }})</span>
            </th>
            <th class="sortable" @click="changeSort('rating')">
              Оцінка
              <span v-if="sortKey === 'rating'">({{ sortDirection }})</span>
            </th>
            <th>Бренд</th>
            <th>Категорія</th>
            <th>Фото</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="product in paginatedData" :key="product.id" class="table-row">
            <td>{{ product.title }}</td>
            <td>{{ product.description }}</td>
            <td>{{ product.price }} $</td>
            <td :class="{ 'high-rating': product.rating >= 4.5, 'low-rating': product.rating < 4.5 }">
              {{ product.rating }}
            </td>
            <td>{{ product.brand }}</td>
            <td>{{ product.category }}</td>
            <td>
              <img :src="product.thumbnail" alt="Фото" class="product-img" >
            </td>
          </tr>
        </tbody>
      </n-table>

      <!-- Завантаження -->
      <div v-else-if="isLoading" class="loading">Завантаження...</div>
      <div v-else-if="error" class="error">Помилка завантаження даних: {{ error.message }}</div>
    </div>

    <div class="pagination">
      <button :disabled="currentPage === 1" class="btn" @click="currentPage--">
        Назад
      </button>
      <span>Сторінка {{ currentPage }} із {{ Math.ceil(filteredData.length / rowsPerPage) }}</span>
      <button
        :disabled="currentPage === Math.ceil(filteredData.length / rowsPerPage)"
        class="btn"
        @click="currentPage++"
      >
        Вперед
      </button>
    </div>
  </div>
</template>

<style>

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.title {
  text-align: center;
  font-size: 24px;
  margin-bottom: 20px;
}

.search-wrap {
  margin-bottom: 20px;
  text-align: center;
}

.search-input {
  padding: 10px;
  width: 300px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.table-container {
  overflow-x: auto;
}

.custom-table {
  width: 100%;
  border-collapse: collapse;
}

.custom-table th,
.custom-table td {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: left;
}

.custom-table th.sortable {
  cursor: pointer;
}

.custom-table th.sortable:hover {
  background-color: #f0f0f0;
}

.product-img {
  width: 100px;
  height: 100px;
  object-fit: cover;
}


.high-rating {
  color: green;
}

.low-rating {
  color: red;
}

.loading,
.error {
  text-align: center;
  font-size: 18px;
  margin-top: 20px;
}

.error {
  color: red;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 20px;
}

.pagination .btn {
  padding: 10px 20px;
  margin: 0 5px;
  border: none;
  background-color: #007bff;
  color: white;
  border-radius: 5px;
  cursor: pointer;
}

.pagination .btn:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
</style>