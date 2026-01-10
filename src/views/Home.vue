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

    <!-- 筛选栏 -->
    <div class="space-y-4 bg-white dark:bg-gray-800 p-4 rounded-xl shadow-sm border border-gray-100 dark:border-gray-700">
      <!-- 时间范围筛选 -->
      <div class="flex flex-col sm:flex-row sm:items-center gap-3 sm:gap-4">
        <div class="flex items-center justify-between sm:justify-start sm:space-x-4">
          <span class="text-sm font-medium text-gray-500">日期范围:</span>
          <button 
            v-if="startDate || endDate"
            @click="startDate = ''; endDate = ''"
            class="sm:hidden text-xs text-indigo-600 hover:text-indigo-500 font-medium"
          >
            重置日期
          </button>
        </div>
        
        <div class="flex items-center space-x-2">
          <input 
            v-model="startDate" 
            type="date" 
            class="flex-1 sm:flex-none px-2 py-1 rounded border border-gray-200 dark:border-gray-600 bg-gray-50 dark:bg-gray-700 text-sm outline-none focus:ring-2 focus:ring-indigo-500 min-w-[130px]"
          />
          <span class="text-gray-400">至</span>
          <input 
            v-model="endDate" 
            type="date" 
            class="flex-1 sm:flex-none px-2 py-1 rounded border border-gray-200 dark:border-gray-600 bg-gray-50 dark:bg-gray-700 text-sm outline-none focus:ring-2 focus:ring-indigo-500 min-w-[130px]"
          />
          <button 
            v-if="startDate || endDate"
            @click="startDate = ''; endDate = ''"
            class="hidden sm:block text-xs text-indigo-600 hover:text-indigo-500 font-medium whitespace-nowrap"
          >
            重置日期
          </button>
        </div>
      </div>

      <!-- 标签筛选 -->
      <div v-if="allTags.length > 0" class="flex flex-col sm:flex-row sm:items-center gap-2 pt-2 border-t border-gray-50 dark:border-gray-700">
        <span class="text-sm font-medium text-gray-500 sm:mr-2">标签筛选:</span>
        <div class="flex flex-wrap gap-2">
          <button
            @click="selectedTag = ''"
            :class="[
              'px-3 py-1 rounded-full text-sm transition font-medium',
              !selectedTag 
                ? 'bg-gray-800 text-white dark:bg-white dark:text-gray-900' 
                : 'bg-white dark:bg-gray-800 text-gray-600 dark:text-gray-400 hover:bg-gray-100 dark:hover:bg-gray-700 border border-transparent hover:border-gray-200 dark:hover:border-gray-600'
            ]"
          >
            全部
          </button>
          <button
            v-for="tag in allTags"
            :key="tag"
            @click="selectedTag = tag === selectedTag ? '' : tag"
            :class="[
              'px-3 py-1 rounded-full text-sm transition font-medium',
              selectedTag === tag 
                ? getTagStyle(tag, true) 
                : getTagStyle(tag, false) + ' hover:opacity-80'
            ]"
          >
            {{ tag }}
          </button>
        </div>
      </div>
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
            <span 
              v-for="tag in post.tags" 
              :key="tag" 
              :class="['px-2 py-0.5 rounded text-xs font-medium', getTagStyle(tag)]"
            >
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
const selectedTag = ref('')
const startDate = ref('')
const endDate = ref('')

// 预设颜色池
const colorMap = [
  { bg: 'bg-blue-100 text-blue-700 dark:bg-blue-900/30 dark:text-blue-300', active: 'bg-blue-600 text-white' },
  { bg: 'bg-green-100 text-green-700 dark:bg-green-900/30 dark:text-green-300', active: 'bg-green-600 text-white' },
  { bg: 'bg-purple-100 text-purple-700 dark:bg-purple-900/30 dark:text-purple-300', active: 'bg-purple-600 text-white' },
  { bg: 'bg-amber-100 text-amber-700 dark:bg-amber-900/30 dark:text-amber-300', active: 'bg-amber-600 text-white' },
  { bg: 'bg-rose-100 text-rose-700 dark:bg-rose-900/30 dark:text-rose-300', active: 'bg-rose-600 text-white' },
  { bg: 'bg-indigo-100 text-indigo-700 dark:bg-indigo-900/30 dark:text-indigo-300', active: 'bg-indigo-600 text-white' },
  { bg: 'bg-emerald-100 text-emerald-700 dark:bg-emerald-900/30 dark:text-emerald-300', active: 'bg-emerald-600 text-white' },
]

// 获取标签颜色的函数
const getTagStyle = (tag, isSelected = false) => {
  let hash = 0
  for (let i = 0; i < tag.length; i++) {
    hash = tag.charCodeAt(i) + ((hash << 5) - hash)
  }
  const index = Math.abs(hash) % colorMap.length
  return isSelected ? colorMap[index].active : colorMap[index].bg
}

const allTags = computed(() => {
  const tags = new Set()
  posts.value.forEach(post => {
    if (Array.isArray(post.tags)) {
      post.tags.forEach(tag => tags.add(tag))
    }
  })
  return Array.from(tags).sort()
})

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
  const modules = import.meta.glob('../posts/*.md', { query: '?raw', import: 'default', eager: true })
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

const filteredPosts = computed(() => {
  let result = posts.value

  // 1. 时间范围筛选
  if (startDate.value || endDate.value) {
    result = result.filter(post => {
      if (!post.date || post.date === '未知日期') return false
      const postDate = new Date(post.date).getTime()
      
      if (startDate.value && postDate < new Date(startDate.value).getTime()) {
        return false
      }
      if (endDate.value && postDate > new Date(endDate.value).getTime()) {
        return false
      }
      return true
    })
  }

  // 2. 标签筛选
  if (selectedTag.value) {
    result = result.filter(post => post.tags && post.tags.includes(selectedTag.value))
  }

  // 3. 模糊搜索
  if (searchQuery.value) {
    const fuseInstance = new Fuse(result, {
      keys: ['title'],
      threshold: 0.4
    })
    result = fuseInstance.search(searchQuery.value).map(r => r.item)
  }

  return result
})
</script>
