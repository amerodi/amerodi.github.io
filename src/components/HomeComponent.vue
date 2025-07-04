<script setup>
import { ref, computed, onMounted } from 'vue'
import CardList from './CardList.vue'
import TagCloud from './TagCloud.vue'

const items = ref([])
const search = ref('')
const selectedTags = ref([])

onMounted(async () => {
  const response = await fetch('/data.json')
  items.value = await response.json()
})

const allTags = computed(() => {
  const tags = new Set()
  items.value.forEach(item => {
    (item.tags || []).forEach(tag => tags.add(tag))
  })
  return Array.from(tags).sort()
})

const filteredItems = computed(() => {
  return items.value.filter(item => {
    const matchSearch =
      !search.value ||
      item.project_name.toLowerCase().includes(search.value.toLowerCase()) ||
      (item.description && item.description.toLowerCase().includes(search.value.toLowerCase()))

    const matchTags = selectedTags.value.length === 0 ||
      selectedTags.value.every(selectedTag => (item.tags || []).includes(selectedTag))

    return matchSearch && matchTags
  })
})
</script>

<style scoped>
@import 'https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css';

/* Dark theme overrides */
header {
  border-color: var(--color-border) !important;
}

h1, h2 {
  color: var(--color-heading) !important;
}

.form-control {
  background-color: var(--color-background-soft);
  border-color: var(--color-border);
  color: var(--color-text);
}

.form-control:focus {
  background-color: var(--color-background-soft);
  border-color: var(--color-border-hover);
  color: var(--color-text);
  box-shadow: 0 0 0 0.2rem rgba(255, 255, 255, 0.1);
}

.form-control::placeholder {
  color: var(--color-text);
  opacity: 0.6;
}
</style>

<template>
  <header class="d-flex align-items-center justify-content-between p-3 mb-4 border-bottom">
    <h1 class="mb-0">A Medical Robot Directory</h1>
    <img src="/src/assets/bot.png" alt="Bot" style="height:48px; width:auto;" />
  </header>
  <main class="container-fluid py-4">
    <div class="row">
      <section class="col-md-9 mb-4">
        <input
            v-model="search"
            type="text"
            placeholder="Search for a project ..."
            class="form-control mb-3"
        />
        <CardList :items="filteredItems" />
      </section>
      <aside class="col-md-3">
        <h2 class="h5 mb-3">Tags</h2>
        <TagCloud :tags="allTags" v-model:selectedTags="selectedTags" />
      </aside>
    </div>
  </main>
</template>
