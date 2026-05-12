<template>
  <AppLayout>
    <div class="h-[calc(100vh-10rem)]">
      <div class="mx-auto flex h-full max-w-7xl flex-col overflow-hidden rounded-[28px] border border-gray-200 bg-white text-gray-900 shadow-xl dark:border-dark-700 dark:bg-dark-900 dark:text-white">
      <div class="flex items-center justify-between border-b border-gray-100 dark:border-dark-700 px-6 py-4">
        <div>
          <h1 class="text-lg font-semibold">SecretAPI Chat</h1>
          <p class="text-xs text-gray-500 dark:text-gray-400">Chat with your API models</p>
        </div>

        <select
          v-model="model"
          class="rounded-full border border-gray-200 bg-gray-50 px-4 py-2 text-sm text-gray-800 outline-none dark:border-dark-600 dark:bg-dark-800 dark:text-white"
        >
          <option class="text-black" value="gpt-4.1-mini">gpt-4.1-mini</option>
          <option class="text-black" value="gpt-4.1">gpt-4.1</option>
          <option class="text-black" value="gpt-4o-mini">gpt-4o-mini</option>
        </select>
      </div>

      <div ref="chatBody" class="flex-1 overflow-y-auto px-6 py-8">
        <div v-if="messages.length === 0" class="flex h-full flex-col items-center justify-center text-center">
          <div class="mb-6 flex h-14 w-14 items-center justify-center rounded-2xl bg-primary-600 text-xl font-bold text-white">
            S
          </div>
          <h2 class="text-3xl font-semibold tracking-tight">What can I help with?</h2>
          <p class="mt-3 max-w-md text-sm text-gray-500 dark:text-gray-400">
            Ask anything using your SecretAPI endpoint.
          </p>
        </div>

        <div v-else class="mx-auto max-w-3xl space-y-6">
          <div
            v-for="(msg, index) in messages"
            :key="index"
            :class="msg.role === 'user' ? 'flex justify-end' : 'flex justify-start'"
          >
            <div
              :class="[
                'max-w-[80%] rounded-3xl px-5 py-4 text-sm leading-6',
                msg.role === 'user'
                  ? 'bg-primary-600 text-white'
                  : 'border border-gray-200 bg-gray-50 text-gray-900 dark:border-dark-700 dark:bg-dark-800 dark:text-white'
              ]"
            >
              <div class="mb-1 text-xs font-semibold opacity-50">
                {{ msg.role === 'user' ? 'You' : 'SecretAPI' }}
              </div>
              <div class="whitespace-pre-wrap">{{ msg.content }}</div>
            </div>
          </div>

          <div v-if="loading" class="flex justify-start">
            <div class="rounded-3xl border border-gray-200 bg-gray-50 px-5 py-4 text-sm text-gray-500 dark:border-dark-700 dark:bg-dark-800 dark:text-gray-400">
              Thinking...
            </div>
          </div>
        </div>
      </div>

      <div class="px-6 pb-6">
        <div class="mx-auto max-w-3xl rounded-[32px] border border-gray-200 bg-white p-2 shadow-xl dark:border-dark-700 dark:bg-dark-800">
          <div class="flex items-center gap-3 rounded-[26px] bg-gray-50 px-4 py-3 dark:bg-dark-900">
            <button
              class="flex h-8 w-8 items-center justify-center rounded-full border border-gray-200 text-xl text-gray-500 dark:border-dark-600 dark:text-gray-300"
              type="button"
              @click="clearChat"
              title="New chat"
            >
              +
            </button>

            <textarea
              v-model="input"
              rows="1"
              class="max-h-32 flex-1 resize-none bg-transparent text-sm text-white outline-none placeholder:text-gray-500 dark:text-gray-400"
              placeholder="Ask anything"
              @keydown.enter.prevent="sendMessage"
            />

            <button
              class="rounded-full bg-primary-600 px-5 py-2 text-sm font-semibold text-white disabled:opacity-40"
              :disabled="loading || !input.trim()"
              @click="sendMessage"
            >
              {{ loading ? 'Sending' : 'Send' }}
            </button>
          </div>
        </div>

        <p class="mt-3 text-center text-xs text-gray-400 dark:text-gray-500">
          SecretAPI Chat can make mistakes. Check important information.
        </p>
      </div>
    </div>
    </div>
  </AppLayout>
</template>

<script setup lang="ts">
import { nextTick, ref } from 'vue'
import AppLayout from '@/components/layout/AppLayout.vue'
import AppLayout from '@/components/layout/AppLayout.vue'
import AppLayout from '@/components/layout/AppLayout.vue'

type ChatMessage = {
  role: 'user' | 'assistant'
  content: string
}

const input = ref('')
const loading = ref(false)
const model = ref('gpt-4.1-mini')
const chatBody = ref<HTMLElement | null>(null)

const messages = ref<ChatMessage[]>([])

const scrollToBottom = async () => {
  await nextTick()
  if (chatBody.value) {
    chatBody.value.scrollTop = chatBody.value.scrollHeight
  }
}

const clearChat = () => {
  messages.value = []
}

const sendMessage = async () => {
  const content = input.value.trim()
  if (!content || loading.value) return

  messages.value.push({
    role: 'user',
    content
  })

  input.value = ''
  loading.value = true
  await scrollToBottom()

  try {
    const res = await fetch('/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: 'Bearer YOUR_API_KEY'
      },
      body: JSON.stringify({
        model: model.value,
        messages: messages.value.map((msg) => ({
          role: msg.role,
          content: msg.content
        }))
      })
    })

    const data = await res.json()

    messages.value.push({
      role: 'assistant',
      content: data.choices?.[0]?.message?.content || data.error?.message || 'No response'
    })
  } catch (err) {
    messages.value.push({
      role: 'assistant',
      content: 'Request failed. Please check your API key or model configuration.'
    })
  } finally {
    loading.value = false
    await scrollToBottom()
  }
}
</script>
