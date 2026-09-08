<template>
  <aside class="sidebar" :class="{ collapsed }">
    <div class="logo-row">
      <div class="logo" v-show="!collapsed">
        <h1>{{ t('nav.companyName') }}</h1>
        <span class="subtitle">{{ t('nav.subtitle') }}</span>
      </div>
      <div class="logo-mark" v-show="collapsed" :title="t('nav.companyName')">
        {{ t('nav.companyName').charAt(0) }}
      </div>
      <button
        class="collapse-toggle"
        type="button"
        :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'"
        @click="toggleCollapsed"
      >
        <svg width="16" height="16" viewBox="0 0 16 16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" :class="{ flipped: collapsed }">
          <path d="M10 3L5 8L10 13"/>
        </svg>
      </button>
    </div>

    <nav class="nav-tabs">
      <router-link to="/" class="nav-link" :class="{ active: $route.path === '/' }" :title="t('nav.overview')">
        <span class="nav-icon">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="2" y="2" width="7" height="7" rx="1"/>
            <rect x="11" y="2" width="7" height="7" rx="1"/>
            <rect x="2" y="11" width="7" height="7" rx="1"/>
            <rect x="11" y="11" width="7" height="7" rx="1"/>
          </svg>
        </span>
        <span class="nav-label" v-show="!collapsed">{{ t('nav.overview') }}</span>
      </router-link>
      <router-link to="/inventory" class="nav-link" :class="{ active: $route.path === '/inventory' }" :title="t('nav.inventory')">
        <span class="nav-icon">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M2 6L10 2L18 6L10 10L2 6Z"/>
            <path d="M2 6V14L10 18L18 14V6"/>
            <path d="M10 10V18"/>
          </svg>
        </span>
        <span class="nav-label" v-show="!collapsed">{{ t('nav.inventory') }}</span>
      </router-link>
      <router-link to="/orders" class="nav-link" :class="{ active: $route.path === '/orders' }" :title="t('nav.orders')">
        <span class="nav-icon">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M7 2H13C13.5523 2 14 2.44772 14 3V4H6V3C6 2.44772 6.44772 2 7 2Z"/>
            <rect x="4" y="4" width="12" height="14" rx="1"/>
            <path d="M7 9H13"/>
            <path d="M7 12H13"/>
            <path d="M7 15H10"/>
          </svg>
        </span>
        <span class="nav-label" v-show="!collapsed">{{ t('nav.orders') }}</span>
      </router-link>
      <router-link to="/spending" class="nav-link" :class="{ active: $route.path === '/spending' }" :title="t('nav.finance')">
        <span class="nav-icon">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <circle cx="10" cy="10" r="8"/>
            <path d="M13 7.5C13 6.5 11.8 6 10 6C8.2 6 7 6.7 7 7.8C7 8.9 8.2 9.3 10 9.7C11.8 10.1 13 10.6 13 11.9C13 13 11.8 13.8 10 13.8C8.2 13.8 7 13.2 7 12.2"/>
            <path d="M10 4.5V6"/>
            <path d="M10 13.8V15.5"/>
          </svg>
        </span>
        <span class="nav-label" v-show="!collapsed">{{ t('nav.finance') }}</span>
      </router-link>
      <router-link to="/demand" class="nav-link" :class="{ active: $route.path === '/demand' }" :title="t('nav.demandForecast')">
        <span class="nav-icon">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M2 15L7 9L11 12L18 4"/>
            <path d="M13 4H18V9"/>
          </svg>
        </span>
        <span class="nav-label" v-show="!collapsed">{{ t('nav.demandForecast') }}</span>
      </router-link>
      <router-link to="/restocking" class="nav-link" :class="{ active: $route.path === '/restocking' }" :title="t('nav.restocking')">
        <span class="nav-icon">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M17 10C17 13.866 13.866 17 10 17C6.13401 17 3 13.866 3 10C3 6.13401 6.13401 3 10 3C12.5264 3 14.7452 4.33902 16 6.30622"/>
            <path d="M16 2.5V6.5H12"/>
          </svg>
        </span>
        <span class="nav-label" v-show="!collapsed">{{ t('nav.restocking') }}</span>
      </router-link>
      <router-link to="/reports" class="nav-link" :class="{ active: $route.path === '/reports' }" title="Reports">
        <span class="nav-icon">
          <svg width="20" height="20" viewBox="0 0 20 20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M4 17V11"/>
            <path d="M10 17V4"/>
            <path d="M16 17V8"/>
          </svg>
        </span>
        <span class="nav-label" v-show="!collapsed">Reports</span>
      </router-link>
    </nav>

    <div class="sidebar-footer">
      <LanguageSwitcher />
      <ProfileMenu
        @show-profile-details="$emit('show-profile-details')"
        @show-tasks="$emit('show-tasks')"
      />
    </div>
  </aside>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { useI18n } from '../composables/useI18n'
import ProfileMenu from './ProfileMenu.vue'
import LanguageSwitcher from './LanguageSwitcher.vue'

defineEmits(['show-profile-details', 'show-tasks'])

const { t } = useI18n()

const STORAGE_KEY = 'sidebar-collapsed'
const BREAKPOINT = 1280

const collapsed = ref(false)
// Tracks whether the user has manually toggled the sidebar since the last
// auto-decision (i.e. since the viewport last crossed the breakpoint).
// A manual choice wins over the responsive default until the breakpoint
// is crossed again, at which point auto-behavior resumes.
let manualOverride = false
let lastSide = null
let resizeTimer = null

const isSmallViewport = () => window.innerWidth < BREAKPOINT

const persist = () => {
  localStorage.setItem(STORAGE_KEY, String(collapsed.value))
}

const applyResponsiveDefault = () => {
  const side = isSmallViewport() ? 'small' : 'large'
  if (side !== lastSide) {
    // Breakpoint crossed - auto behavior resumes.
    manualOverride = false
    lastSide = side
  }
  if (!manualOverride) {
    collapsed.value = side === 'small'
    persist()
  }
}

const handleResize = () => {
  clearTimeout(resizeTimer)
  resizeTimer = setTimeout(applyResponsiveDefault, 200)
}

const toggleCollapsed = () => {
  collapsed.value = !collapsed.value
  manualOverride = true
  persist()
}

onMounted(() => {
  const stored = localStorage.getItem(STORAGE_KEY)
  lastSide = isSmallViewport() ? 'small' : 'large'

  if (stored !== null) {
    collapsed.value = stored === 'true'
  } else {
    collapsed.value = lastSide === 'small'
    persist()
  }

  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
  clearTimeout(resizeTimer)
})
</script>

<style scoped>
.sidebar {
  width: 260px;
  flex-shrink: 0;
  background: #ffffff;
  border-right: 1px solid #e2e8f0;
  position: sticky;
  top: 0;
  height: 100vh;
  display: flex;
  flex-direction: column;
  padding: var(--space-6) var(--space-4) var(--space-4);
  transition: width 0.2s ease;
  /* No overflow set here (stays "visible") so the ProfileMenu/LanguageSwitcher
     dropdowns - which are position:absolute and wider than the sidebar,
     especially when collapsed - aren't clipped. Per the CSS overflow spec,
     setting overflow-y here would force overflow-x to compute as "auto" too
     and clip those dropdowns; scrolling is instead scoped to .nav-tabs below. */
}

.sidebar.collapsed {
  width: 72px;
  padding: var(--space-6) var(--space-2) var(--space-4);
}

.nav-tabs {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  flex: 1 1 auto;
  min-height: 0;
  overflow-y: auto;
  overflow-x: hidden;
}

.logo-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: var(--space-2);
  padding: 0 var(--space-3) var(--space-6);
}

.sidebar.collapsed .logo-row {
  flex-direction: column;
  gap: var(--space-3);
  padding: 0 0 var(--space-6);
}

.logo {
  display: flex;
  flex-direction: column;
  gap: var(--space-1);
  min-width: 0;
}

.logo h1 {
  font-size: 1.375rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
  white-space: nowrap;
}

.subtitle {
  font-size: 0.813rem;
  color: #64748b;
  font-weight: 400;
  white-space: nowrap;
}

.logo-mark {
  width: 32px;
  height: 32px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  background: #eff6ff;
  color: #2563eb;
  font-weight: 700;
  font-size: 1rem;
}

.collapse-toggle {
  flex-shrink: 0;
  width: 28px;
  height: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: none;
  border: 1px solid #e2e8f0;
  border-radius: 6px;
  color: #64748b;
  cursor: pointer;
  transition: all 0.2s ease;
}

.collapse-toggle:hover {
  background: #f1f5f9;
  color: #0f172a;
}

.collapse-toggle svg {
  transition: transform 0.2s ease;
}

.collapse-toggle svg.flipped {
  transform: rotate(180deg);
}

.nav-link {
  display: flex;
  align-items: center;
  gap: var(--space-3);
  padding: 0.625rem var(--space-3);
  color: #64748b;
  text-decoration: none;
  font-weight: 500;
  font-size: 0.938rem;
  border-radius: 6px;
  transition: all 0.2s ease;
  position: relative;
}

.sidebar.collapsed .nav-link {
  justify-content: center;
  padding: 0.625rem;
}

.nav-label {
  white-space: nowrap;
  overflow: hidden;
}

.nav-icon {
  width: 20px;
  height: 20px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.nav-link:hover {
  color: #0f172a;
  background: #f1f5f9;
}

.nav-link.active {
  color: #2563eb;
  background: #eff6ff;
}

.nav-link.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  width: 3px;
  background: #2563eb;
}

.sidebar-footer {
  margin-top: auto;
  display: flex;
  flex-direction: column;
  gap: var(--space-3);
  padding-top: var(--space-4);
}

.sidebar-footer :deep(.language-switcher),
.sidebar-footer :deep(.profile-menu) {
  width: 100%;
}

.sidebar-footer :deep(.language-button),
.sidebar-footer :deep(.profile-button) {
  width: 100%;
}

/* Collapsed footer: shrink language/profile buttons to icon-only,
   and flip their dropdowns to open beside the sidebar (flyout) instead
   of right-aligned, since right:0 would push a 160-280px wide dropdown
   off the left edge of the viewport once the sidebar is only ~72px wide. */
.sidebar.collapsed .sidebar-footer :deep(.language-switcher),
.sidebar.collapsed .sidebar-footer :deep(.profile-menu) {
  width: auto;
}

.sidebar.collapsed .sidebar-footer :deep(.language-button),
.sidebar.collapsed .sidebar-footer :deep(.profile-button) {
  width: auto;
  justify-content: center;
  padding: 0.5rem;
  gap: 0;
}

.sidebar.collapsed .sidebar-footer :deep(.language-label),
.sidebar.collapsed .sidebar-footer :deep(.profile-name),
.sidebar.collapsed .sidebar-footer :deep(.chevron) {
  display: none;
}

.sidebar.collapsed .sidebar-footer :deep(.dropdown-menu) {
  left: 100%;
  right: auto;
  top: 0;
  margin-left: var(--space-2);
}
</style>
