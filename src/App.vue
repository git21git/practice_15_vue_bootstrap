<template>
  <div class="app-shell">
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary">
      <div class="container">
        <span class="navbar-brand fw-semibold">Моя коллекция книг</span>
        <span class="badge text-bg-light text-primary">Практика 15</span>
      </div>
    </nav>

    <main class="container py-4 py-lg-5">
      <header class="mb-4">
        <p class="text-uppercase text-primary fw-semibold small mb-2">Vue 3 + Bootstrap 5</p>
        <h1 class="h2 mb-2">Личная библиотека</h1>
        <p class="text-secondary mb-0">
          Добавляйте книги, ищите по названию или автору, сортируйте список и сохраняйте коллекцию в браузере.
        </p>
      </header>

      <section class="toolbar border rounded-3 p-3 p-lg-4 mb-4">
        <form class="row g-3 align-items-end" @submit.prevent="addBook">
          <div class="col col-12 col-md-6">
            <label class="form-label" for="title">Название книги</label>
            <input
              id="title"
              v-model.trim="newBook.title"
              class="form-control"
              type="text"
              placeholder="Например, Мастер и Маргарита"
            />
          </div>

          <div class="col col-12 col-md-6">
            <label class="form-label" for="author">Автор</label>
            <input
              id="author"
              v-model.trim="newBook.author"
              class="form-control"
              type="text"
              placeholder="Например, Михаил Булгаков"
            />
          </div>

          <div class="col col-12">
            <label class="form-label" for="description">Описание</label>
            <textarea
              id="description"
              v-model.trim="newBook.description"
              class="form-control"
              rows="3"
              placeholder="Короткая заметка о книге"
            ></textarea>
          </div>

          <div class="col col-12 col-lg-8">
            <label class="form-label" for="cover">Ссылка на фото обложки</label>
            <input
              id="cover"
              v-model.trim="newBook.cover"
              class="form-control"
              type="url"
              placeholder="https://example.com/book-cover.jpg"
            />
          </div>

          <div class="col col-12 col-lg-4">
            <button class="btn btn-primary w-100" type="submit">Добавить книгу</button>
          </div>
        </form>

        <div v-if="formMessage" class="alert alert-warning mb-0 mt-3" role="alert">
          {{ formMessage }}
        </div>
      </section>

      <section class="toolbar border rounded-3 p-3 p-lg-4 mb-4">
        <div class="row g-3 align-items-end">
          <div class="col col-12 col-lg-5">
            <label class="form-label" for="search">Поиск</label>
            <div class="input-group">
              <span class="input-group-text">Найти</span>
              <input
                id="search"
                v-model.trim="searchQuery"
                class="form-control"
                type="search"
                placeholder="Название или автор"
              />
            </div>
          </div>

          <div class="col col-12 col-md-6 col-lg-3">
            <label class="form-label" for="filter">Фильтр</label>
            <select id="filter" v-model="activeFilter" class="form-control">
              <option value="all">Все книги</option>
              <option value="withDescription">С описанием</option>
              <option value="withoutDescription">Без описания</option>
              <option value="withCover">С фото обложки</option>
              <option value="withoutCover">Без фото обложки</option>
            </select>
          </div>

          <div class="col col-12 col-md-6 col-lg-4">
            <label class="form-label" for="sort">Сортировка</label>
            <select id="sort" v-model="sortMode" class="form-control">
              <option value="added">По порядку добавления</option>
              <option value="titleAsc">По названию: А-Я</option>
              <option value="titleDesc">По названию: Я-А</option>
            </select>
          </div>
        </div>
      </section>

      <div class="d-flex flex-wrap align-items-center gap-2 mb-3">
        <span class="badge text-bg-primary">Всего книг: {{ books.length }}</span>
        <span class="badge text-bg-success">Показано: {{ filteredBooks.length }}</span>
        <span v-if="searchQuery" class="badge text-bg-secondary">Поиск: {{ searchQuery }}</span>
      </div>

      <div v-if="filteredBooks.length" class="row g-4">
        <div v-for="book in filteredBooks" :key="book.id" class="col col-12 col-md-6 col-xl-4">
          <article class="card h-100 shadow-sm">
            <img
              v-if="book.cover"
              :src="book.cover"
              class="card-img-top cover-image"
              :alt="`Обложка книги ${book.title}`"
            />
            <div
              v-else
              class="cover-placeholder bg-light d-flex align-items-center justify-content-center text-secondary"
            >
              Обложка не добавлена
            </div>

            <div class="card-body d-flex flex-column">
              <div class="d-flex justify-content-between align-items-start gap-2 mb-2">
                <h2 class="card-title h5 mb-0">{{ book.title }}</h2>
                <span class="badge text-bg-info">Книга</span>
              </div>

              <p class="text-secondary mb-2">{{ book.author }}</p>

              <p v-if="book.description" class="book-description mb-3">
                {{ book.description }}
              </p>
              <p v-else class="book-description text-secondary mb-3">
                Описание пока не добавлено.
              </p>

              <div class="d-flex flex-wrap gap-2 mb-3">
                <span v-if="book.description" class="badge text-bg-success">Есть описание</span>
                <span v-else class="badge text-bg-secondary">Без описания</span>
                <span v-if="book.cover" class="badge text-bg-success">Есть фото</span>
                <span v-else class="badge text-bg-secondary">Без фото</span>
              </div>

              <button class="btn btn-danger mt-auto" type="button" @click="deleteBook(book.id)">
                Удалить книгу
              </button>
            </div>
          </article>
        </div>
      </div>

      <div v-else class="alert alert-info" role="alert">
        Книги не найдены. Измените поиск, фильтр или добавьте новую книгу.
      </div>
    </main>
  </div>
</template>

<script setup>
import { computed, ref, watch } from 'vue'

const STORAGE_KEY = 'practice-15-book-collection'
const baseUrl = import.meta.env.BASE_URL

const defaultBooks = [
  {
    id: 1,
    title: 'Маленький принц',
    author: 'Антуан де Сент-Экзюпери',
    description: 'Философская сказка о дружбе, ответственности и умении видеть главное.',
    cover: `${baseUrl}covers/little-prince.svg`,
    createdAt: 1,
  },
  {
    id: 2,
    title: 'Гарри Поттер и философский камень',
    author: 'Дж. К. Роулинг',
    description: 'История о первой встрече Гарри с миром магии и школой Хогвартс.',
    cover: '',
    createdAt: 2,
  },
  {
    id: 3,
    title: '451 градус по Фаренгейту',
    author: 'Рэй Брэдбери',
    description: '',
    cover: `${baseUrl}covers/fahrenheit-451.svg`,
    createdAt: 3,
  },
]

const loadBooks = () => {
  const savedBooks = localStorage.getItem(STORAGE_KEY)

  if (!savedBooks) {
    return defaultBooks
  }

  try {
    const parsedBooks = JSON.parse(savedBooks)
    return Array.isArray(parsedBooks) ? parsedBooks : defaultBooks
  } catch {
    return defaultBooks
  }
}

const books = ref(loadBooks())
const searchQuery = ref('')
const activeFilter = ref('all')
const sortMode = ref('added')
const formMessage = ref('')

const newBook = ref({
  title: '',
  author: '',
  description: '',
  cover: '',
})

const filteredBooks = computed(() => {
  const query = searchQuery.value.toLowerCase()

  let result = books.value.filter((book) => {
    const matchesSearch =
      book.title.toLowerCase().includes(query) || book.author.toLowerCase().includes(query)

    const hasDescription = Boolean(book.description)
    const hasCover = Boolean(book.cover)

    const matchesFilter =
      activeFilter.value === 'all' ||
      (activeFilter.value === 'withDescription' && hasDescription) ||
      (activeFilter.value === 'withoutDescription' && !hasDescription) ||
      (activeFilter.value === 'withCover' && hasCover) ||
      (activeFilter.value === 'withoutCover' && !hasCover)

    return matchesSearch && matchesFilter
  })

  if (sortMode.value === 'titleAsc') {
    result = [...result].sort((a, b) => a.title.localeCompare(b.title, 'ru'))
  }

  if (sortMode.value === 'titleDesc') {
    result = [...result].sort((a, b) => b.title.localeCompare(a.title, 'ru'))
  }

  return result
})

const resetForm = () => {
  newBook.value = {
    title: '',
    author: '',
    description: '',
    cover: '',
  }
}

const addBook = () => {
  formMessage.value = ''

  if (!newBook.value.title || !newBook.value.author) {
    formMessage.value = 'Заполните название книги и автора.'
    return
  }

  books.value.unshift({
    id: Date.now(),
    title: newBook.value.title,
    author: newBook.value.author,
    description: newBook.value.description,
    cover: newBook.value.cover,
    createdAt: Date.now(),
  })

  resetForm()
}

const deleteBook = (bookId) => {
  books.value = books.value.filter((book) => book.id !== bookId)
}

watch(
  books,
  (updatedBooks) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(updatedBooks))
  },
  { deep: true },
)
</script>
