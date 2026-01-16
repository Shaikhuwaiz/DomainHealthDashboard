<template>
  <div class="min-h-screen bg-gray-50 p-6">
    <div class="mx-auto max-w-6xl space-y-4">
      <h1 class="text-2xl font-bold">Domain Health Dashboard</h1>

      <!-- Top Controls -->
      <div
        class="flex flex-col gap-3 sm:flex-row sm:items-center sm:justify-between"
      >
        <input
          v-model="q"
          placeholder="Search domain..."
          class="w-full rounded-lg border bg-white px-3 py-2 text-sm outline-none focus:ring-2"
        />

        <div
          class="flex w-full flex-col gap-2 sm:w-auto sm:flex-row sm:items-center"
        >
          <select
            v-model="healthFilter"
            class="w-full rounded-lg border bg-white px-3 py-2 text-sm sm:w-48"
          >
            <option value="all">All</option>
            <option value="good">Good</option>
            <option value="warning">Warning</option>
            <option value="bad">Bad</option>
          </select>

          <button
            @click="refreshAll"
            class="w-full rounded-lg bg-black px-4 py-2 text-sm text-white hover:opacity-90 sm:w-auto"
          >
            Refresh All
          </button>

          <button
            @click="apiDown = !apiDown"
            class="w-full rounded-lg border bg-white px-3 py-2 text-xs hover:bg-gray-50 sm:w-auto"
          >
            Toggle API Down
          </button>
        </div>
      </div>

      <!-- Error banner -->
      <div
        v-if="apiDown"
        class="rounded-xl border border-red-200 bg-red-50 p-3 text-sm text-red-700"
      >
        Instantly API is down. Showing last cached data.
      </div>

      <!-- Refresh status -->
      <p v-if="refreshStatus" class="text-xs text-gray-600">
        {{ refreshStatus }}
      </p>

      <!-- Table -->
      <div class="overflow-hidden rounded-xl border bg-white">
        <table class="w-full text-left text-sm">
          <thead class="bg-gray-100">
            <tr>
              <th class="px-4 py-3">Domain</th>
              <th class="px-4 py-3">Inboxes</th>
              <th class="px-4 py-3">Reply %</th>
              <th class="px-4 py-3">Bounce %</th>
              <th class="px-4 py-3">Health</th>
              <th class="px-4 py-3">Last Updated</th>
            </tr>
          </thead>

          <!-- ✅ Normal rows -->
          <tbody v-if="!loading">
            <tr
              v-for="d in paginatedDomains"
              :key="d.domain"
              class="border-t hover:bg-gray-50"
            >
              <td class="px-4 py-3">
                <NuxtLink
                  :to="`/domain/${encodeURIComponent(d.domain)}`"
                  class="font-medium text-blue-600 hover:underline"
                >
                  {{ d.domain }}
                </NuxtLink>
              </td>
              <td class="px-4 py-3">{{ d.inboxCount }}</td>
              <td class="px-4 py-3">{{ d.replyRate.toFixed(2) }}</td>
              <td class="px-4 py-3">{{ d.bounceRate.toFixed(2) }}</td>
              <td class="px-4 py-3">
                <span
                  class="rounded-full px-2 py-1 text-xs font-semibold"
                  :class="badgeClass(d.health)"
                >
                  {{ d.health }}
                </span>
              </td>
              <td class="px-4 py-3 text-gray-600">{{ d.lastUpdated }}</td>
            </tr>

            <!-- Empty state -->
            <tr v-if="filteredDomains.length === 0">
              <td colspan="6" class="px-4 py-6 text-center text-gray-500">
                No domains found.
              </td>
            </tr>
          </tbody>

          <!-- ✅ Loading skeleton -->
          <tbody v-else>
            <tr v-for="n in 5" :key="n" class="border-t">
              <td class="px-4 py-3" colspan="6">
                <div
                  class="h-4 w-full animate-pulse rounded bg-gray-200"
                ></div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- ✅ Pagination UI -->
      <div class="flex items-center justify-between text-sm text-gray-600">
        <span>
          Showing {{ paginatedDomains.length }} of {{ filteredDomains.length }}
          domains (Page {{ page }} / {{ totalPages }})
        </span>

        <div class="flex gap-2">
          <button
            @click="prevPage"
            :disabled="page === 1"
            class="rounded border bg-white px-3 py-1 hover:bg-gray-50 disabled:opacity-50"
          >
            Prev
          </button>

          <button
            @click="nextPage"
            :disabled="page === totalPages"
            class="rounded border bg-white px-3 py-1 hover:bg-gray-50 disabled:opacity-50"
          >
            Next
          </button>
        </div>
      </div>

      <p class="text-xs text-gray-500">
        Demo UI with mock data
      </p>
    </div>
  </div>
</template>

<script setup lang="ts">
type Health = "good" | "warning" | "bad"

type DomainRow = {
  domain: string
  inboxCount: number
  replyRate: number
  bounceRate: number
  health: Health
  lastUpdated: string
}

const q = ref("")
const healthFilter = ref<"all" | Health>("all")

const loading = ref(false)
const apiDown = ref(false)
const refreshStatus = ref("")

// ✅ pagination
const page = ref(1)
const perPage = 50

function randomHealth(): Health {
  const r = Math.random()
  if (r < 0.6) return "good"
  if (r < 0.85) return "warning"
  return "bad"
}

function randomUpdated(): string {
  const list = ["just now", "2 min ago", "12 min ago", "1 hr ago", "2 hr ago"]
  return list[Math.floor(Math.random() * list.length)] ?? "just now"
}

// ✅ Generate 150 domains (looks realistic)
const domains = ref<DomainRow[]>(
  Array.from({ length: 150 }).map((_, idx) => {
    const health = randomHealth()

    const replyRate =
      health === "good"
        ? 6 + Math.random() * 4
        : health === "warning"
        ? 2 + Math.random() * 3
        : 0.5 + Math.random() * 1.5

    const bounceRate =
      health === "good"
        ? 0.5 + Math.random() * 1.0
        : health === "warning"
        ? 1.5 + Math.random() * 2.5
        : 4 + Math.random() * 4

    return {
      domain: idx === 0 ? "talaria.co.in" : `domain-${idx + 1}.com`,
      inboxCount: 20 + Math.floor(Math.random() * 120),
      replyRate,
      bounceRate,
      health,
      lastUpdated: randomUpdated(),
    }
  })
)

const filteredDomains = computed(() => {
  const search = q.value.toLowerCase().trim()

  return domains.value.filter((d) => {
    const matchesSearch = d.domain.toLowerCase().includes(search)
    const matchesHealth =
      healthFilter.value === "all" ? true : d.health === healthFilter.value
    return matchesSearch && matchesHealth
  })
})

const totalPages = computed(() =>
  Math.max(1, Math.ceil(filteredDomains.value.length / perPage))
)

const paginatedDomains = computed(() => {
  const start = (page.value - 1) * perPage
  return filteredDomains.value.slice(start, start + perPage)
})

// reset page when filter/search changes
watch([q, healthFilter], () => {
  page.value = 1
})

function prevPage() {
  if (page.value > 1) page.value--
}

function nextPage() {
  if (page.value < totalPages.value) page.value++
}

function refreshAll() {
  loading.value = true
  refreshStatus.value = "Refreshing... (mock 2 seconds)"

  setTimeout(() => {
    loading.value = false
    refreshStatus.value = "Updated just now "

    domains.value = domains.value.map((d) => ({
      ...d,
      lastUpdated: "just now",
    }))
  }, 2000)
}

function badgeClass(health: Health) {
  if (health === "good") return "bg-green-100 text-green-700"
  if (health === "warning") return "bg-yellow-100 text-yellow-800"
  return "bg-red-100 text-red-700"
}
</script>
