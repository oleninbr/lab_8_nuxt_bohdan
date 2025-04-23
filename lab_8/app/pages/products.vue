<template>
    <div class="p-4">
      <Head>
        <Title>Список продуктів</Title>
      </Head>
  
      <h1 class="text-2xl font-bold mb-4">Список продуктів</h1>
  
      <UInput v-model="q" placeholder="Пошук..." class="mb-4 w-full" />
  
      <UTable
        :rows="filteredRows"
        :columns="columns"
        :sort="{ column: 'title', direction: 'asc' }"
        :page="{ size: 5 }"
      >
        <template #thumbnail-data="{ row }">
          <img :src="row.thumbnail" alt="thumbnail" width="100" height="100" >
        </template>
  
        <template #rating-data="{ row }">
          <span :class="row.rating < 4.5 ? 'text-red-600' : 'text-green-600'">
            {{ row.rating }}
          </span>
        </template>
      </UTable>
    </div>
  </template>
  
  <script setup>
  const q = ref('')
  const { data: products } = await useFetch('https://dummyjson.com/products', {
    transform: (data) => data.products
  })
  
  const columns = [
    { key: 'title', label: 'Назва', sortable: true },
    { key: 'description', label: 'Опис', sortable: true },
    { key: 'price', label: 'Ціна', sortable: true },
    { key: 'rating', label: 'Оцінка', sortable: true },
    { key: 'brand', label: 'Бренд', sortable: true },
    { key: 'category', label: 'Категорія', sortable: true },
    { key: 'thumbnail', label: 'Фото' }
  ]
  
  const filteredRows = computed(() => {
    if (!q.value) return products.value
    return products.value.filter((p) =>
      Object.values(p).some((v) => String(v).toLowerCase().includes(q.value.toLowerCase()))
    )
  })
  </script>
  