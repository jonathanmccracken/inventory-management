<template>
  <div :class="['app', { 'sidebar-collapsed': collapsed }]">
    <aside :class="['sidebar', { 'is-collapsed': collapsed }]">
      <div class="sidebar-logo">
        <div class="logo-content">
          <h1>{{ t('nav.companyName') }}</h1>
          <span class="logo-sub">{{ t('nav.subtitle') }}</span>
        </div>
        <button class="sidebar-toggle" @click="collapsed = !collapsed" :title="collapsed ? 'Expand sidebar' : 'Collapse sidebar'">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <polyline v-if="!collapsed" points="15 18 9 12 15 6"/>
            <polyline v-else points="9 18 15 12 9 6"/>
          </svg>
        </button>
      </div>
      <nav class="sidebar-nav">
        <router-link to="/" :class="{ active: $route.path === '/' }">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="3" y="3" width="7" height="7"/><rect x="14" y="3" width="7" height="7"/>
            <rect x="14" y="14" width="7" height="7"/><rect x="3" y="14" width="7" height="7"/>
          </svg>
          <span class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>
        <router-link to="/inventory" :class="{ active: $route.path === '/inventory' }">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"/>
            <polyline points="3.27 6.96 12 12.01 20.73 6.96"/><line x1="12" y1="22.08" x2="12" y2="12"/>
          </svg>
          <span class="nav-label">{{ t('nav.inventory') }}</span>
        </router-link>
        <router-link to="/orders" :class="{ active: $route.path === '/orders' }">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <path d="M9 5H7a2 2 0 0 0-2 2v12a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V7a2 2 0 0 0-2-2h-2"/>
            <rect x="9" y="3" width="6" height="4" rx="1"/>
            <line x1="9" y1="12" x2="15" y2="12"/><line x1="9" y1="16" x2="12" y2="16"/>
          </svg>
          <span class="nav-label">{{ t('nav.orders') }}</span>
        </router-link>
        <router-link to="/spending" :class="{ active: $route.path === '/spending' }">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <rect x="1" y="4" width="22" height="16" rx="2" ry="2"/>
            <line x1="1" y1="10" x2="23" y2="10"/>
          </svg>
          <span class="nav-label">{{ t('nav.finance') }}</span>
        </router-link>
        <router-link to="/demand" :class="{ active: $route.path === '/demand' }">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <polyline points="23 6 13.5 15.5 8.5 10.5 1 18"/>
            <polyline points="17 6 23 6 23 12"/>
          </svg>
          <span class="nav-label">{{ t('nav.demandForecast') }}</span>
        </router-link>
        <router-link to="/reports" :class="{ active: $route.path === '/reports' }">
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
            <line x1="18" y1="20" x2="18" y2="10"/>
            <line x1="12" y1="20" x2="12" y2="4"/>
            <line x1="6" y1="20" x2="6" y2="14"/>
          </svg>
          <span class="nav-label">Reports</span>
        </router-link>
      </nav>
      <div class="sidebar-spacer" />
      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="main-wrapper">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />
  </div>
</template>

<script>
import { ref, onMounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)
    const collapsed = ref(false)
    const apiTasks = ref([])

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(loadTasks)

    return {
      t,
      collapsed,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask
    }
  }
}
</script>

<style>
:root {
  --sidebar-width: 240px;
  --sidebar-collapsed-width: 60px;
  --sidebar-bg: #0f172a;
  --sidebar-text: #94a3b8;
  --sidebar-text-active: #ffffff;
  --sidebar-hover-bg: rgba(255, 255, 255, 0.06);
  --sidebar-active-bg: rgba(37, 99, 235, 0.18);
  --sidebar-border: rgba(255, 255, 255, 0.08);
  --accent: #2563eb;
  --text: #0f172a;
  --text-secondary: #1e293b;
  --text-muted: #64748b;
  --text-table-th: #475569;
  --text-table-td: #334155;
  --border: #e2e8f0;
  --border-light: #f1f5f9;
  --surface: #ffffff;
  --background: #f8fafc;
  --shadow-sm: 0 1px 3px 0 rgba(0, 0, 0, 0.05);
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
  background: var(--background);
  color: var(--text-secondary);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ── Shell ── */
.app {
  display: grid;
  grid-template-columns: var(--sidebar-width) 1fr;
  min-height: 100vh;
  transition: grid-template-columns 0.2s ease;
}

.app.sidebar-collapsed {
  grid-template-columns: var(--sidebar-collapsed-width) 1fr;
}

/* ── Sidebar ── */
.sidebar {
  position: sticky;
  top: 0;
  height: 100vh;
  overflow-y: auto;
  background: var(--sidebar-bg);
  border-right: 1px solid var(--sidebar-border);
  display: flex;
  flex-direction: column;
  z-index: 100;
  width: var(--sidebar-width);
  transition: width 0.2s ease;
}

.sidebar.is-collapsed {
  width: var(--sidebar-collapsed-width);
}

.sidebar-logo {
  padding: 20px 16px 16px;
  border-bottom: 1px solid var(--sidebar-border);
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.logo-content {
  min-width: 0;
}

.sidebar-logo h1 {
  font-size: 1.0625rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: -0.025em;
  margin: 0;
}

.logo-sub {
  display: block;
  font-size: 0.75rem;
  color: var(--sidebar-text);
  margin-top: 3px;
  font-weight: 400;
}

.sidebar-nav {
  padding: 10px 8px;
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.sidebar-nav a {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 10px;
  border-radius: 6px;
  border-left: 2px solid transparent;
  color: var(--sidebar-text);
  font-size: 0.875rem;
  font-weight: 500;
  text-decoration: none;
  transition: background 0.15s ease, color 0.15s ease;
}

.sidebar-nav a:hover {
  background: var(--sidebar-hover-bg);
  color: var(--sidebar-text-active);
}

.sidebar-nav a.active,
.sidebar-nav a.router-link-active {
  background: var(--sidebar-active-bg);
  border-left-color: var(--accent);
  color: var(--sidebar-text-active);
}

.sidebar-nav svg {
  flex-shrink: 0;
  opacity: 0.8;
}

.sidebar-spacer {
  flex: 1;
}

.sidebar-footer {
  padding: 10px 8px 14px;
  border-top: 1px solid var(--sidebar-border);
  display: flex;
  flex-direction: column;
  gap: 6px;
}

/* ── Main content area ── */
.main-wrapper {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  overflow: hidden;
  background: var(--background);
}

.main-content {
  flex: 1;
  padding: 1.5rem 2rem;
}

/* ── Page layout ── */
.page-header {
  margin-bottom: 1.5rem;
}

.page-header h2 {
  font-size: 1.875rem;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 0.375rem;
  letter-spacing: -0.025em;
}

.page-header p {
  color: var(--text-muted);
  font-size: 0.938rem;
}

/* ── Stats & Cards ── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1.25rem;
  margin-bottom: 1.5rem;
}

.stat-card {
  background: var(--surface);
  padding: 1.25rem;
  border-radius: 10px;
  border: 1px solid var(--border);
  transition: all 0.2s ease;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.06);
}

.stat-label {
  color: var(--text-muted);
  font-size: 0.875rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.625rem;
}

.stat-value {
  font-size: 2.25rem;
  font-weight: 700;
  color: var(--text);
  letter-spacing: -0.025em;
}

.stat-card.warning .stat-value { color: #ea580c; }
.stat-card.success .stat-value { color: #059669; }
.stat-card.danger  .stat-value { color: #dc2626; }
.stat-card.info    .stat-value { color: var(--accent); }

.card {
  background: var(--surface);
  border-radius: 10px;
  padding: 1.25rem;
  border: 1px solid var(--border);
  margin-bottom: 1.25rem;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
  padding-bottom: 0.875rem;
  border-bottom: 1px solid var(--border);
}

.card-title {
  font-size: 1.125rem;
  font-weight: 700;
  color: var(--text);
  letter-spacing: -0.025em;
}

/* ── Tables ── */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: var(--background);
  border-top: 1px solid var(--border);
  border-bottom: 1px solid var(--border);
}

th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: var(--text-table-th);
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid var(--border-light);
  color: var(--text-table-td);
  font-size: 0.875rem;
}

tbody tr {
  transition: background-color 0.15s ease;
}

tbody tr:hover {
  background: var(--background);
}

/* ── Badges ── */
.badge {
  display: inline-block;
  padding: 0.313rem 0.75rem;
  border-radius: 6px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.025em;
}

.badge.success    { background: #d1fae5; color: #065f46; }
.badge.warning    { background: #fed7aa; color: #92400e; }
.badge.danger     { background: #fecaca; color: #991b1b; }
.badge.info       { background: #dbeafe; color: #1e40af; }
.badge.increasing { background: #d1fae5; color: #065f46; }
.badge.decreasing { background: #fecaca; color: #991b1b; }
.badge.stable     { background: #e0e7ff; color: #3730a3; }
.badge.high       { background: #fecaca; color: #991b1b; }
.badge.medium     { background: #fed7aa; color: #92400e; }
.badge.low        { background: #dbeafe; color: #1e40af; }

/* ── States ── */
.loading {
  text-align: center;
  padding: 3rem;
  color: var(--text-muted);
  font-size: 0.938rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 1rem;
  border-radius: 8px;
  margin: 1rem 0;
  font-size: 0.938rem;
}

/* ── Sidebar toggle button ── */
.sidebar-toggle {
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  height: 28px;
  border: none;
  background: transparent;
  color: var(--sidebar-text);
  border-radius: 5px;
  cursor: pointer;
  transition: background 0.15s, color 0.15s;
}

.sidebar-toggle:hover {
  background: var(--sidebar-hover-bg);
  color: var(--sidebar-text-active);
}

/* ── Collapsed sidebar states ── */
.sidebar.is-collapsed .nav-label,
.sidebar.is-collapsed .logo-sub {
  display: none;
}

.sidebar.is-collapsed .sidebar-logo h1 {
  display: none;
}

.sidebar.is-collapsed .logo-content {
  display: none;
}

.sidebar.is-collapsed .sidebar-nav a {
  justify-content: center;
  padding: 10px;
  gap: 0;
}

.sidebar.is-collapsed .sidebar-footer {
  align-items: center;
  padding-left: 0;
  padding-right: 0;
}

.sidebar.is-collapsed .profile-button {
  justify-content: center;
  padding: 8px;
}

.sidebar.is-collapsed .profile-name,
.sidebar.is-collapsed .profile-chevron {
  display: none;
}

/* ── Mobile ── */
@media (max-width: 768px) {
  .app {
    grid-template-columns: 1fr;
  }
  .sidebar {
    display: none;
  }
  .main-wrapper {
    min-height: 100vh;
  }
}
</style>
