<template>
  <div class="changelogs">
    <!-- FILTERS PANEL -->
    <div class="filters-panel">
      <div class="search-container">
        <div class="search-field">
          <span class="search-glass">🔍</span>
          <input 
            type="text" 
            v-model="search" 
            placeholder="Search commits..."
          >
          <span v-if="search" class="search-clear" @click="search = ''">✕</span>
        </div>
        
        <div class="search-hints">
          <button class="hint-chip" @click="search = 'feature'">✨ feature</button>
          <button class="hint-chip" @click="search = 'fix'">🔧 fix</button>
          <button class="hint-chip" @click="search = 'api'">⚡ api</button>
          <button class="hint-chip" @click="search = 'docs'">📚 docs</button>
        </div>
      </div>

      <div class="filters-row">
        <div class="sort-group">
          <button 
            class="sort-pill" 
            :class="{ active: sortOrder === 'desc' }"
            @click="sortOrder = 'desc'"
          >
            <span class="pill-icon">⬇️</span>
            <span class="pill-text">Newest</span>
          </button>
          <button 
            class="sort-pill" 
            :class="{ active: sortOrder === 'asc' }"
            @click="sortOrder = 'asc'"
          >
            <span class="pill-icon">⬆️</span>
            <span class="pill-text">Oldest</span>
          </button>
        </div>

        <div class="tags-group">
          <PageTagGreen 
            :class="{ 'tag-active': tagFilter === 'feature' }" 
            @click="tagFilter = tagFilter === 'feature' ? 'all' : 'feature'"
          >✨</PageTagGreen>
          <PageTagRed 
            :class="{ 'tag-active': tagFilter === 'fix' }" 
            @click="tagFilter = tagFilter === 'fix' ? 'all' : 'fix'"
          >🔧</PageTagRed>
          <PageTagBlue 
            :class="{ 'tag-active': tagFilter === 'docs' }" 
            @click="tagFilter = tagFilter === 'docs' ? 'all' : 'docs'"
          >📚</PageTagBlue>
          <PageTagOrange 
            :class="{ 'tag-active': tagFilter === 'update' }" 
            @click="tagFilter = tagFilter === 'update' ? 'all' : 'update'"
          >🔄</PageTagOrange>
        </div>

        <button class="reset-pill" @click="resetFilters">
          <span class="pill-icon">↺</span>
          <span class="pill-text">Reset</span>
        </button>

        <div class="count-badge">
          <span class="count-number">{{ filteredCommits.length }}</span>
          <span class="count-label">commits</span>
        </div>
      </div>

      <transition name="fade-slide">
        <div class="active-filters" v-if="search || tagFilter !== 'all'">
          <div v-if="search" class="filter-tag">
            <span>🔍 {{ search }}</span>
            <button @click="search = ''">✕</button>
          </div>
          <div v-if="tagFilter !== 'all'" class="filter-tag">
            <span>🏷️ {{ tagFilter }}</span>
            <button @click="tagFilter = 'all'">✕</button>
          </div>
        </div>
      </transition>
    </div>

    <!-- COMMITS LIST -->
    <div class="commits-flow">
      <transition-group name="list" tag="div" class="commits-grid">
        <div v-for="(commit, idx) in filteredCommits" :key="commit.id" 
             class="commit-module">
          
          <div class="glass-card">
            <!-- HEADER -->
            <div class="module-header">
              <div class="header-breadcrumbs">
                <span class="breadcrumb-hash">{{ commit.hash.substring(0, 7) }}</span>
                <span class="breadcrumb-date">
                  <span class="breadcrumb-icon">📅</span>
                  {{ formatDate(commit.date) }}
                </span>
                <span class="breadcrumb-branch">
                  <span class="breadcrumb-icon">🌿</span>
                  {{ commit.branch }}
                </span>
              </div>
              <div class="header-metrics">
                <span class="metric-add">+{{ commit.add }}</span>
                <span class="metric-del">-{{ commit.del }}</span>
              </div>
            </div>

            <!-- TITLE -->
            <div class="module-title">
              <span class="title-marker">{{ commit.emoji }}</span>
              <h3>{{ commit.message }}</h3>
            </div>

            <!-- DESCRIPTION -->
            <transition name="fade">
              <div v-if="commit.description" class="description-bubble">
                <p>{{ commit.description }}</p>
              </div>
            </transition>

            <!-- INTERACTION BAR -->
            <div class="interaction-bar">
              <div class="actor">
                <div class="actor-avatar">
                  <img :src="`https://github.com/${commit.author}.png`" :alt="commit.author">
                  <span class="avatar-glow"></span>
                </div>
                <span class="actor-name">{{ commit.author }}</span>
              </div>
              
              <div class="actions">
                <button class="action-liquid" @click="toggle(commit.id)">
                  <span class="action-icon">{{ open[commit.id] ? '−' : '+' }}</span>
                  <span class="action-text">{{ open[commit.id] ? 'Hide' : 'View' }}</span>
                  <span class="action-badge">{{ commit.files.length }}</span>
                  <span class="action-arrow">{{ open[commit.id] ? '▲' : '▼' }}</span>
                </button>
                
                <a :href="`https://github.com/ianncxd/wiki-wildfire-inc/commit/${commit.id}`" 
                   target="_blank" 
                   class="action-liquid github-action">
                  <span class="action-icon">🔗</span>
                  <span class="action-text">GitHub</span>
                  <span class="action-arrow">→</span>
                </a>
              </div>
            </div>

            <!-- EXPAND CONTENT -->
            <transition name="liquid">
              <div v-if="open[commit.id]" class="liquid-expand">
                <!-- FILES -->
                <div class="expand-section">
                  <div class="section-label">
                    <span class="label-icon">📄</span>
                    <span class="label-text">Files changed</span>
                    <span class="label-count">{{ commit.files.length }}</span>
                  </div>
                  <div class="files-pool">
                    <div v-for="file in commit.files" :key="file" class="file-bubble">
                      <span class="bubble-icon">📄</span>
                      <span class="bubble-path">{{ file }}</span>
                    </div>
                  </div>
                </div>

                <!-- CHANGES -->
                <div v-if="commit.changes" class="expand-section">
                  <div class="section-label">
                    <span class="label-icon">🔄</span>
                    <span class="label-text">Key changes</span>
                    <span class="label-count">{{ commit.changes.length }}</span>
                  </div>
                  <div class="changes-stream">
                    <div v-for="change in commit.changes" :key="change" class="change-drop">
                      <span class="drop-bullet">●</span>
                      <span class="drop-text">{{ change }}</span>
                    </div>
                  </div>
                </div>
              </div>
            </transition>

            <!-- TAGS -->
            <div class="tags-cloud">
              <PageTagRed v-if="commit.tags.includes('fix')">🔧 fix</PageTagRed>
              <PageTagGreen v-if="commit.tags.includes('feature')">✨ feature</PageTagGreen>
              <PageTagBlue v-if="commit.tags.includes('docs')">📚 docs</PageTagBlue>
              <PageTagPurple v-if="commit.tags.includes('api')">⚡ api</PageTagPurple>
              <PageTagOrange v-if="commit.tags.includes('update')">🔄 update</PageTagOrange>
              <PageTagPink v-if="commit.tags.includes('tutorial')">📖 tutorial</PageTagPink>
              <PageTagIndigo v-if="commit.tags.includes('server')">🌐 server</PageTagIndigo>
              <PageTagYellow v-if="commit.tags.includes('security')">🔒 security</PageTagYellow>
              <PageTagTeal v-if="commit.tags.includes('refactor')">🛠️ refactor</PageTagTeal>
            </div>
          </div>
        </div>
      </transition-group>
    </div>

    <!-- EMPTY STATE -->
    <transition name="fade">
      <div v-if="filteredCommits.length === 0" class="empty-liquid">
        <div class="empty-sphere">
          <span>🔍</span>
        </div>
        <p>No commits found</p>
        <button @click="resetFilters">Reset filters</button>
      </div>
    </transition>
  </div>
</template>

<script>
export default {
  name: 'Changelogs',
  
  data() {
    return {
      search: '',
      sortOrder: 'desc',
      tagFilter: 'all',
      open: {},
      
      commits: [
        {
          id: 'a0d9e8f7b6c5d4e3f2g1h0i9j8k7l6m5',
          hash: 'a0d9e8f',
          message: '🎉 Initial commit - Wiki structure',
          description: 'Created the initial structure for the WildFire wiki. Added main pages and basic configuration.',
          emoji: '🎉',
          author: 'ianncxd',
          date: '2024-01-15',
          branch: 'main',
          files: [
            'README.md',
            'docs/index.md',
            'docs/getting-started.md',
            'docs/guides/index.md',
            '.vitepress/config.js'
          ],
          changes: [
            'Created main wiki structure',
            'Added getting started guide',
            'Configured VitePress',
            'Set up basic navigation'
          ],
          tags: ['feature', 'docs'],
          add: 1245,
          del: 0
        },
        {
          id: 'b1c2d3e4f5g6h7i8j9k0l1m2n3o4p5q6',
          hash: 'b1c2d3e',
          message: '📝 Add FAQ section',
          description: 'Created comprehensive FAQ section with answers to common questions about WildFire services and community.',
          emoji: '📝',
          author: 'ianncxd',
          date: '2024-01-18',
          branch: 'main',
          files: [
            'docs/faq/index.md',
            'docs/faq/general.md',
            'docs/faq/technical.md',
            'docs/faq/community.md'
          ],
          changes: [
            'Added 25+ FAQ entries',
            'Categorized questions by topic',
            'Added searchable format',
            'Linked to relevant guides'
          ],
          tags: ['feature', 'docs'],
          add: 3456,
          del: 0
        },
        {
          id: 'c7d8e9f0a1b2c3d4e5f6g7h8i9j0k1l2',
          hash: 'c7d8e9f',
          message: '🔧 Fix broken links in documentation',
          description: 'Fixed multiple broken links across the wiki, updated outdated references, and improved navigation.',
          emoji: '🔧',
          author: 'ianncxd',
          date: '2024-01-22',
          branch: 'hotfix',
          files: [
            'docs/guides/deployment.md',
            'docs/guides/configuration.md',
            'docs/api/endpoints.md',
            'docs/index.md'
          ],
          changes: [
            'Fixed 15 broken external links',
            'Updated 8 outdated references',
            'Improved internal linking structure',
            'Added redirects for moved pages'
          ],
          tags: ['fix'],
          add: 234,
          del: 89
        },
        {
          id: 'd3e4f5g6h7i8j9k0l1m2n3o4p5q6r7s8',
          hash: 'd3e4f5g',
          message: '⚙️ Update VitePress configuration',
          description: 'Updated VitePress config with new theme options, added search functionality, and improved sidebar navigation.',
          emoji: '⚙️',
          author: 'ianncxd',
          date: '2024-01-25',
          branch: 'main',
          files: [
            '.vitepress/config.js',
            '.vitepress/theme/index.js',
            '.vitepress/theme/style.css',
            'package.json'
          ],
          changes: [
            'Enabled full-text search',
            'Updated theme configuration',
            'Added dark mode support',
            'Improved mobile responsiveness'
          ],
          tags: ['config', 'update'],
          add: 567,
          del: 123
        },
        {
          id: 'e9f0a1b2c3d4e5f6g7h8i9j0k1l2m3n4',
          hash: 'e9f0a1b',
          message: '📚 Add deployment guides',
          description: 'Added comprehensive deployment guides for different platforms including Docker, VPS, and cloud providers.',
          emoji: '📚',
          author: 'ianncxd',
          date: '2024-01-28',
          branch: 'feature/guides',
          files: [
            'docs/guides/deployment/docker.md',
            'docs/guides/deployment/vps.md',
            'docs/guides/deployment/cloud.md',
            'docs/guides/deployment/netlify.md'
          ],
          changes: [
            'Docker deployment guide with examples',
            'VPS setup instructions for Ubuntu',
            'AWS and GCP deployment options',
            'Netlify one-click deploy guide'
          ],
          tags: ['feature', 'docs'],
          add: 2890,
          del: 0
        },
        {
          id: 'f1a2b3c4d5e6f7g8h9i0j1k2l3m4n5o6',
          hash: 'f1a2b3c',
          message: '🔄 Update API documentation',
          description: 'Updated API documentation with new endpoints, authentication methods, and code examples.',
          emoji: '🔄',
          author: 'ianncxd',
          date: '2024-02-01',
          branch: 'main',
          files: [
            'docs/api/authentication.md',
            'docs/api/endpoints.md',
            'docs/api/examples.md',
            'docs/api/errors.md'
          ],
          changes: [
            'Added 5 new API endpoints',
            'Updated OAuth2 authentication flow',
            'Added code examples in JS, Python, and curl',
            'Improved error handling documentation'
          ],
          tags: ['update', 'api'],
          add: 1876,
          del: 234
        },
        {
          id: 'g2h3i4j5k6l7m8n9o0p1q2r3s4t5u6v7',
          hash: 'g2h3i4j',
          message: '🐛 Fix mobile navigation',
          description: 'Fixed responsive issues with mobile navigation menu. Improved touch targets and menu behavior on small screens.',
          emoji: '🐛',
          author: 'ianncxd',
          date: '2024-02-05',
          branch: 'hotfix/mobile',
          files: [
            '.vitepress/theme/components/NavBar.vue',
            '.vitepress/theme/styles/mobile.css',
            'docs/index.md'
          ],
          changes: [
            'Fixed hamburger menu on iOS',
            'Improved touch target sizes',
            'Added smooth animations',
            'Fixed z-index issues'
          ],
          tags: ['fix'],
          add: 345,
          del: 178
        },
        {
          id: 'h5i6j7k8l9m0n1o2p3q4r5s6t7u8v9w0',
          hash: 'h5i6j7k',
          message: '✨ Add search functionality',
          description: 'Implemented full-text search across the wiki using FlexSearch. Added search modal and keyboard shortcuts.',
          emoji: '✨',
          author: 'ianncxd',
          date: '2024-02-08',
          branch: 'feature/search',
          files: [
            '.vitepress/theme/components/SearchModal.vue',
            '.vitepress/theme/composables/useSearch.ts',
            'package.json',
            'docs/index.md'
          ],
          changes: [
            'Implemented FlexSearch integration',
            'Added search modal with keyboard shortcut (Ctrl+K)',
            'Real-time search results',
            'Highlighted matching terms'
          ],
          tags: ['feature', 'config'],
          add: 2345,
          del: 0
        },
        {
          id: 'i8j9k0l1m2n3o4p5q6r7s8t9u0v1w2x3',
          hash: 'i8j9k0l',
          message: '📖 Create contributor guide',
          description: 'Added comprehensive guide for contributors including coding standards, PR process, and community guidelines.',
          emoji: '📖',
          author: 'ianncxd',
          date: '2024-02-12',
          branch: 'main',
          files: [
            'CONTRIBUTING.md',
            'docs/contributing/index.md',
            'docs/contributing/code-style.md',
            'docs/contributing/pr-guide.md'
          ],
          changes: [
            'Added coding standards section',
            'Pull request template and process',
            'Commit message conventions',
            'Community guidelines'
          ],
          tags: ['docs'],
          add: 3456,
          del: 0
        },
        {
          id: 'j0k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5',
          hash: 'j0k1l2m',
          message: '⚡ Optimize images and assets',
          description: 'Optimized all images and assets for faster loading. Implemented lazy loading and responsive images.',
          emoji: '⚡',
          author: 'ianncxd',
          date: '2024-02-15',
          branch: 'main',
          files: [
            'docs/assets/**/*.png',
            'docs/assets/**/*.jpg',
            'docs/assets/**/*.svg',
            '.vitepress/config.js'
          ],
          changes: [
            'Compressed 50+ images (60% size reduction)',
            'Implemented lazy loading',
            'Added responsive image srcsets',
            'Optimized SVG files'
          ],
          tags: ['update'],
          add: 456,
          del: 2345
        },
        {
          id: 'k1l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6',
          hash: 'k1l2m3n',
          message: '🔧 Fix typo in README',
          description: 'Fixed a small typo in the main README file and updated the getting started instructions.',
          emoji: '🔧',
          author: 'ianncxd',
          date: '2024-02-18',
          branch: 'hotfix',
          files: [
            'README.md',
            'docs/getting-started.md'
          ],
          changes: [
            'Fixed spelling errors',
            'Updated installation steps',
            'Added quick start example'
          ],
          tags: ['fix'],
          add: 45,
          del: 12
        },
        {
          id: 'l2m3n4o5p6q7r8s9t0u1v2w3x4y5z6a7',
          hash: 'l2m3n4o',
          message: '📚 Add troubleshooting section',
          description: 'Added a comprehensive troubleshooting section with common issues and solutions for WildFire services.',
          emoji: '📚',
          author: 'ianncxd',
          date: '2024-02-20',
          branch: 'main',
          files: [
            'docs/troubleshooting/index.md',
            'docs/troubleshooting/common-issues.md',
            'docs/troubleshooting/faq.md'
          ],
          changes: [
            '20+ common issues with solutions',
            'Diagnostic commands section',
            'Error code reference',
            'Community support links'
          ],
          tags: ['docs', 'feature'],
          add: 2345,
          del: 0
        },
        {
          id: 'm3n4o5p6q7r8s9t0u1v2w3x4y5z6a7b8',
          hash: 'm3n4o5p',
          message: '⚙️ Update theme configuration',
          description: 'Updated the theme configuration with new colors, fonts, and layout options for better readability.',
          emoji: '⚙️',
          author: 'ianncxd',
          date: '2024-02-23',
          branch: 'main',
          files: [
            '.vitepress/theme/index.js',
            '.vitepress/theme/styles/vars.css',
            '.vitepress/theme/styles/globals.css'
          ],
          changes: [
            'New color scheme with better contrast',
            'Updated typography system',
            'Improved spacing and layout',
            'Added print styles'
          ],
          tags: ['config', 'update'],
          add: 678,
          del: 345
        },
        {
          id: 'n4o5p6q7r8s9t0u1v2w3x4y5z6a7b8c9',
          hash: 'n4o5p6q',
          message: '🔄 Update dependencies',
          description: 'Updated all npm dependencies to their latest versions. Fixed security vulnerabilities and improved performance.',
          emoji: '🔄',
          author: 'ianncxd',
          date: '2024-02-26',
          branch: 'main',
          files: [
            'package.json',
            'package-lock.json'
          ],
          changes: [
            'Updated VitePress to v1.0.0',
            'Patched security vulnerabilities',
            'Removed deprecated packages',
            'Improved build performance'
          ],
          tags: ['update'],
          add: 567,
          del: 234
        },
        {
          id: 'o5p6q7r8s9t0u1v2w3x4y5z6a7b8c9d0',
          hash: 'o5p6q7r',
          message: '✨ Add community section',
          description: 'Created a new community section with information about Discord, forums, events, and how to get involved.',
          emoji: '✨',
          author: 'ianncxd',
          date: '2024-02-29',
          branch: 'feature/community',
          files: [
            'docs/community/index.md',
            'docs/community/discord.md',
            'docs/community/events.md',
            'docs/community/contributors.md'
          ],
          changes: [
            'Discord server rules and channels',
            'Monthly community events calendar',
            'Contributor spotlight section',
            'Community showcase'
          ],
          tags: ['feature', 'docs'],
          add: 2890,
          del: 0
        },
        {
          id: 'p6q7r8s9t0u1v2w3x4y5z6a7b8c9d0e1',
          hash: 'p6q7r8s',
          message: '🔧 Fix search modal on mobile',
          description: 'Fixed search modal display issues on mobile devices and improved touch interactions.',
          emoji: '🔧',
          author: 'ianncxd',
          date: '2024-03-02',
          branch: 'hotfix',
          files: [
            '.vitepress/theme/components/SearchModal.vue',
            '.vitepress/theme/styles/mobile.css'
          ],
          changes: [
            'Fixed modal positioning',
            'Improved touch scrolling',
            'Added swipe to close',
            'Fixed keyboard issues'
          ],
          tags: ['fix'],
          add: 234,
          del: 89
        },
        {
          id: 'q7r8s9t0u1v2w3x4y5z6a7b8c9d0e1f2',
          hash: 'q7r8s9t',
          message: '📝 Add API rate limiting guide',
          description: 'Added detailed documentation about API rate limits, headers, and best practices for handling limits.',
          emoji: '📝',
          author: 'ianncxd',
          date: '2024-03-05',
          branch: 'main',
          files: [
            'docs/api/rate-limiting.md',
            'docs/api/best-practices.md'
          ],
          changes: [
            'Rate limit headers explanation',
            'Backoff strategies',
            'Code examples for handling limits',
            'Quota management tips'
          ],
          tags: ['docs', 'api'],
          add: 1456,
          del: 0
        }
      ]
    }
  },

  computed: {
    filteredCommits() {
      let filtered = [...this.commits]

      if (this.search) {
        const q = this.search.toLowerCase()
        filtered = filtered.filter(c => 
          c.message.toLowerCase().includes(q) ||
          c.description?.toLowerCase().includes(q) ||
          c.author.includes(q) ||
          c.tags.some(t => t.includes(q))
        )
      }

      if (this.tagFilter !== 'all') {
        filtered = filtered.filter(c => c.tags.includes(this.tagFilter))
      }

      filtered.sort((a, b) => {
        return this.sortOrder === 'desc' 
          ? new Date(b.date) - new Date(a.date)
          : new Date(a.date) - new Date(b.date)
      })

      return filtered
    }
  },

  methods: {
    formatDate(date) {
      const d = new Date(date)
      const now = new Date()
      const diff = Math.floor((now - d) / (1000 * 60 * 60 * 24))
      
      if (diff === 0) return 'today'
      if (diff === 1) return 'yesterday'
      if (diff < 7) return `${diff} days ago`
      return d.toLocaleDateString('en-US', { month: 'short', day: 'numeric' })
    },

    toggle(id) {
      this.open[id] = !this.open[id]
      this.open = {...this.open}
    },

    resetFilters() {
      this.search = ''
      this.sortOrder = 'desc'
      this.tagFilter = 'all'
    }
  },

  mounted() {
    const openState = {}
    this.commits.forEach(c => {
      openState[c.id] = false
    })
    this.open = openState
  }
}
</script>

<style scoped>
/* ===== CSS VARIABLES - DARK MODE DEFAULT ===== */
.changelogs {
  --primary: #ff8c00;
  --primary-dark: #ff4500;
  --primary-glow: rgba(255, 140, 0, 0.25);
  --primary-soft: rgba(255, 140, 0, 0.15);
  
  --bg-base: #050505;
  --bg-surface: rgba(10, 10, 10, 0.85);
  --bg-surface-raised: rgba(18, 18, 18, 0.9);
  --bg-elevated: rgba(25, 25, 25, 0.95);
  
  --text-primary: #ffffff;
  --text-secondary: #f0f0f0;
  --text-tertiary: #a0a0a0;
  
  --border-subtle: rgba(255, 255, 255, 0.08);
  --border-strong: rgba(255, 140, 0, 0.3);
  
  --blur-amount: 20px;
  --blur-amount-strong: 30px;
  
  --green: #34c759;
  --green-soft: rgba(52, 199, 89, 0.15);
  --red: #ff3b30;
  --red-soft: rgba(255, 59, 48, 0.15);
  
  max-width: 750px;
  margin: 0 auto;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  color: var(--text-primary);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  transition: background-color 0.3s ease, color 0.2s ease;
}

/* ===== LIGHT THEME OVERRIDES ===== */
html:not(.dark) .changelogs {
  --bg-base: #f5f5f7;
  --bg-surface: rgba(255, 255, 255, 0.8);
  --bg-surface-raised: rgba(255, 255, 255, 0.9);
  --bg-elevated: rgba(255, 255, 255, 0.95);
  
  --text-primary: #1d1d1f;
  --text-secondary: #333336;
  --text-tertiary: #6e6e73;
  
  --border-subtle: rgba(0, 0, 0, 0.08);
  --border-strong: rgba(255, 140, 0, 0.3);
  
  --primary-soft: rgba(255, 140, 0, 0.08);
}

/* ===== FILTERS PANEL ===== */
.filters-panel {
  background: var(--bg-surface);
  backdrop-filter: blur(var(--blur-amount));
  -webkit-backdrop-filter: blur(var(--blur-amount));
  border-radius: 24px;
  padding: 20px;
  margin-bottom: 28px;
  border: 1px solid var(--border-subtle);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.5);
  transition: all 0.3s cubic-bezier(0.2, 0.9, 0.4, 1);
}

html:not(.dark) .filters-panel {
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

/* SEARCH */
.search-container {
  margin-bottom: 16px;
}

.search-field {
  position: relative;
  margin-bottom: 10px;
}

.search-glass {
  position: absolute;
  left: 14px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--text-tertiary);
  font-size: 16px;
  opacity: 0.8;
  pointer-events: none;
  transition: color 0.2s;
}

.search-field input {
  width: 100%;
  padding: 14px 40px 14px 44px;
  background: var(--bg-elevated);
  border: 1px solid var(--border-subtle);
  border-radius: 40px;
  color: var(--text-primary);
  font-size: 16px;
  transition: all 0.2s cubic-bezier(0.2, 0.9, 0.4, 1);
}

.search-field input:focus {
  outline: none;
  border-color: var(--primary);
  background: var(--bg-elevated);
  box-shadow: 0 0 0 3px var(--primary-soft);
}

.search-field input::placeholder {
  color: var(--text-tertiary);
  opacity: 0.6;
}

.search-clear {
  position: absolute;
  right: 14px;
  top: 50%;
  transform: translateY(-50%);
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--border-subtle);
  border-radius: 50%;
  color: var(--text-tertiary);
  font-size: 11px;
  cursor: pointer;
  transition: all 0.2s;
}

.search-clear:hover {
  background: var(--primary);
  color: white;
  transform: translateY(-50%) scale(1.1);
}

.search-hints {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.hint-chip {
  padding: 6px 14px;
  background: var(--bg-elevated);
  border: 1px solid var(--border-subtle);
  border-radius: 30px;
  color: var(--text-secondary);
  font-size: 12px;
  cursor: pointer;
  transition: all 0.2s;
  backdrop-filter: blur(5px);
}

.hint-chip:hover {
  background: var(--primary-soft);
  border-color: var(--primary);
  color: var(--primary);
  transform: translateY(-1px);
}

/* FILTERS ROW */
.filters-row {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.sort-group {
  display: flex;
  gap: 4px;
  background: var(--bg-elevated);
  padding: 4px;
  border-radius: 40px;
  border: 1px solid var(--border-subtle);
}

.sort-pill {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 6px 14px;
  background: transparent;
  border: none;
  border-radius: 40px;
  color: var(--text-tertiary);
  cursor: pointer;
  transition: all 0.2s;
  font-size: 13px;
}

.sort-pill.active {
  background: var(--primary-soft);
  color: var(--primary);
}

.pill-icon {
  font-size: 14px;
}

.tags-group {
  display: flex;
  gap: 4px;
  background: var(--bg-elevated);
  padding: 4px;
  border-radius: 40px;
  border: 1px solid var(--border-subtle);
}

.tags-group :deep(.page-tag) {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 0;
  font-size: 18px;
  opacity: 0.6;
  cursor: pointer;
  transition: all 0.2s;
}

.tags-group :deep(.page-tag.tag-active) {
  opacity: 1;
  background: var(--primary-soft);
  transform: scale(1.05);
}

.reset-pill {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 6px 16px;
  background: var(--bg-elevated);
  border: 1px solid var(--border-subtle);
  border-radius: 40px;
  color: var(--text-tertiary);
  cursor: pointer;
  transition: all 0.2s;
  font-size: 13px;
}

.reset-pill:hover {
  border-color: var(--primary);
  color: var(--primary);
  transform: rotate(180deg);
}

.count-badge {
  margin-left: auto;
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 6px 16px;
  background: var(--primary-soft);
  border-radius: 40px;
  border: 1px solid var(--border-strong);
}

.count-number {
  color: var(--primary);
  font-size: 16px;
  font-weight: 600;
}

.count-label {
  color: var(--text-tertiary);
  font-size: 12px;
}

/* ACTIVE FILTERS */
.active-filters {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-top: 16px;
  padding-top: 16px;
  border-top: 1px solid var(--border-subtle);
}

.filter-tag {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 12px;
  background: var(--primary-soft);
  border: 1px solid var(--border-strong);
  border-radius: 40px;
  color: var(--primary);
  font-size: 12px;
  backdrop-filter: blur(5px);
}

.filter-tag button {
  width: 18px;
  height: 18px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--border-subtle);
  border: none;
  border-radius: 50%;
  color: var(--text-tertiary);
  font-size: 9px;
  cursor: pointer;
  padding: 0;
  transition: all 0.2s;
}

.filter-tag button:hover {
  background: var(--primary);
  color: white;
  transform: scale(1.1);
}

/* ===== COMMITS FLOW ===== */
.commits-flow {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.commits-grid {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.commit-module {
  animation: floatIn 0.5s cubic-bezier(0.2, 0.9, 0.3, 1) forwards;
  opacity: 0;
  transform: translateY(20px);
}

@keyframes floatIn {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* GLASS CARD */
.glass-card {
  background: var(--bg-surface);
  backdrop-filter: blur(var(--blur-amount));
  -webkit-backdrop-filter: blur(var(--blur-amount));
  border-radius: 28px;
  padding: 24px;
  border: 1px solid var(--border-subtle);
  transition: all 0.3s cubic-bezier(0.2, 0.9, 0.4, 1);
  position: relative;
  overflow: hidden;
}

.glass-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  right: 100%;
  height: 2px;
  background: linear-gradient(90deg, 
    transparent, 
    var(--primary), 
    var(--primary-dark), 
    transparent
  );
  transition: left 0.6s cubic-bezier(0.2, 0.9, 0.4, 1);
}

.glass-card:hover {
  transform: translateY(-3px) scale(1.01);
  border-color: var(--border-strong);
  background: var(--bg-surface-raised);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.5);
}

.glass-card:hover::before {
  left: -30%;
  right: -30%;
}

html:not(.dark) .glass-card:hover {
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
}

/* MODULE HEADER */
.module-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  flex-wrap: wrap;
  gap: 12px;
}

.header-breadcrumbs {
  display: flex;
  align-items: center;
  gap: 12px;
  flex-wrap: wrap;
}

.breadcrumb-hash {
  font-family: 'SF Mono', 'JetBrains Mono', monospace;
  font-size: 12px;
  color: var(--primary);
  background: var(--primary-soft);
  padding: 4px 12px;
  border-radius: 30px;
  letter-spacing: 0.3px;
  border: 1px solid var(--border-strong);
  transition: all 0.2s;
}

.breadcrumb-hash:hover {
  transform: scale(1.02);
}

.breadcrumb-date, .breadcrumb-branch {
  display: flex;
  align-items: center;
  gap: 4px;
  color: var(--text-tertiary);
  font-size: 12px;
  transition: color 0.2s;
}

.breadcrumb-icon {
  font-size: 12px;
  opacity: 0.7;
  transition: opacity 0.2s;
}

.breadcrumb-date:hover, .breadcrumb-branch:hover {
  color: var(--primary);
}

.breadcrumb-date:hover .breadcrumb-icon,
.breadcrumb-branch:hover .breadcrumb-icon {
  opacity: 1;
}

.header-metrics {
  display: flex;
  gap: 6px;
}

.metric-add, .metric-del {
  padding: 4px 12px;
  border-radius: 30px;
  font-size: 11px;
  font-weight: 600;
  transition: all 0.2s;
}

.metric-add {
  background: var(--green-soft);
  color: var(--green);
  border: 1px solid rgba(52, 199, 89, 0.3);
}

.metric-del {
  background: var(--red-soft);
  color: var(--red);
  border: 1px solid rgba(255, 59, 48, 0.3);
}

.metric-add:hover, .metric-del:hover {
  transform: scale(1.05);
}

/* MODULE TITLE */
.module-title {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 18px;
}

.title-marker {
  font-size: 28px;
  min-width: 40px;
  text-align: center;
  filter: drop-shadow(0 4px 8px var(--primary-glow));
  transition: transform 0.3s;
}

.glass-card:hover .title-marker {
  transform: scale(1.05) rotate(5deg);
}

.module-title h3 {
  color: var(--text-primary);
  font-size: 17px;
  font-weight: 500;
  margin: 0;
  line-height: 1.4;
  letter-spacing: -0.2px;
  transition: color 0.2s;
}

/* DESCRIPTION BUBBLE */
.description-bubble {
  margin-bottom: 20px;
  padding: 16px 20px;
  background: var(--bg-elevated);
  border-radius: 20px;
  border: 1px solid var(--border-subtle);
  backdrop-filter: blur(5px);
  position: relative;
  transition: all 0.3s;
}

.description-bubble:hover {
  border-color: var(--primary-soft);
  background: var(--bg-surface-raised);
}

.description-bubble p {
  color: var(--text-secondary);
  font-size: 14px;
  line-height: 1.6;
  margin: 0;
  font-weight: 400;
}

/* INTERACTION BAR */
.interaction-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 20px;
  border-bottom: 1px solid var(--border-subtle);
  transition: border-color 0.3s;
}

.glass-card:hover .interaction-bar {
  border-bottom-color: var(--border-strong);
}

.actor {
  display: flex;
  align-items: center;
  gap: 10px;
}

.actor-avatar {
  position: relative;
  width: 36px;
  height: 36px;
}

.actor-avatar img {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  border: 2px solid var(--primary);
  position: relative;
  z-index: 2;
  transition: border-color 0.3s;
}

.avatar-glow {
  position: absolute;
  top: -2px;
  left: -2px;
  right: -2px;
  bottom: -2px;
  border-radius: 50%;
  background: var(--primary);
  opacity: 0.3;
  filter: blur(4px);
  z-index: 1;
  transition: opacity 0.3s, transform 0.3s;
}

.actor:hover .avatar-glow {
  opacity: 0.6;
  transform: scale(1.1);
}

.actor-name {
  color: var(--primary);
  font-size: 14px;
  font-weight: 500;
  transition: color 0.2s;
}

.actions {
  display: flex;
  gap: 8px;
}

.action-liquid {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background: var(--bg-elevated);
  border: 1px solid var(--border-subtle);
  border-radius: 40px;
  color: var(--text-tertiary);
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.2, 0.9, 0.4, 1);
  font-size: 13px;
  backdrop-filter: blur(5px);
}

.action-liquid:hover {
  background: var(--primary-soft);
  border-color: var(--primary);
  color: var(--primary);
  transform: scale(1.02) translateY(-1px);
  box-shadow: 0 4px 12px var(--primary-glow);
}

.github-action:hover {
  background: var(--bg-elevated);
  border-color: var(--text-tertiary);
  color: var(--text-primary);
  transform: scale(1.02) translateY(-1px);
}

.action-icon {
  font-size: 14px;
  transition: transform 0.3s;
}

.action-liquid:hover .action-icon {
  transform: scale(1.1);
}

.action-badge {
  background: var(--primary-soft);
  color: var(--primary);
  padding: 2px 8px;
  border-radius: 30px;
  font-size: 10px;
  font-weight: 600;
  margin-left: 2px;
  transition: all 0.3s;
}

.action-liquid:hover .action-badge {
  background: var(--primary);
  color: white;
}

.action-arrow {
  font-size: 10px;
  transition: transform 0.3s;
}

.action-liquid:hover .action-arrow {
  transform: translateX(4px);
}

/* LIQUID EXPAND */
.liquid-expand {
  margin-bottom: 20px;
  animation: liquidAppear 0.4s cubic-bezier(0.2, 0.9, 0.3, 1);
  transform-origin: top;
}

@keyframes liquidAppear {
  0% {
    opacity: 0;
    transform: scaleY(0.7) translateY(-20px);
  }
  50% {
    opacity: 0.8;
    transform: scaleY(1.02) translateY(2px);
  }
  100% {
    opacity: 1;
    transform: scaleY(1) translateY(0);
  }
}

.expand-section {
  margin-bottom: 20px;
}

.expand-section:last-child {
  margin-bottom: 0;
}

.section-label {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-bottom: 14px;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border-subtle);
  transition: border-color 0.3s;
}

.glass-card:hover .section-label {
  border-bottom-color: var(--border-strong);
}

.label-icon {
  color: var(--primary);
  font-size: 14px;
  transition: transform 0.3s;
}

.label-icon:hover {
  transform: rotate(10deg);
}

.label-text {
  color: var(--primary);
  font-size: 13px;
  font-weight: 500;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.label-count {
  margin-left: auto;
  background: var(--primary-soft);
  color: var(--primary);
  padding: 2px 10px;
  border-radius: 30px;
  font-size: 11px;
  font-weight: 600;
  transition: all 0.3s;
}

.label-count:hover {
  background: var(--primary);
  color: white;
  transform: scale(1.05);
}

/* FILES POOL */
.files-pool {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.file-bubble {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 14px;
  background: var(--bg-elevated);
  border-radius: 14px;
  border: 1px solid var(--border-subtle);
  transition: all 0.3s;
  backdrop-filter: blur(5px);
  animation: fileSlide 0.3s ease forwards;
  opacity: 0;
  transform: translateX(-10px);
}

@keyframes fileSlide {
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.file-bubble:hover {
  background: var(--primary-soft);
  transform: translateX(8px);
  border-color: var(--primary);
  box-shadow: 0 4px 12px var(--primary-glow);
}

.bubble-icon {
  color: var(--primary);
  font-size: 12px;
  opacity: 0.8;
  transition: transform 0.3s;
}

.file-bubble:hover .bubble-icon {
  transform: scale(1.2);
}

.bubble-path {
  color: #88c0d0;
  font-family: 'SF Mono', 'JetBrains Mono', monospace;
  font-size: 12px;
  word-break: break-all;
  transition: color 0.3s;
}

.file-bubble:hover .bubble-path {
  color: var(--primary);
}

/* CHANGES STREAM */
.changes-stream {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.change-drop {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 0;
  border-bottom: 1px dashed var(--border-subtle);
  transition: all 0.3s;
  animation: changeFade 0.3s ease forwards;
  opacity: 0;
  transform: translateY(5px);
}

@keyframes changeFade {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.change-drop:last-child {
  border-bottom: none;
}

.drop-bullet {
  color: var(--primary);
  font-size: 10px;
  min-width: 20px;
  text-align: center;
  transition: all 0.3s;
}

.change-drop:hover .drop-bullet {
  transform: scale(1.8);
  color: var(--primary-dark);
}

.drop-text {
  color: var(--text-secondary);
  font-size: 13px;
  line-height: 1.5;
  flex: 1;
  transition: color 0.3s;
}

.change-drop:hover .drop-text {
  color: var(--primary);
}

/* TAGS CLOUD */
.tags-cloud {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  margin-top: 8px;
}

.tags-cloud :deep(.page-tag) {
  padding: 6px 14px;
  font-size: 11px;
  backdrop-filter: blur(5px);
  transition: all 0.3s;
  opacity: 0.9;
}

.tags-cloud :deep(.page-tag:hover) {
  transform: translateY(-3px) scale(1.02);
  filter: brightness(1.2);
  box-shadow: 0 4px 12px var(--primary-glow);
  opacity: 1;
}

/* ===== TRANSITIONS ===== */
.list-enter-active,
.list-leave-active {
  transition: all 0.3s ease;
}

.list-enter-from {
  opacity: 0;
  transform: translateY(30px);
}

.list-leave-to {
  opacity: 0;
  transform: translateY(-30px);
}

.list-move {
  transition: transform 0.3s ease;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: all 0.3s ease;
}

.fade-slide-enter-from {
  opacity: 0;
  transform: translateY(-10px);
}

.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.liquid-enter-active,
.liquid-leave-active {
  transition: all 0.3s cubic-bezier(0.2, 0.9, 0.3, 1);
}

.liquid-enter-from {
  opacity: 0;
  transform: scaleY(0.8) translateY(-15px);
}

.liquid-leave-to {
  opacity: 0;
  transform: scaleY(0.8) translateY(-15px);
}

/* EMPTY LIQUID */
.empty-liquid {
  text-align: center;
  padding: 60px 20px;
  background: var(--bg-surface);
  backdrop-filter: blur(var(--blur-amount));
  border-radius: 40px;
  border: 1px solid var(--border-subtle);
  transition: all 0.3s;
}

.empty-sphere {
  width: 80px;
  height: 80px;
  margin: 0 auto 20px;
  background: var(--primary-soft);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 32px;
  border: 1px solid var(--border-strong);
  animation: floatPulse 3s infinite ease-in-out;
}

@keyframes floatPulse {
  0%, 100% {
    transform: translateY(0) scale(1);
    box-shadow: 0 0 0 0 var(--primary-glow);
  }
  50% {
    transform: translateY(-10px) scale(1.05);
    box-shadow: 0 0 30px 10px var(--primary-glow);
  }
}

.empty-liquid p {
  color: var(--text-tertiary);
  font-size: 15px;
  margin: 0 0 20px;
  transition: color 0.2s;
}

.empty-liquid button {
  padding: 10px 24px;
  background: var(--bg-elevated);
  border: 1px solid var(--border-subtle);
  border-radius: 40px;
  color: var(--primary);
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
  backdrop-filter: blur(5px);
}

.empty-liquid button:hover {
  border-color: var(--primary);
  background: var(--primary-soft);
  transform: translateY(-2px);
  box-shadow: 0 8px 20px var(--primary-glow);
}

/* RESPONSIVE */
@media (max-width: 600px) {
  .filters-row {
    flex-direction: column;
    align-items: stretch;
  }
  
  .sort-group, .tags-group, .reset-pill, .count-badge {
    width: 100%;
    justify-content: center;
  }
  
  .count-badge {
    margin-left: 0;
  }
  
  .module-header {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .header-metrics {
    width: 100%;
    justify-content: flex-end;
  }
  
  .interaction-bar {
    flex-direction: column;
    align-items: flex-start;
    gap: 15px;
  }
  
  .actions {
    width: 100%;
  }
  
  .action-liquid {
    flex: 1;
    justify-content: center;
  }
  
  .search-hints {
    justify-content: center;
  }
}
</style>