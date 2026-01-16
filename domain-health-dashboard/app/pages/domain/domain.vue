<template>
  <div class="min-h-screen bg-gray-50 p-6">
    <div class="mx-auto max-w-5xl space-y-4">
      <div class="flex items-center justify-between">
        <div>
          <h1 class="text-2xl font-bold">{{ domain }}</h1>
          <p class="text-sm text-gray-500">Inbox-level stats (mock)</p>
        </div>

        <NuxtLink to="/" class="rounded-lg border bg-white px-3 py-2 text-sm hover:bg-gray-50">
          ← Back
        </NuxtLink>
      </div>

      <div class="overflow-hidden rounded-xl border bg-white">
        <table class="w-full text-left text-sm">
          <thead class="bg-gray-100">
            <tr>
              <th class="px-4 py-3">Inbox</th>
              <th class="px-4 py-3">Sent</th>
              <th class="px-4 py-3">Replies</th>
              <th class="px-4 py-3">Reply %</th>
              <th class="px-4 py-3">Bounce %</th>
              <th class="px-4 py-3">Health</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="i in inboxes" :key="i.email" class="border-t">
              <td class="px-4 py-3 font-medium">{{ i.email }}</td>
              <td class="px-4 py-3">{{ i.sent }}</td>
              <td class="px-4 py-3">{{ i.replies }}</td>
              <td class="px-4 py-3">{{ i.replyRate.toFixed(2) }}</td>
              <td class="px-4 py-3">{{ i.bounceRate.toFixed(2) }}</td>
              <td class="px-4 py-3">
                <span class="rounded-full px-2 py-1 text-xs font-semibold"
                      :class="badgeClass(i.health)">
                  {{ i.health }}
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
type Health = 'good' | 'warning' | 'bad'

const route = useRoute()
const domain = computed(() => String(route.params.domain || ''))

const inboxes = ref([
  { email: `sales@${domain.value}`, sent: 1200, replies: 90, replyRate: 7.5, bounceRate: 1.2, health: 'good' as Health },
  { email: `info@${domain.value}`, sent: 800, replies: 18, replyRate: 2.25, bounceRate: 3.4, health: 'warning' as Health },
  { email: `support@${domain.value}`, sent: 600, replies: 5, replyRate: 0.83, bounceRate: 6.1, health: 'bad' as Health },
])

function badgeClass(health: Health) {
  if (health === 'good') return 'bg-green-100 text-green-700'
  if (health === 'warning') return 'bg-yellow-100 text-yellow-800'
  return 'bg-red-100 text-red-700'
}
</script>
