<template>
  <div class="p-6">
    <div class="max-w-4xl mx-auto">
      <h1 class="text-3xl font-bold mb-6">SecretAPI Chat</h1>

      <div class="border rounded-2xl p-4 h-[500px] overflow-y-auto bg-white mb-4">
        <div
          v-for="(msg, index) in messages"
          :key="index"
          class="mb-4"
        >
          <div class="font-semibold">
            {{ msg.role }}
          </div>

          <div class="mt-1 text-gray-700">
            {{ msg.content }}
          </div>
        </div>
      </div>

      <div class="flex gap-2">
        <input
          v-model="input"
          class="flex-1 border rounded-xl px-4 py-3"
          placeholder="Ask anything..."
          @keyup.enter="sendMessage"
        />

        <button
          class="bg-black text-white px-6 rounded-xl"
          @click="sendMessage"
        >
          Send
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const input = ref('')

const messages = ref([
  {
    role: 'assistant',
    content: 'Welcome to SecretAPI Chat.'
  }
])

const sendMessage = async () => {
  if (!input.value) return

  messages.value.push({
    role: 'user',
    content: input.value
  })

  const userInput = input.value

  input.value = ''

  try {
    const res = await fetch('/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        Authorization: 'Bearer YOUR_API_KEY'
      },
      body: JSON.stringify({
        model: 'gpt-4.1-mini',
        messages: [
          {
            role: 'user',
            content: userInput
          }
        ]
      })
    })

    const data = await res.json()

    messages.value.push({
      role: 'assistant',
      content: data.choices?.[0]?.message?.content || 'No response'
    })
  } catch (err) {
    messages.value.push({
      role: 'assistant',
      content: 'Request failed.'
    })
  }
}
</script>
