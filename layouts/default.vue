<script setup lang="ts">
// Simple language state
const locale = ref<'th' | 'en'>('th')

const messages = {
  th: {
    nav: {
      dashboard: "แดชบอร์ด",
      bookings: "การจอง",
      vehicles: "ยานพาหนะ",
      users: "ผู้ใช้งาน",
      settings: "การตั้งค่า",
      collapse: "ยุบเมนู"
    },
    header: {
      title: "ระบบจองรถ",
      logout: "ออกจากระบบ"
    }
  },
  en: {
    nav: {
      dashboard: "Dashboard",
      bookings: "Bookings",
      vehicles: "Vehicles",
      users: "Users",
      settings: "Settings",
      collapse: "Collapse Menu"
    },
    header: {
      title: "Car Booking System",
      logout: "Logout"
    }
  }
}

const t = (key: string): string => {
  const keys = key.split('.')
  let value: any = messages[locale.value]

  for (const k of keys) {
    value = value?.[k]
  }

  return value || key
}

const switchLanguage = () => {
  locale.value = locale.value === 'th' ? 'en' : 'th'
}

const navItems = computed(() => [
  { label: t('nav.dashboard'), icon: "i-heroicons-home", to: "/" },
  { label: t('nav.bookings'), icon: "i-heroicons-calendar", to: "/bookings" },
  { label: t('nav.vehicles'), icon: "i-heroicons-truck", to: "/vehicles" },
  { label: t('nav.users'), icon: "i-heroicons-user-group", to: "/users" },
  { label: t('nav.settings'), icon: "i-heroicons-cog-6-tooth", to: "/settings" },
])

// State for sidebar
const isSidebarOpen = ref(true)
const isMobile = ref(false)

// Check if mobile on mount and resize
onMounted(() => {
  const checkMobile = () => {
    isMobile.value = window.innerWidth < 768
    if (isMobile.value) {
      isSidebarOpen.value = false
    }
  }

  checkMobile()
  window.addEventListener('resize', checkMobile)

  onUnmounted(() => {
    window.removeEventListener('resize', checkMobile)
  })
})

const toggleSidebar = () => {
  isSidebarOpen.value = !isSidebarOpen.value
}

const closeSidebarOnMobile = () => {
  if (isMobile.value) {
    isSidebarOpen.value = false
  }
}
</script>

<template>
  <div class="flex h-screen bg-gray-50 relative">
    <!-- Mobile Overlay -->
    <div
      v-if="isMobile && isSidebarOpen"
      class="fixed inset-0 bg-black bg-opacity-50 z-40 md:hidden"
      @click="closeSidebarOnMobile"
    ></div>

    <!-- Sidebar -->
    <aside
      :class="[
        'bg-white border-r border-gray-200 shadow-sm transition-all duration-300 z-50',
        isMobile ? 'fixed left-0 top-0 h-full' : 'relative',
        isSidebarOpen ? (isMobile ? 'w-64' : 'w-64') : 'w-16',
        !isSidebarOpen && isMobile ? '-translate-x-full' : 'translate-x-0'
      ]"
    >
      <div class="p-4">
        <!-- Logo/Brand -->
        <div class="flex items-center mb-8">
          <div class="text-2xl flex-shrink-0">🚗</div>
          <div
            v-show="isSidebarOpen"
            class="ml-3 text-xl font-bold text-gray-800 transition-all duration-300 overflow-hidden"
          >
            CarBooking
          </div>
        </div>

        <!-- Toggle Button (Desktop) -->
        <div v-if="!isMobile" class="mb-6">
          <UButton
            @click="toggleSidebar"
            size="sm"
            color="gray"
            variant="ghost"
            :class="[
              'w-full transition-all duration-300',
              isSidebarOpen ? 'justify-start' : 'justify-center'
            ]"
          >
            <UIcon
              :name="isSidebarOpen ? 'i-heroicons-chevron-left' : 'i-heroicons-chevron-right'"
              class="w-4 h-4 flex-shrink-0"
            />
            <span
              v-show="isSidebarOpen"
              class="ml-2 transition-all duration-300 overflow-hidden"
            >
              {{ t('nav.collapse') }}
            </span>
          </UButton>
        </div>

        <!-- Navigation Menu -->
        <nav class="space-y-2">
          <NuxtLink
            v-for="item in navItems"
            :key="item.to"
            :to="item.to"
            @click="closeSidebarOnMobile"
            :class="[
              'flex items-center px-3 py-2 text-gray-700 rounded-lg hover:bg-gray-100 transition-colors group relative',
              !isSidebarOpen ? 'justify-center' : 'gap-3'
            ]"
            active-class="bg-blue-50 text-blue-700 border-r-2 border-blue-700"
          >
            <UIcon :name="item.icon" class="w-5 h-5 flex-shrink-0" />
            <span
              v-show="isSidebarOpen"
              class="font-medium transition-opacity duration-300"
            >
              {{ item.label }}
            </span>
            <!-- Tooltip for collapsed state (Desktop only) -->
            <div
              v-if="!isSidebarOpen && !isMobile"
              class="absolute left-16 bg-gray-800 text-white px-2 py-1 rounded text-sm opacity-0 group-hover:opacity-100 transition-opacity duration-200 pointer-events-none z-10 whitespace-nowrap"
            >
              {{ item.label }}
            </div>
          </NuxtLink>
        </nav>
      </div>
    </aside>

    <!-- Main Content -->
    <div class="flex flex-col flex-1 min-w-0">
      <!-- Topbar -->
      <header class="flex items-center justify-between px-4 md:px-6 py-4 bg-white border-b border-gray-200 shadow-sm">
        <div class="flex items-center gap-4">
          <!-- Mobile Menu Button -->
          <UButton
            v-if="isMobile"
            @click="toggleSidebar"
            size="sm"
            color="gray"
            variant="ghost"
            class="md:hidden"
          >
            <UIcon name="i-heroicons-bars-3" class="w-5 h-5" />
          </UButton>

          <h1 class="text-lg md:text-xl font-semibold text-gray-800">{{ t('header.title') }}</h1>
        </div>

        <div class="flex items-center gap-4">
          <!-- Language Switcher -->
          <UButton
            @click="switchLanguage(locale === 'th' ? 'en' : 'th')"
            size="sm"
            color="gray"
            variant="ghost"
            class="gap-2"
          >
            <UIcon name="i-heroicons-language" class="w-4 h-4" />
            <span class="hidden sm:inline">{{ locale === 'th' ? 'ไทย' : 'English' }}</span>
          </UButton>

          <UAvatar src="https://i.pravatar.cc/40" alt="User" size="sm" />

          <UButton size="sm" color="gray" variant="ghost" class="hidden sm:flex">
            <UIcon name="i-heroicons-arrow-right-on-rectangle" class="w-4 h-4 mr-2" />
            {{ t('header.logout') }}
          </UButton>
          <!-- Mobile logout button (icon only) -->
          <UButton size="sm" color="gray" variant="ghost" class="sm:hidden">
            <UIcon name="i-heroicons-arrow-right-on-rectangle" class="w-4 h-4" />
          </UButton>
        </div>
      </header>

      <!-- Page Content -->
      <main class="flex-1 p-4 md:p-6 overflow-y-auto">
        <slot />
      </main>
    </div>
  </div>
</template>
