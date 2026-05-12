<template>
  <div class="h-[calc(100vh-4rem)] bg-slate-50 px-6 py-6">
    <div class="mx-auto flex h-full max-w-7xl flex-col rounded-[28px] bg-[#050505] text-white shadow-2xl">
      <div class="flex items-center justify-between border-b border-white/10 px-6 py-4">
        <div>
          <h1 class="text-lg font-semibold">SecretAPI Chat</h1>
          <p class="text-xs text-white/45">Chat with your API models</p>
        </div>

        <select
          v-model="model"
          class="rounded-full border border-white/10 bg-white/10 px-4 py-2 text-sm text-white outline-none"
        >
          <option class="text-black" value="gpt-4.1-mini">gpt-4.1-mini</option>
          <option class="text-black" value="gpt-4.1">gpt-4.1</option>
          <option class="text-black" value="gpt-4o-mini">gpt-4o-mini</option>
        </select>
      </div>

      <div ref="chatBody" class="flex-1 overflow-y-auto px-6 py-8">
        <div v-if="messages.length === 0" class="flex h-full flex-col items-center justify-center text-center">
          <div class="mb-6 flex h-14 w-14 items-center justify-center rounded-2xl bg-white text-xl font-bold text-black">
            S
          </div>
          <h2 class="text-3xl font-semibold tracking-tight">What can I help with?</h2>
          <p class="mt-3 max-w-md text-sm text-white/45">
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
                  ? 'bg-white text-black'
                  : 'bg-zinc-900 text-white border border-white/10'
              ]"
            >
              <div class="mb-1 text-xs font-semibold opacity-50">
                {{ msg.role === 'user' ? 'You' : 'SecretAPI' }}
              </div>
              <div class="whitespace-pre-wrap">{{ msg.content }}</div>
            </div>
          </div>

          <div v-if="loading" class="flex justify-start">
            <div class="rounded-3xl border border-white/10 bg-zinc-900 px-5 py-4 text-sm text-white/60">
              Thinking...
            </div>
          </div>
        </div>
      </div>

      <div class="px-6 pb-6">
        <div class="mx-auto max-w-3xl rounded-[32px] bg-white p-2 shadow-xl">
          <div class="flex items-center gap-3 rounded-[26px] bg-black px-4 py-3">
            <button
              class="flex h-8 w-8 items-center justify-center rounded-full border border-white/15 text-xl text-white/70"
              type="button"
              @click="clearChat"
              title="New chat"
            >
              +
            </button>

            <textarea
              v-model="input"
              rows="1"
              class="max-h-32 flex-1 resize-none bg-transparent text-sm text-white outline-none placeholder:text-white/45"
              placeholder="Ask anything"
              @keydown.enter.prevent="sendMessage"
            />

            <button
              class="rounded-full bg-white px-5 py-2 text-sm font-semibold text-black disabled:opacity-40"
              :disabled="loading || !input.trim()"
              @click="sendMessage"
            >
              {{ loading ? 'Sending' : 'Send' }}
            </button>
          </div>
        </div>

        <p class="mt-3 text-center text-xs text-white/30">
          SecretAPI Chat can make mistakes. Check important information.
        </p>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { nextTick, ref } from 'vue'

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
