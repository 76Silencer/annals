<template>
  <div class="space-y-6">
    <div class="relative">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="搜索帖子标题..."
        class="w-full px-4 py-3 pl-10 rounded-lg border border-gray-300 dark:border-gray-700 bg-white dark:bg-gray-800 focus:ring-2 focus:ring-indigo-500 outline-none transition"
      />
      <span class="absolute left-3 top-3.5 text-gray-400">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
        </svg>
      </span>
    </div>

    <div v-if="filteredPosts.length > 0" class="grid gap-4">
      <router-link
        v-for="post in filteredPosts"
        :key="post.id"
        :to="{ name: 'post', params: { id: post.id } }"
        class="block p-6 bg-white dark:bg-gray-800 rounded-xl shadow-sm hover:shadow-md transition border border-transparent hover:border-indigo-100 dark:hover:border-indigo-900"
      >
        <h2 class="text-xl font-semibold text-gray-900 dark:text-gray-100 mb-2">{{ post.title }}</h2>
        <div class="flex items-center text-sm text-gray-500 space-x-4">
          <span>{{ post.date }}</span>
          <span v-if="post.tags" class="flex gap-2">
            <span v-for="tag in post.tags" :key="tag" class="px-2 py-0.5 bg-gray-100 dark:bg-gray-700 rounded text-xs">
              {{ tag }}
            </span>
          </span>
        </div>
      </router-link>
    </div>
    
    <div v-else class="text-center py-12 text-gray-500">
      未找到相关帖子
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import Fuse from 'fuse.js'

const posts = ref([])
const searchQuery = ref('')

const parseFrontmatter = (fileContent) => {
  const match = fileContent.match(/^---\r?\n([\s\S]+?)\r?\n---\r?\n([\s\S]*)$/)
  if (!match) return { data: {}, content: fileContent }
  
  const yaml = match[1]
  const body = match[2]
  const data = {}
  
  yaml.split('\n').forEach(line => {
    const [key, ...valueParts] = line.split(':')
    if (key && valueParts.length) {
      let val = valueParts.join(':').trim()
      // 简单的数组处理 [a, b]
      if (val.startsWith('[') && val.endsWith(']')) {
        val = val.slice(1, -1).split(',').map(s => s.trim())
      }
      data[key.trim()] = val
    }
  })
  
  return { data, content: body }
}

// 自动加载 posts 目录下的所有 md 文件
const loadPosts = async () => {
  const modules = import.meta.glob('../posts/*.md', { as: 'raw', eager: true })
  const postList = []

  for (const path in modules) {
    const fileContent = modules[path]
    const { data } = parseFrontmatter(fileContent)
    const id = path.split('/').pop().replace('.md', '')
    
    postList.push({
      id,
      title: data.title || id,
      date: data.date || '未知日期',
      tags: data.tags || [],
      content: fileContent
    })
  }
  
  // 按日期降序排序
  posts.value = postList.sort((a, b) => {
    const dateA = new Date(a.date)
    const dateB = new Date(b.date)
    return dateB - dateA
  })
}

onMounted(() => {
  loadPosts()
})

const fuse = computed(() => {
  return new Fuse(posts.value, {
    keys: ['title'],
    threshold: 0.4
  })
})

const filteredPosts = computed(() => {
  if (!searchQuery.value) return posts.value
  return fuse.value.search(searchQuery.value).map(result => result.item)
})
</script>

