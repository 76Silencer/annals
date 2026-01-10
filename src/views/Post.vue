<template>
  <div v-if="post" class="bg-white dark:bg-gray-800 rounded-2xl shadow-sm p-6 sm:p-10">
    <header class="mb-8 pb-8 border-b border-gray-100 dark:border-gray-700">
      <router-link to="/" class="text-indigo-600 hover:text-indigo-500 text-sm mb-4 inline-block flex items-center">
        <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" />
        </svg>
        返回列表
      </router-link>
      <h1 class="text-3xl font-bold text-gray-900 dark:text-gray-100 mb-4">{{ post.title }}</h1>
      <div class="flex items-center text-gray-500 text-sm space-x-4">
        <span>{{ post.date }}</span>
        <div v-if="post.tags" class="flex gap-2">
          <span 
            v-for="tag in post.tags" 
            :key="tag" 
            :class="['px-2 py-0.5 rounded text-xs font-medium', getTagStyle(tag)]"
          >
            {{ tag }}
          </span>
        </div>
      </div>
    </header>

    <article class="prose dark:prose-invert max-w-none" v-html="renderedContent"></article>
  </div>
  <div v-else class="text-center py-20">
    加载中...
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { useRoute } from 'vue-router'
import MarkdownIt from 'markdown-it'

const props = defineProps(['id'])
const route = useRoute()
const post = ref(null)
const md = new MarkdownIt({
  html: true,
  linkify: true,
  typographer: true
})

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
      if (val.startsWith('[') && val.endsWith(']')) {
        val = val.slice(1, -1).split(',').map(s => s.trim())
      }
      data[key.trim()] = val
    }
  })
  
  return { data, content: body }
}

const renderedContent = computed(() => {
  if (!post.value) return ''
  return md.render(post.value.content)
})

const loadPost = async () => {
  const id = props.id || route.params.id
  try {
    // 动态导入 md 文件
    const modules = import.meta.glob('../posts/*.md', { query: '?raw', import: 'default' })
    const path = `../posts/${id}.md`
    
    if (modules[path]) {
      const fileContent = await modules[path]()
      const { data, content: markdownBody } = parseFrontmatter(fileContent)
      
      post.value = {
        title: data.title || id,
        date: data.date || '未知日期',
        tags: data.tags || [],
        content: markdownBody
      }
    }
  } catch (error) {
    console.error('Failed to load post:', error)
  }
}

onMounted(() => {
  loadPost()
})
</script>

<style>
/* 自定义一些文章样式 */
.prose h1 { @apply text-3xl font-bold mb-6 mt-8; }
.prose h2 { @apply text-2xl font-semibold mb-4 mt-6; }
.prose p { @apply mb-4 leading-relaxed; }
.prose ul { @apply list-disc pl-6 mb-4; }
.prose ol { @apply list-decimal pl-6 mb-4; }
.prose blockquote { @apply border-l-4 border-gray-300 dark:border-gray-600 pl-4 italic my-4; }
.prose code { @apply bg-gray-100 dark:bg-gray-700 px-1 rounded text-sm; }
.prose pre { @apply bg-gray-900 text-gray-100 p-4 rounded-lg overflow-x-auto my-4; }
.prose pre code { @apply bg-transparent p-0; }
</style>

