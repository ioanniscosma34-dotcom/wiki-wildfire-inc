<template>
  <div v-if="isHomePage" class="wildfire-home orbitron-font">
    <HomeNavbar v-if="isHomePage" />
    
    <img 
      src="/wallpaper/poza102.webp"
      alt=""
      class="wildfire-wallpaper"
      :class="{ 
        'wildfire-wallpaper-hover': isHovered
      }"
      width="1920"
      height="1080"
      fetchpriority="high"
      loading="eager"
      decoding="async"
      role="presentation"
    />
    
    <img 
      src="/wallpaper/da.webp"
      alt=""
      class="wildfire-wallpaper-overlay"
      :class="{ 
        'wildfire-wallpaper-overlay-loaded': isOverlayLoaded && isHovered 
      }"
      width="1920"
      height="1080"
      fetchpriority="low"
      loading="lazy"
      decoding="async"
      role="presentation"
    />
    
    <div class="wildfire-bg">
      <div class="wildfire-gradient"></div>
      <div class="wildfire-grid-primary"></div>
      <div class="wildfire-grid-secondary"></div>
      <div class="wildfire-scanlines"></div>
      <div class="wildfire-spotlight-tl"></div>
      <div class="wildfire-spotlight-br"></div>
      <div class="scroll-overlay" :style="{ opacity: scrollOverlayOpacity }"></div>
    </div>
    
    <main class="wildfire-hero">
      <div class="wildfire-container">
        <div class="wildfire-logo-container" @mouseenter="triggerBoomEffect">
          <div class="wildfire-logo-glow" :class="{ 'boom-active': isBoomActive }"></div>
          <div class="logo-boom-ring" v-for="n in 3" :key="n" :class="{ 'boom-active': isBoomActive }" :style="{ '--i': n }"></div>
          <img 
            src="/icons/wildfire.webp" 
            alt="Wildfire.ro" 
            class="wildfire-logo"
            :class="{ 'logo-start': logoStart, 'boom-active': isBoomActive }"
            width="150"
            height="150"
            fetchpriority="high"
            decoding="sync"
            @mouseenter="isHovered = true"
            @mouseleave="isHovered = false"
            ref="logoImg"
          >
        </div>

        <!-- Titlu cu litere mari, wild alb, fire gradient, ro gradient, docs alb -->
        <h1 class="wildfire-title typing-text orbitron-font" :class="{ 'typing-complete': titleTyped }">
          <span v-if="!titleTyped">
            <span v-for="(char, index) in displayTitleArray" :key="index" :class="getCharClass(index)">
              {{ char }}
            </span>
            <span class="cursor"></span>
          </span>
          <span v-else>
            <span class="wild-white">WILD</span><span class="fire-gradient-text">FIRE</span><span class="dot-orange">.</span><span class="ro-gradient-text">RO</span><span class="docs-white"> DOCS</span>
          </span>
        </h1>

        <!-- Feature badges cu efect de typing -->
        <div class="feature-badges">
          <div class="feature-badge orbitron-font" v-for="(badge, index) in featureBadges" :key="index">
            <span class="feature-icon">{{ badge.icon }}</span>
            <span class="feature-text typing-text orbitron-font" :class="{ 'typing-complete': badgesTyped[index] }">{{ displayBadges[index] }}<span v-if="!badgesTyped[index]" class="cursor"></span></span>
          </div>
        </div>

        <div class="wildfire-button-wrapper">
          <a 
            href="/informatii/about" 
            class="wildfire-button orbitron-font"
            @mouseenter="isHovered = true"
            @mouseleave="isHovered = false"
          >
            <span class="typing-text orbitron-font" :class="{ 'typing-complete': buttonTyped }">{{ displayButton }}<span v-if="!buttonTyped" class="cursor"></span></span>
            <span class="wildfire-button-arrow">→</span>
          </a>
        </div>

        <!-- Home Searchbar cu efect de typing ciclic -->
        <div class="home-search">
          <button class="home-search-button orbitron-font" @click="openSearch" type="button">
            <img 
              src="/icons/searchbutton.svg" 
              class="search-icon" 
              alt="search" 
              width="16" 
              height="16"
              loading="eager"
              decoding="async"
            >
            <span class="home-search-text typing-text orbitron-font">{{ displaySearchText }}<span class="cursor"></span></span>
            <span class="home-search-shortcut orbitron-font">CTRL K</span>
          </button>
        </div>

        <div class="community-tag delayed-tag">
          <span class="tag-badge typing-text orbitron-font" :class="{ 'typing-complete': tagTyped }">{{ displayTag }}<span v-if="!tagTyped" class="cursor"></span></span>
          <span class="tag-year orbitron-font">since 2021</span>
        </div>

        <!-- Last Updates -->
        <div class="last-updates-wrapper reveal-element" ref="lastUpdatesRef">
          <LastUpdates />
        </div>

        <!-- Wiki Showcase -->
        <section class="wiki-showcase reveal-element orbitron-font" ref="wikiSectionRef" id="wiki-section">
          <div class="section-divider">
            <span class="divider-line"></span>
            <img src="/icons/wildfire.webp" alt="divider" class="divider-icon" width="24" height="24" />
            <span class="divider-line"></span>
          </div>

          <div class="wiki-layout">
            <div class="wiki-left reveal-element orbitron-font" ref="wikiLeftRef">
              <div class="wiki-header">
                <div class="wiki-left-line">
                  <span class="line"></span>
                  <span class="text typing-text orbitron-font" :class="{ 'typing-complete': wikiHeaderTyped }">{{ displayWikiHeader }}<span v-if="!wikiHeaderTyped" class="cursor"></span></span>
                </div>
                <h2 class="wiki-title typing-text orbitron-font" :class="{ 'typing-complete': wikiTitleTyped }">{{ displayWikiTitle }}<span v-if="!wikiTitleTyped" class="cursor"></span></h2>
              </div>
              
              <div class="wiki-description">
                <p class="typing-text orbitron-font" :class="{ 'typing-complete': wikiDescTyped }">{{ displayWikiDesc }}<span v-if="!wikiDescTyped" class="cursor"></span></p>
              </div>
              
              <div class="wiki-cta">
                <a href="/informatii" class="wiki-cta-link orbitron-font">
                  <span class="typing-text orbitron-font" :class="{ 'typing-complete': wikiCTATyped }">{{ displayWikiCTA }}<span v-if="!wikiCTATyped" class="cursor"></span></span>
                  <svg width="16" height="16" viewBox="0 0 16 16" fill="none">
                    <path d="M6 3L11 8L6 13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                  </svg>
                </a>
              </div>
            </div>

            <div class="wiki-right">
              <!-- Primul rând - 3 carduri -->
              <div class="wiki-row">
                <div class="wiki-card reveal-element orbitron-font" ref="card1Ref" v-if="categoryCounts.systems > 0">
                  <div class="card-glow"></div>
                  <div class="card-icon">
                    <svg width="22" height="22" viewBox="0 0 22 22" fill="none">
                      <rect x="3" y="5" width="16" height="12" rx="2" stroke="currentColor" stroke-width="1.5"/>
                      <circle cx="7" cy="11" r="1.5" fill="currentColor"/>
                      <circle cx="15" cy="11" r="1.5" fill="currentColor"/>
                    </svg>
                  </div>
                  <div class="card-content">
                    <h3 class="card-title orbitron-font">Regulamente</h3>
                    <p class="card-desc orbitron-font">Toate regulile și normele de comportament pentru comunitate</p>
                    <div class="card-footer">
                      <span class="card-badge orbitron-font">{{ categoryCounts.systems }} pagini</span>
                      <a href="/systems" class="card-link">
                        <svg width="14" height="14" viewBox="0 0 14 14" fill="none">
                          <path d="M5 3L9 7L5 11" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        </svg>
                      </a>
                    </div>
                  </div>
                </div>

                <div class="wiki-card reveal-element orbitron-font" ref="card2Ref" v-if="categoryCounts.regulamente > 0">
                  <div class="card-glow"></div>
                  <div class="card-icon">
                    <svg width="22" height="22" viewBox="0 0 22 22" fill="none">
                      <path d="M11 3L14 8H19L16 12L18 17L11 14L4 17L6 12L3 8H8L11 3Z" stroke="currentColor" stroke-width="1.5"/>
                    </svg>
                  </div>
                  <div class="card-content">
                    <h3 class="card-title orbitron-font">Sisteme</h3>
                    <p class="card-desc orbitron-font">Sisteme specifice serverului și funcționalități unice</p>
                    <div class="card-footer">
                      <span class="card-badge orbitron-font">{{ categoryCounts.regulamente }} pagini</span>
                      <a href="/informatii/regulamente" class="card-link">
                        <svg width="14" height="14" viewBox="0 0 14 14" fill="none">
                          <path d="M5 3L9 7L5 11" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        </svg>
                      </a>
                    </div>
                  </div>
                </div>

                <div class="wiki-card reveal-element orbitron-font" ref="card3Ref" v-if="categoryCounts.market > 0">
                  <div class="card-glow"></div>
                  <div class="card-icon">
                    <svg width="22" height="22" viewBox="0 0 22 22" fill="none">
                      <circle cx="11" cy="11" r="7" stroke="currentColor" stroke-width="1.5"/>
                      <path d="M11 7V11L14 13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
                    </svg>
                  </div>
                  <div class="card-content">
                    <h3 class="card-title orbitron-font">Shop</h3>
                    <p class="card-desc orbitron-font">Toate informațiile despre marketplace-ul serverului</p>
                    <div class="card-footer">
                      <span class="card-badge orbitron-font">{{ categoryCounts.market }} pagini</span>
                      <a href="/market" class="card-link">
                        <svg width="14" height="14" viewBox="0 0 14 14" fill="none">
                          <path d="M5 3L9 7L5 11" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        </svg>
                      </a>
                    </div>
                  </div>
                </div>
              </div>

              <!-- Al doilea rând - 3 carduri -->
              <div class="wiki-row" v-if="categoryCounts.info > 0 || categoryCounts.currency > 0 || categoryCounts.vip > 0">
                <div class="wiki-card reveal-element orbitron-font" ref="card4Ref" v-if="categoryCounts.info > 0">
                  <div class="card-glow"></div>
                  <div class="card-icon">
                    <svg width="22" height="22" viewBox="0 0 22 22" fill="none">
                      <rect x="3" y="3" width="16" height="16" rx="2" stroke="currentColor" stroke-width="1.5"/>
                      <path d="M7 7H15M7 11H12M7 15H10" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
                    </svg>
                  </div>
                  <div class="card-content">
                    <h3 class="card-title orbitron-font">Info & Guides</h3>
                    <p class="card-desc orbitron-font">Documentație și ghiduri</p>
                    <div class="card-footer">
                      <span class="card-badge orbitron-font">{{ categoryCounts.info }} pagini</span>
                      <a href="/informatii" class="card-link">
                        <svg width="14" height="14" viewBox="0 0 14 14" fill="none">
                          <path d="M5 3L9 7L5 11" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        </svg>
                      </a>
                    </div>
                  </div>
                </div>

                <div class="wiki-card reveal-element orbitron-font" ref="card5Ref" v-if="categoryCounts.currency > 0">
                  <div class="card-glow"></div>
                  <div class="card-icon">
                    <svg width="22" height="22" viewBox="0 0 22 22" fill="none">
                      <circle cx="11" cy="11" r="7" stroke="currentColor" stroke-width="1.5"/>
                      <path d="M11 7V11L14 13" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
                    </svg>
                  </div>
                  <div class="card-content">
                    <h3 class="card-title orbitron-font">Currency</h3>
                    <p class="card-desc orbitron-font">Fire Coins și Credits</p>
                    <div class="card-footer">
                      <span class="card-badge orbitron-font">{{ categoryCounts.currency }} pagini</span>
                      <a href="/currency" class="card-link">
                        <svg width="14" height="14" viewBox="0 0 14 14" fill="none">
                          <path d="M5 3L9 7L5 11" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        </svg>
                      </a>
                    </div>
                  </div>
                </div>

                <div class="wiki-card reveal-element orbitron-font" ref="card6Ref" v-if="categoryCounts.vip > 0">
                  <div class="card-glow"></div>
                  <div class="card-icon">
                    <svg width="22" height="22" viewBox="0 0 22 22" fill="none">
                      <path d="M11 3L14 8H19L16 12L18 17L11 14L4 17L6 12L3 8H8L11 3Z" stroke="currentColor" stroke-width="1.5"/>
                    </svg>
                  </div>
                  <div class="card-content">
                    <h3 class="card-title orbitron-font">Custom features</h3>
                    <p class="card-desc orbitron-font">Toate caracteristicile personalizate disponibile</p>
                    <div class="card-footer">
                      <span class="card-badge orbitron-font">{{ categoryCounts.vip }} pagini</span>
                      <a href="/market/vip" class="card-link">
                        <svg width="14" height="14" viewBox="0 0 14 14" fill="none">
                          <path d="M5 3L9 7L5 11" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
                        </svg>
                      </a>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="section-divider">
            <span class="divider-line"></span>
            <img src="/icons/wildfire.webp" alt="divider" class="divider-icon" width="24" height="24" />
            <span class="divider-line"></span>
          </div>
        </section>
      </div>
    </main>

    <div class="scroll-indicator" :class="{ 'scroll-visible': showScrollIndicator }">
      <div class="scroll-line">
        <div class="scroll-progress" :style="{ height: scrollProgress + '%' }"></div>
      </div>
      <div class="scroll-sections">
        <a href="#wiki-section" class="scroll-dot orbitron-font" :class="{ 'active': activeSection === 'wiki' }">
          <span class="dot"></span>
          <span class="dot-label">Wiki</span>
        </a>
        <a href="#last-updates" class="scroll-dot orbitron-font" :class="{ 'active': activeSection === 'updates' }">
          <span class="dot"></span>
          <span class="dot-label">Updates</span>
        </a>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, onUnmounted, watch } from 'vue'
import { useData } from 'vitepress'
import LastUpdates from './LastUpdates.vue'
import HomeNavbar from './HomeNavbar.vue'

const { page, theme } = useData()

const isHomePage = computed(() => {
  return page.value.relativePath === 'index.md' || page.value.relativePath === 'index'
})

const isHovered = ref(false)
const isOverlayLoaded = ref(false)
const logoStart = ref(false)
const isBoomActive = ref(false)
let boomTimeout: ReturnType<typeof setTimeout> | null = null

// Titlu 
const displayTitle = ref('')
const displayTitleArray = computed(() => displayTitle.value.split(''))
const titleTyped = ref(false)
const titleOriginal = 'WILDFIRE.RO DOCS'

// Referințe pentru elemente
const lastUpdatesRef = ref<HTMLElement | null>(null)
const wikiSectionRef = ref<HTMLElement | null>(null)
const wikiLeftRef = ref<HTMLElement | null>(null)
const card1Ref = ref<HTMLElement | null>(null)
const card2Ref = ref<HTMLElement | null>(null)
const card3Ref = ref<HTMLElement | null>(null)
const card4Ref = ref<HTMLElement | null>(null)
const card5Ref = ref<HTMLElement | null>(null)
const card6Ref = ref<HTMLElement | null>(null)

// Scroll indicator
const showScrollIndicator = ref(false)
const scrollProgress = ref(0)
const activeSection = ref('')
const scrollOverlayOpacity = ref(0)

// Funcție pentru a determina clasa culorii pentru fiecare caracter în timpul typing-ului
const getCharClass = (index: number) => {
  const text = displayTitle.value
  if (index < 4) return 'wild-white' // "WILD"
  if (index >= 4 && index < 8) return 'fire-orange' // "FIRE"
  if (index === 8) return 'dot-orange' // "."
  if (index >= 9 && index < 11) return 'ro-orange' // "RO"
  if (index >= 11) return 'docs-white' // " DOCS" (inclusiv spațiul)
  return ''
}

// ===== EFFECT DE TYPING =====
// Badge-uri
const featureBadges = [
  { icon: '✓', text: 'Documentatie' },
  { icon: '⚡', text: 'Informatie' },
  { icon: '🛡️', text: 'Features' }
]

const displayBadges = ref(['', '', ''])
const badgesTyped = ref([false, false, false])

// Buton
const displayButton = ref('')
const buttonTyped = ref(false)
const buttonOriginal = 'Ghid de pornire'

// Tag
const displayTag = ref('')
const tagTyped = ref(false)
const tagOriginal = '@Wildfire.ro'

// Wiki section
const displayWikiHeader = ref('')
const wikiHeaderTyped = ref(false)
const wikiHeaderOriginal = 'WIKIPEDIA'

const displayWikiTitle = ref('')
const wikiTitleTyped = ref(false)
const wikiTitleOriginal = 'Enciclopedia WildFire'

const displayWikiDesc = ref('')
const wikiDescTyped = ref(false)
const wikiDescOriginal = 'WildFire Wiki este centrul de cunoștințe al comunității noastre. Aici găsești documentație completă despre toate sistemele, regulamentele și resursele disponibile.'

const displayWikiCTA = ref('')
const wikiCTATyped = ref(false)
const wikiCTAOriginal = 'Explorează tot'

// Searchbar
const displaySearchText = ref('')
const searchSuggestions = [
  'Caută în documentație...',
  'Regulament Jucatori',
  'Informatii',
  'Knifes',
  'Fire Coins',
  'VIP Tiers',
  'Sisteme',
  'Shop',
  'Stats'
]

let currentSuggestionIndex = 0
let searchTypingTimeout: ReturnType<typeof setTimeout> | null = null
let searchPauseTimeout: ReturnType<typeof setTimeout> | null = null

// Funcția pentru titlu
const typeTitle = async () => {
  for (let i = 0; i <= titleOriginal.length; i++) {
    displayTitle.value = titleOriginal.substring(0, i)
    await new Promise(resolve => setTimeout(resolve, 50))
  }
  titleTyped.value = true
}

// Funcția pentru badge-uri
const typeBadges = async () => {
  for (let b = 0; b < featureBadges.length; b++) {
    const text = featureBadges[b].text
    for (let i = 0; i <= text.length; i++) {
      const newBadges = [...displayBadges.value]
      newBadges[b] = text.substring(0, i)
      displayBadges.value = newBadges
      await new Promise(resolve => setTimeout(resolve, 30))
    }
    badgesTyped.value[b] = true
    await new Promise(resolve => setTimeout(resolve, 150))
  }
}

// Funcția pentru buton
const typeButton = async () => {
  for (let i = 0; i <= buttonOriginal.length; i++) {
    displayButton.value = buttonOriginal.substring(0, i)
    await new Promise(resolve => setTimeout(resolve, 40))
  }
  buttonTyped.value = true
}

// Funcția pentru tag
const typeTag = async () => {
  for (let i = 0; i <= tagOriginal.length; i++) {
    displayTag.value = tagOriginal.substring(0, i)
    await new Promise(resolve => setTimeout(resolve, 40))
  }
  tagTyped.value = true
}

// Funcția pentru Wiki section
const typeWikiSection = async () => {
  // Wiki Header
  for (let i = 0; i <= wikiHeaderOriginal.length; i++) {
    displayWikiHeader.value = wikiHeaderOriginal.substring(0, i)
    await new Promise(resolve => setTimeout(resolve, 30))
  }
  wikiHeaderTyped.value = true
  
  // Wiki Title
  await new Promise(resolve => setTimeout(resolve, 150))
  for (let i = 0; i <= wikiTitleOriginal.length; i++) {
    displayWikiTitle.value = wikiTitleOriginal.substring(0, i)
    await new Promise(resolve => setTimeout(resolve, 40))
  }
  wikiTitleTyped.value = true
  
  // Wiki Description
  await new Promise(resolve => setTimeout(resolve, 200))
  for (let i = 0; i <= wikiDescOriginal.length; i++) {
    displayWikiDesc.value = wikiDescOriginal.substring(0, i)
    await new Promise(resolve => setTimeout(resolve, 20))
  }
  wikiDescTyped.value = true
  
  // Wiki CTA
  await new Promise(resolve => setTimeout(resolve, 150))
  for (let i = 0; i <= wikiCTAOriginal.length; i++) {
    displayWikiCTA.value = wikiCTAOriginal.substring(0, i)
    await new Promise(resolve => setTimeout(resolve, 40))
  }
  wikiCTATyped.value = true
}

// Funcția pentru searchbar
const typeSearchNext = async () => {
  const nextText = searchSuggestions[currentSuggestionIndex]
  
  if (displaySearchText.value.length > 0) {
    for (let i = displaySearchText.value.length; i > 0; i--) {
      displaySearchText.value = displaySearchText.value.substring(0, i - 1)
      await new Promise(resolve => setTimeout(resolve, 15))
    }
  }
  
  for (let i = 0; i <= nextText.length; i++) {
    displaySearchText.value = nextText.substring(0, i)
    await new Promise(resolve => setTimeout(resolve, 25))
  }
  
  currentSuggestionIndex = (currentSuggestionIndex + 1) % searchSuggestions.length
  searchPauseTimeout = setTimeout(typeSearchNext, 1000)
}

// Pornește efectele de typing în ordine cronologică
const startTypingEffects = () => {
  // 1. Titlu - începe primul
  setTimeout(() => typeTitle(), 100)
  
  // 2. Badge-uri - după titlu
  setTimeout(() => typeBadges(), 600)
  
  // 3. Buton - după badge-uri
  setTimeout(() => typeButton(), 1200)
  
  // 4. Tag - după buton
  setTimeout(() => typeTag(), 1500)
  
  // 5. Wiki section - după tag
  setTimeout(() => typeWikiSection(), 1900)
  
  // 6. Searchbar - ultimul, după wiki și continuă ciclic
  setTimeout(() => {
    displaySearchText.value = ''
    setTimeout(() => {
      currentSuggestionIndex = 0
      typeSearchNext()
    }, 150)
  }, 1000)
}

// ===== FUNCȚII PENTRU CONFIG =====
const countPagesInSidebar = () => {
  let total = 0
  let categories = 0
  let sections = 0
  
  let infoCount = 0
  let currencyCount = 0
  let systemsCount = 0
  let marketCount = 0
  let regulamenteCount = 0
  let vipCount = 0

  if (theme.value?.sidebar && Array.isArray(theme.value.sidebar)) {
    theme.value.sidebar.forEach((section: any) => {
      categories++
      
      const sectionText = typeof section.text === 'string' 
        ? section.text.toLowerCase() 
        : section.text?.toString().toLowerCase() || ''
      
      if (section.items && Array.isArray(section.items)) {
        section.items.forEach((item: any) => {
          total++
          
          if (sectionText.includes('informații')) infoCount++
          else if (sectionText.includes('currency')) currencyCount++
          else if (sectionText.includes('systems')) systemsCount++
          else if (sectionText.includes('market')) marketCount++
          
          const itemText = typeof item.text === 'string' 
            ? item.text.toLowerCase() 
            : item.text?.toString().toLowerCase() || ''
            
          if (itemText.includes('regulament')) regulamenteCount++
          if (itemText.includes('vip')) vipCount++
          
          if (item.items && Array.isArray(item.items)) {
            sections += item.items.length
          }
        })
      }
    })
  }

  return {
    total, categories, sections,
    infoCount, currencyCount, systemsCount, marketCount, regulamenteCount, vipCount
  }
}

const counts = countPagesInSidebar()

const categoryCounts = computed(() => ({
  info: counts.infoCount,
  currency: counts.currencyCount,
  systems: counts.systemsCount,
  market: counts.marketCount,
  regulamente: counts.regulamenteCount,
  vip: counts.vipCount
}))

// ===== FUNCȚII UTILITARE =====
const openSearch = (e: MouseEvent) => {
  e.preventDefault()
  e.stopPropagation()
  
  const searchButton = document.querySelector('.VPNavBarSearch button')
  if (searchButton) {
    ;(searchButton as HTMLElement).click()
    return
  }
  
  const event = new KeyboardEvent('keydown', {
    key: 'k',
    ctrlKey: true,
    metaKey: true,
    bubbles: true,
    cancelable: true
  })
  window.dispatchEvent(event)
}

const toggleDefaultNavbar = (hide: boolean) => {
  const vitepressNavbar = document.querySelector('.VPNav')
  if (vitepressNavbar) {
    (vitepressNavbar as HTMLElement).style.display = hide ? 'none' : ''
  }
  
  const vitepressSidebar = document.querySelector('.VPSidebar')
  if (vitepressSidebar) {
    (vitepressSidebar as HTMLElement).style.display = hide ? 'none' : ''
  }
  
  const vitepressContent = document.querySelector('.VPContent')
  if (vitepressContent) {
    (vitepressContent as HTMLElement).style.marginTop = hide ? '0' : ''
  }
  
  const localNav = document.querySelector('.VPLocalNav')
  if (localNav) {
    (localNav as HTMLElement).style.display = hide ? 'none' : ''
  }
}

const triggerBoomEffect = () => {
  if (boomTimeout) {
    clearTimeout(boomTimeout)
  }
  isBoomActive.value = true
  boomTimeout = setTimeout(() => {
    isBoomActive.value = false
  }, 500)
}

// ===== FUNCȚII PENTRU SCROLL =====
const isElementInViewport = (el: HTMLElement | null) => {
  if (!el) return false
  const rect = el.getBoundingClientRect()
  const windowHeight = window.innerHeight
  return rect.top < windowHeight * 0.85 && rect.bottom > 0
}

const applyRevealEffect = () => {
  const elements = [
    lastUpdatesRef.value,
    wikiSectionRef.value,
    wikiLeftRef.value,
    card1Ref.value,
    card2Ref.value,
    card3Ref.value,
    card4Ref.value,
    card5Ref.value,
    card6Ref.value
  ].filter(el => el !== null)

  elements.forEach(el => {
    if (isElementInViewport(el)) {
      el.classList.add('revealed')
    } else {
      el.classList.remove('revealed')
    }
  })
}

const handleScroll = () => {
  const scrollY = window.scrollY
  const windowHeight = window.innerHeight
  const documentHeight = document.documentElement.scrollHeight
  
  scrollProgress.value = (scrollY / (documentHeight - windowHeight)) * 100
  showScrollIndicator.value = scrollY > 200
  
  const isDark = document.documentElement.classList.contains('dark')
  if (isDark) {
    scrollOverlayOpacity.value = Math.min(scrollY / 400, 1)
  } else {
    scrollOverlayOpacity.value = Math.min(scrollY / 800, 0.6)
  }
  
  const wikiSection = document.getElementById('wiki-section')
  const updatesSection = document.querySelector('.last-updates-wrapper')
  
  if (wikiSection) {
    const wikiRect = wikiSection.getBoundingClientRect()
    if (wikiRect.top < windowHeight/2 && wikiRect.bottom > 0) {
      activeSection.value = 'wiki'
    } else if (activeSection.value === 'wiki') {
      activeSection.value = ''
    }
  }
  
  if (updatesSection) {
    const updatesRect = updatesSection.getBoundingClientRect()
    if (updatesRect.top < windowHeight/2 && updatesRect.bottom > 0) {
      activeSection.value = 'updates'
    } else if (activeSection.value === 'updates') {
      activeSection.value = ''
    }
  }

  applyRevealEffect()
}

watch(isHomePage, (newValue) => {
  toggleDefaultNavbar(newValue)
}, { immediate: true })

onMounted(() => {
  toggleDefaultNavbar(isHomePage.value)
  
  setTimeout(() => {
    const overlayImg = new Image()
    overlayImg.src = '/wallpaper/da.webp'
    overlayImg.fetchPriority = 'low'
    overlayImg.onload = () => {
      isOverlayLoaded.value = true
    }
  }, 2000)
  
  setTimeout(() => {
    logoStart.value = true
  }, 800)
  
  // Pornește efectele de typing în ordine
  startTypingEffects()
  
  window.addEventListener('scroll', handleScroll)
  handleScroll()
  
  setTimeout(applyRevealEffect, 100)
})

onUnmounted(() => {
  toggleDefaultNavbar(false)
  if (boomTimeout) {
    clearTimeout(boomTimeout)
  }
  if (searchTypingTimeout) {
    clearTimeout(searchTypingTimeout)
  }
  if (searchPauseTimeout) {
    clearTimeout(searchPauseTimeout)
  }
  window.removeEventListener('scroll', handleScroll)
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;500;600;700;800;900&display=swap');

.orbitron-font {
  font-family: 'Orbitron', sans-serif;
  letter-spacing: 0.3px;
}

.wildfire-home {
  position: relative;
  max-width: 1200px;
  margin: 0 auto;
  padding: 100px 24px 80px;
  min-height: 100vh;
  font-family: 'Orbitron', sans-serif;
  letter-spacing: 0.3px;
}

.wildfire-wallpaper,
.wildfire-wallpaper-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: -2;
  object-fit: cover;
  width: 100%;
  height: 100%;
  transition: opacity 1.2s cubic-bezier(0.4, 0, 0.2, 1),
              transform 0.9s cubic-bezier(0.34, 1.56, 0.64, 1),
              filter 0.9s cubic-bezier(0.4, 0, 0.2, 1);
  opacity: 1;
  transform: translateZ(0);
  backface-visibility: hidden;
  user-select: none;
  pointer-events: none;
}

.wildfire-wallpaper-overlay {
  z-index: -1;
  opacity: 0;
  transition: opacity 0.8s cubic-bezier(0.4, 0, 0.2, 1);
}

.wildfire-wallpaper-overlay-loaded {
  opacity: 1 !important;
}

.wildfire-wallpaper-hover {
  transform: scale(1.08) translateZ(0) !important;
  filter: brightness(1.08) contrast(1.08) saturate(1.1);
}

.wildfire-home .wildfire-bg {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: -1;
  pointer-events: none;
  transform: translateZ(0);
}

:not(.dark) .wildfire-home .wildfire-bg {
  background: rgba(255, 255, 255, 0.7);
  backdrop-filter: blur(2px);
}

:not(.dark) .wildfire-home .wildfire-gradient {
  position: absolute;
  inset: 0;
  background: linear-gradient(to bottom, rgba(220, 38, 38, 0.06), rgba(249, 115, 22, 0.03), rgba(255, 255, 255, 0.7));
}

.wildfire-home .wildfire-grid-primary,
.wildfire-home .wildfire-grid-secondary,
.wildfire-home .wildfire-scanlines {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

:not(.dark) .wildfire-home .wildfire-grid-primary {
  opacity: 0.08;
  background-image: 
    linear-gradient(to right, rgba(255, 69, 0, 0.15) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(255, 69, 0, 0.15) 1px, transparent 1px);
  background-size: 40px 40px;
}

:not(.dark) .wildfire-home .wildfire-grid-secondary {
  opacity: 0.05;
  background-image: 
    linear-gradient(to right, rgba(255, 100, 0, 0.1) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(255, 100, 0, 0.1) 1px, transparent 1px);
  background-size: 20px 20px;
}

:not(.dark) .wildfire-home .wildfire-scanlines {
  opacity: 0.02;
  background-image: repeating-linear-gradient(0deg, rgba(0,0,0,0.05) 0px, rgba(0,0,0,0.05) 1px, transparent 1px, transparent 2px);
  background-size: 100% 2px;
}

.wildfire-home .wildfire-spotlight-tl,
.wildfire-home .wildfire-spotlight-br {
  position: absolute;
  width: 384px;
  height: 384px;
  border-radius: 50%;
  filter: blur(48px);
  opacity: 0;
  transition: opacity 0.3s ease;
  transform: translateZ(0);
  backface-visibility: hidden;
}

:not(.dark) .wildfire-home .wildfire-spotlight-tl {
  top: 0;
  left: 25%;
  background: rgba(220, 38, 38, 0.06);
}

:not(.dark) .wildfire-home .wildfire-spotlight-br {
  bottom: 0;
  right: 25%;
  background: rgba(249, 115, 22, 0.04);
}

.wildfire-wallpaper-hover ~ .wildfire-bg .wildfire-spotlight-tl,
.wildfire-wallpaper-hover ~ .wildfire-bg .wildfire-spotlight-br {
  opacity: 1;
  animation: spotlight-pulse 2.2s ease-in-out infinite;
}

.wildfire-wallpaper-hover ~ .wildfire-bg .wildfire-spotlight-br {
  animation-delay: 0.3s;
}

.wildfire-wallpaper-hover ~ .wildfire-bg .wildfire-grid-primary {
  animation: grid-pulse 2s ease-in-out infinite;
}

.wildfire-wallpaper-hover ~ .wildfire-bg .wildfire-grid-secondary {
  animation: grid-pulse-slow 2.5s ease-in-out infinite;
}

@keyframes grid-pulse {
  0% { opacity: 0.08; }
  50% { opacity: 0.2; }
  100% { opacity: 0.08; }
}

@keyframes grid-pulse-slow {
  0% { opacity: 0.05; }
  50% { opacity: 0.15; }
  100% { opacity: 0.05; }
}

@keyframes spotlight-pulse {
  0% { 
    opacity: 0.2; 
    transform: scale(1) translateZ(0);
    filter: blur(48px);
  }
  50% { 
    opacity: 0.5; 
    transform: scale(1.2) translateZ(0);
    filter: blur(52px);
  }
  100% { 
    opacity: 0.2; 
    transform: scale(1) translateZ(0);
    filter: blur(48px);
  }
}

.dark .wildfire-home .wildfire-bg {
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(2px);
}

.dark .wildfire-home .wildfire-gradient {
  position: absolute;
  inset: 0;
  background: linear-gradient(to bottom, rgba(69, 10, 10, 0.4), rgba(30, 15, 0, 0.3), rgba(0, 0, 0, 0.7));
}

.dark .wildfire-home .wildfire-grid-primary {
  opacity: 0.15;
  background-image: 
    linear-gradient(to right, rgba(255, 69, 0, 0.2) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(255, 69, 0, 0.2) 1px, transparent 1px);
  background-size: 40px 40px;
}

.dark .wildfire-home .wildfire-grid-secondary {
  opacity: 0.1;
  background-image: 
    linear-gradient(to right, rgba(255, 140, 0, 0.15) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(255, 140, 0, 0.15) 1px, transparent 1px);
  background-size: 20px 20px;
}

.dark .wildfire-home .wildfire-scanlines {
  opacity: 0.05;
  background-image: repeating-linear-gradient(0deg, rgba(0,0,0,0.8) 0px, rgba(0,0,0,0.8) 1px, transparent 1px, transparent 2px);
  background-size: 100% 2px;
}

.dark .wildfire-home .wildfire-spotlight-tl {
  top: 0;
  left: 25%;
  background: rgba(220, 38, 38, 0.2);
}

.dark .wildfire-home .wildfire-spotlight-br {
  bottom: 0;
  right: 25%;
  background: rgba(234, 88, 12, 0.2);
}

.dark .wildfire-wallpaper-hover ~ .wildfire-bg .wildfire-grid-primary {
  animation: grid-pulse-dark 2s ease-in-out infinite;
}

.dark .wildfire-wallpaper-hover ~ .wildfire-bg .wildfire-grid-secondary {
  animation: grid-pulse-slow-dark 2.5s ease-in-out infinite;
}

.dark .wildfire-wallpaper-hover ~ .wildfire-bg .wildfire-spotlight-tl,
.dark .wildfire-wallpaper-hover ~ .wildfire-bg .wildfire-spotlight-br {
  animation: spotlight-pulse-dark 2.2s ease-in-out infinite;
}

@keyframes grid-pulse-dark {
  0% { opacity: 0.15; }
  50% { opacity: 0.28; }
  100% { opacity: 0.15; }
}

@keyframes grid-pulse-slow-dark {
  0% { opacity: 0.1; }
  50% { opacity: 0.22; }
  100% { opacity: 0.1; }
}

@keyframes spotlight-pulse-dark {
  0% { 
    opacity: 0.2; 
    transform: scale(1) translateZ(0);
    filter: blur(48px);
  }
  50% { 
    opacity: 0.6; 
    transform: scale(1.2) translateZ(0);
    filter: blur(52px);
  }
  100% { 
    opacity: 0.2; 
    transform: scale(1) translateZ(0);
    filter: blur(48px);
  }
}

.scroll-overlay {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    to bottom,
    transparent 0%,
    rgba(0, 0, 0, 0.1) 15%,
    rgba(0, 0, 0, 0.2) 40%,
    rgba(0, 0, 0, 0.3) 70%,
    rgba(0, 0, 0, 0.4) 100%
  );
  pointer-events: none;
  z-index: 1;
  transition: opacity 0.1s ease;
  opacity: 0;
}

.dark .scroll-overlay {
  background: linear-gradient(
    to bottom,
    transparent 0%,
    rgba(0, 0, 0, 0.7) 15%,
    rgba(0, 0, 0, 0.95) 40%,
    rgba(0, 0, 0, 0.98) 70%,
    rgba(0, 0, 0, 1) 100%
  );
}

.wildfire-home .wildfire-hero {
  position: relative;
  z-index: 10;
  width: 100%;
  transform: translateZ(0);
}

.wildfire-home .wildfire-container {
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.community-tag {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
  padding: 6px 14px;
  background: rgba(255, 69, 0, 0.05);
  border: 1px solid rgba(255, 69, 0, 0.2);
  border-radius: 40px;
  font-size: 12px;
  opacity: 0;
  animation: fadeIn 0.4s ease 0.2s forwards;
}

.dark .community-tag {
  background: rgba(255, 69, 0, 0.1);
  border-color: rgba(255, 69, 0, 0.3);
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}

.tag-badge {
  color: #ff4500;
  font-weight: 700;
}

.tag-year {
  color: var(--vp-c-text-2);
}

.wildfire-home .wildfire-logo-container {
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 15px;
  width: 150px;
  height: 150px;
  cursor: pointer;
}

.wildfire-home .wildfire-logo-glow {
  position: absolute;
  width: 100%;
  height: 100%;
  background: radial-gradient(circle, rgba(255, 69, 0, 0.2) 0%, transparent 70%);
  border-radius: 50%;
  filter: blur(25px);
  opacity: 0;
  transition: opacity 0.3s ease;
  pointer-events: none;
  z-index: 1;
}

.wildfire-home .wildfire-logo-glow.boom-active {
  animation: boomGlow 0.5s ease-out forwards;
}

@keyframes boomGlow {
  0% { opacity: 0; transform: scale(1); }
  40% { opacity: 0.5; transform: scale(1.2); }
  100% { opacity: 0; transform: scale(1); }
}

.logo-boom-ring {
  position: absolute;
  width: 100%;
  height: 100%;
  border: 2px solid rgba(255, 69, 0, 0.4);
  border-radius: 50%;
  opacity: 0;
  transform: scale(1);
  pointer-events: none;
  z-index: 2;
}

.logo-boom-ring.boom-active {
  animation: boomRing 0.5s ease-out forwards;
  animation-delay: calc(0.05s * var(--i));
}

@keyframes boomRing {
  0% { opacity: 0; transform: scale(1); border-width: 2px; }
  40% { opacity: 0.5; transform: scale(1.3); border-width: 1.5px; }
  100% { opacity: 0; transform: scale(1); border-width: 1px; }
}

.wildfire-home .wildfire-logo {
  width: 140px;
  height: 140px;
  object-fit: contain;
  position: relative;
  z-index: 3;
  cursor: pointer;
  filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.15));
  opacity: 0;
}

.logo-start {
  animation: startSpin 0.7s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

@keyframes startSpin {
  0% { transform: scale(0.2) rotate(-360deg); opacity: 0; }
  50% { transform: scale(1.2) rotate(25deg); opacity: 1; }
  80% { transform: scale(0.95) rotate(-5deg); opacity: 1; }
  100% { transform: scale(1) rotate(0deg); opacity: 1; }
}

.wildfire-home .wildfire-logo.boom-active {
  animation: logoBoom 0.5s cubic-bezier(0.4, 0, 0.2, 1) forwards;
}

@keyframes logoBoom {
  0% { transform: scale(1) rotate(0deg); }
  25% { transform: scale(1.2) rotate(15deg); }
  50% { transform: scale(1.25) rotate(-10deg); }
  75% { transform: scale(1.05) rotate(5deg); }
  100% { transform: scale(1) rotate(0deg); }
}

.wildfire-home .wildfire-logo:hover {
  animation: logoHover 0.4s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
  opacity: 1 !important;
}

@keyframes logoHover {
  0% { transform: scale(1) rotate(0deg); }
  40% { transform: scale(1.15) rotate(12deg); }
  70% { transform: scale(1.05) rotate(-4deg); }
  100% { transform: scale(1.1) rotate(3deg); }
}

/* Titlu cu litere mari */
.wildfire-title {
  font-size: 48px;
  font-weight: 800;
  margin: 0 0 8px 0;
  letter-spacing: -0.02em;
  line-height: 1.1;
  text-align: center;
  display: inline-block;
  text-transform: uppercase;
}

/* Culori pentru typing */
.wild-white {
  color: #ffffff;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.fire-orange {
  color: #ff4500;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.dot-orange {
  color: #ff4500;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.ro-orange {
  color: #ff4500;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
}

.docs-white {
  color: #f0f0f0;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
  font-weight: 700;
}

/* Gradient pentru FIRE după typing - efect de foc lent și subtil */
.fire-gradient-text {
  background: linear-gradient(
    135deg,
    #ff4500 0%,
    #ff5722 30%,
    #ff4500 70%,
    #ff5722 100%
  );
  background-size: 300% 300%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  animation: fireFlicker 8s ease-in-out infinite;
  text-shadow: 0 0 5px rgba(255, 69, 0, 0.3);
  position: relative;
  display: inline-block;
}

.fire-gradient-text::after {
  content: 'FIRE';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    135deg,
    rgba(255, 69, 0, 0.15) 0%,
    rgba(255, 87, 34, 0.15) 50%,
    rgba(255, 69, 0, 0.15) 100%
  );
  filter: blur(6px);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  z-index: -1;
  animation: fireGlow 8s ease-in-out infinite;
}

/* Gradient pentru RO după typing - același efect lent și subtil */
.ro-gradient-text {
  background: linear-gradient(
    135deg,
    #ff4500 0%,
    #ff5722 30%,
    #ff4500 70%,
    #ff5722 100%
  );
  background-size: 300% 300%;
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  animation: roFlicker 8s ease-in-out infinite;
  text-shadow: 0 0 5px rgba(255, 69, 0, 0.3);
  position: relative;
  display: inline-block;
}

.ro-gradient-text::after {
  content: 'RO';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    135deg,
    rgba(255, 69, 0, 0.15) 0%,
    rgba(255, 87, 34, 0.15) 50%,
    rgba(255, 69, 0, 0.15) 100%
  );
  filter: blur(6px);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
  z-index: -1;
  animation: roGlow 8s ease-in-out infinite;
}

@keyframes fireFlicker {
  0% {
    background-position: 0% 50%;
    text-shadow: 0 0 5px rgba(255, 69, 0, 0.3);
  }
  25% {
    background-position: 50% 50%;
    text-shadow: 0 0 8px rgba(255, 87, 34, 0.4);
  }
  50% {
    background-position: 100% 50%;
    text-shadow: 0 0 10px rgba(255, 69, 0, 0.5);
  }
  75% {
    background-position: 50% 50%;
    text-shadow: 0 0 8px rgba(255, 87, 34, 0.4);
  }
  100% {
    background-position: 0% 50%;
    text-shadow: 0 0 5px rgba(255, 69, 0, 0.3);
  }
}

@keyframes roFlicker {
  0% {
    background-position: 0% 50%;
    text-shadow: 0 0 5px rgba(255, 69, 0, 0.3);
  }
  25% {
    background-position: 50% 50%;
    text-shadow: 0 0 8px rgba(255, 87, 34, 0.4);
  }
  50% {
    background-position: 100% 50%;
    text-shadow: 0 0 10px rgba(255, 69, 0, 0.5);
  }
  75% {
    background-position: 50% 50%;
    text-shadow: 0 0 8px rgba(255, 87, 34, 0.4);
  }
  100% {
    background-position: 0% 50%;
    text-shadow: 0 0 5px rgba(255, 69, 0, 0.3);
  }
}

@keyframes fireGlow {
  0% {
    opacity: 0.15;
    filter: blur(4px);
  }
  50% {
    opacity: 0.3;
    filter: blur(6px);
  }
  100% {
    opacity: 0.15;
    filter: blur(4px);
  }
}

@keyframes roGlow {
  0% {
    opacity: 0.15;
    filter: blur(4px);
  }
  50% {
    opacity: 0.3;
    filter: blur(6px);
  }
  100% {
    opacity: 0.15;
    filter: blur(4px);
  }
}

.dark .wild-white,
.dark .docs-white {
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.5);
}

.dark .fire-orange,
.dark .dot-orange,
.dark .ro-orange {
  text-shadow: 0 2px 8px rgba(0, 0, 0, 0.5);
}

.dark .fire-gradient-text {
  text-shadow: 0 0 20px rgba(255, 69, 0, 0.7);
}

.dark .ro-gradient-text {
  text-shadow: 0 0 10px rgba(255, 69, 0, 0.4);
}

.feature-badges {
  display: flex;
  gap: 12px;
  justify-content: center;
  margin-bottom: 25px;
  flex-wrap: wrap;
  opacity: 0;
  animation: fadeIn 0.4s ease 0.5s forwards;
}

.feature-badge {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 5px 12px;
  background: var(--vp-c-bg-soft);
  border: 1px solid var(--vp-c-divider);
  border-radius: 30px;
  font-size: 13px;
  transition: all 0.2s ease;
}

.feature-badge:hover {
  border-color: #ff4500;
  transform: translateY(-2px);
}

.feature-icon {
  color: #ff4500;
  font-size: 14px;
}

.feature-text {
  color: var(--vp-c-text-1);
}

.wildfire-button-wrapper {
  display: flex;
  justify-content: center;
  margin-bottom: 30px;
  opacity: 0;
  animation: fadeIn 0.4s ease 0.6s forwards;
}

.wildfire-button {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  background: linear-gradient(135deg, #ff4500, #ff5722);
  color: white;
  padding: 12px 28px;
  border-radius: 40px;
  font-weight: 600;
  font-size: 16px;
  text-decoration: none;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(255, 69, 0, 0.25);
  border: none;
}

.wildfire-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(255, 69, 0, 0.35);
  gap: 14px;
}

.wildfire-button-arrow {
  font-size: 18px;
  transition: transform 0.2s ease;
}

.wildfire-button:hover .wildfire-button-arrow {
  transform: translateX(4px);
}

/* ===== SEARCHBAR STYLING ===== */
.home-search {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
  width: 100%;
  opacity: 0;
  animation: fadeIn 0.4s ease 0.7s forwards;
  position: relative;
}

.home-search-button {
  display: flex;
  align-items: center;
  justify-content: space-between;
  width: 100%;
  max-width: 400px;
  padding: 12px 20px;
  background: #ffffff;
  border: 2px solid transparent;
  border-radius: 40px;
  color: #1a1a1a;
  font-size: 14px;
  font-weight: 500;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
  height: 46px;
  position: relative;
  overflow: hidden;
}

.home-search-button::before {
  content: '';
  position: absolute;
  top: -2px;
  left: -2px;
  right: -2px;
  bottom: -2px;
  background: linear-gradient(45deg, #ff4500, #ff8c00, #ff4500, #ff8c00);
  background-size: 300% 300%;
  border-radius: 42px;
  opacity: 0;
  transition: opacity 0.3s ease;
  z-index: -1;
  animation: borderRotate 3s ease infinite;
}

.home-search-button:hover::before {
  opacity: 1;
}

.home-search-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(255, 69, 0, 0.2);
  background: #ffffff;
}

@keyframes borderRotate {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.dark .home-search-button {
  background: #1a1a1a;
  border: 1px solid #333333;
  color: #ffffff;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
}

.dark .home-search-button:hover {
  background: #1a1a1a;
  box-shadow: 0 8px 20px rgba(255, 69, 0, 0.3);
}

.search-icon {
  width: 15px;
  height: 15px;
  margin-right: 6px;
  filter: brightness(0.3);
}

.dark .search-icon {
  filter: brightness(0) invert(1);
}

.home-search-text {
  flex: 1;
  text-align: left;
  font-size: 13px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  color: inherit;
  display: flex;
  align-items: center;
}

.home-search-shortcut {
  font-size: 10px;
  font-weight: 600;
  padding: 3px 8px;
  border-radius: 5px;
  margin-left: 6px;
  background: #f0f0f0;
  color: #333333;
  border: 1px solid #dddddd;
}

.dark .home-search-shortcut {
  background: #2a2a2a;
  color: #ffffff;
  border: none;
}

.home-search-button:hover .home-search-shortcut {
  background: #ff4500;
  color: #ffffff;
  border-color: #ff4500;
}

.last-updates-wrapper {
  width: 100%;
  margin-bottom: 30px;
}

.wiki-showcase {
  width: 100%;
  margin: 20px 0 40px;
}

.section-divider {
  display: flex;
  align-items: center;
  gap: 16px;
  margin: 30px 0;
}

.divider-line {
  flex: 1;
  height: 1px;
  background: linear-gradient(90deg, transparent, rgba(255, 69, 0, 0.3), rgba(255, 69, 0, 0.5), rgba(255, 69, 0, 0.3), transparent);
}

.divider-icon {
  font-size: 20px;
  color: #ff4500;
  filter: drop-shadow(0 0 8px rgba(255, 69, 0, 0.5));
}

.wiki-layout {
  display: grid;
  grid-template-columns: 1fr 2fr;
  gap: 40px;
  margin: 30px 0;
}

.wiki-left {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.wiki-header {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.wiki-left-line {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 12px;
}

.wiki-left-line .line {
  width: 30px;
  height: 1px;
  background: linear-gradient(90deg, #ff4500, #ff4500, transparent);
}

.wiki-left-line .text {
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 2px;
  color: #ff4500;
  text-transform: uppercase;
}

.wiki-title {
  font-size: 32px;
  font-weight: 700;
  margin: 0;
  color: white;
  line-height: 1.2;
}

.wiki-description {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.wiki-description p {
  font-size: 15px;
  line-height: 1.7;
  color: rgba(255, 255, 255, 0.8);
  margin: 0;
}

.text-highlight {
  color: #ff4500;
  font-weight: 600;
}

.wiki-cta {
  margin-top: 16px;
}

.wiki-cta-link {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 10px 24px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 40px;
  color: white;
  font-weight: 600;
  font-size: 14px;
  text-decoration: none;
  transition: all 0.2s ease;
}

.wiki-cta-link:hover {
  background: #ff4500;
  color: white;
  gap: 14px;
  border-color: #ff4500;
}

.wiki-cta-link svg {
  transition: transform 0.2s ease;
}

.wiki-cta-link:hover svg {
  transform: translateX(4px);
}

.wiki-right {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.wiki-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
}

.wiki-card {
  position: relative;
  background: rgba(255, 255, 255, 0.03);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(255, 255, 255, 0.05);
  border-radius: 16px;
  padding: 16px;
  transition: all 0.2s ease;
  overflow: hidden;
  display: flex;
  flex-direction: column;
  height: 100%;
}

.wiki-card:hover {
  transform: translateY(-2px);
  border-color: rgba(255, 69, 0, 0.3);
  background: rgba(255, 255, 255, 0.05);
}

.card-glow {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  background: radial-gradient(circle, rgba(255, 69, 0, 0.1) 0%, transparent 70%);
  opacity: 0;
  transition: all 0.3s ease;
  pointer-events: none;
  z-index: 0;
}

.wiki-card:hover .card-glow {
  width: 200px;
  height: 200px;
  opacity: 1;
  transform: translate(-50%, -50%);
}

.card-icon {
  width: 36px;
  height: 36px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 10px;
  color: #ff4500;
  margin-bottom: 12px;
  transition: all 0.2s ease;
  position: relative;
  z-index: 2;
}

.wiki-card:hover .card-icon {
  background: #ff4500;
  color: white;
}

.card-content {
  position: relative;
  z-index: 2;
  display: flex;
  flex-direction: column;
  gap: 8px;
  flex: 1;
}

.card-title {
  font-size: 16px;
  font-weight: 600;
  margin: 0;
  color: white;
}

.card-desc {
  font-size: 12px;
  line-height: 1.4;
  color: rgba(255, 255, 255, 0.5);
  margin: 0;
  flex: 1;
}

.card-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: 16px;
  padding-top: 12px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  min-height: 40px;
}

.card-badge {
  font-size: 10px;
  font-weight: 600;
  padding: 4px 10px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 20px;
  color: rgba(255, 255, 255, 0.7);
  border: 1px solid rgba(255, 255, 255, 0.1);
  line-height: 1;
  display: inline-flex;
  align-items: center;
}

.card-link {
  width: 32px;
  height: 32px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 50%;
  color: white;
  transition: all 0.2s ease;
  border: 1px solid rgba(255, 255, 255, 0.1);
  padding: 0;
  line-height: 1;
  flex-shrink: 0;
}

.card-link svg {
  width: 14px;
  height: 14px;
  display: block;
}

.card-link:hover {
  background: #ff4500;
  transform: translateX(3px);
  border-color: transparent;
}

/* ===== EFFECT DE TYPING ===== */
.typing-text {
  position: relative;
  display: inline-block;
}

.cursor {
  display: inline-block;
  width: 2px;
  height: 1.2em;
  background-color: #ff4500;
  margin-left: 2px;
  animation: blink 0.8s infinite;
  vertical-align: middle;
}

@keyframes blink {
  0%, 50% { opacity: 1; }
  51%, 100% { opacity: 0; }
}

.typing-complete .cursor {
  display: none;
}

.reveal-element {
  opacity: 0;
  transform: translateY(30px);
  transition: all 0.6s cubic-bezier(0.4, 0, 0.2, 1);
}

.reveal-element.revealed {
  opacity: 1;
  transform: translateY(0);
}

.wiki-left.revealed {
  transition-delay: 0.1s;
}

.wiki-card:nth-child(1).revealed {
  transition-delay: 0.2s;
}

.wiki-card:nth-child(2).revealed {
  transition-delay: 0.3s;
}

.wiki-card:nth-child(3).revealed {
  transition-delay: 0.4s;
}

.wiki-card:nth-child(4).revealed {
  transition-delay: 0.5s;
}

.wiki-card:nth-child(5).revealed {
  transition-delay: 0.6s;
}

.wiki-card:nth-child(6).revealed {
  transition-delay: 0.7s;
}

.last-updates-wrapper.revealed {
  transition-delay: 0.8s;
}

.delayed-tag {
  opacity: 0;
  animation: fadeInTag 0.5s ease 1.2s forwards;
}

@keyframes fadeInTag {
  from { 
    opacity: 0; 
    transform: translateY(10px); 
  }
  to { 
    opacity: 1; 
    transform: translateY(0); 
  }
}

.scroll-indicator {
  position: fixed;
  right: 20px;
  top: 50%;
  transform: translateY(-50%);
  z-index: 100;
  display: flex;
  gap: 12px;
  align-items: center;
  opacity: 0;
  transition: opacity 0.3s ease;
  pointer-events: none;
}

.scroll-indicator.scroll-visible {
  opacity: 1;
  pointer-events: auto;
}

.scroll-line {
  width: 2px;
  height: 80px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 2px;
  overflow: hidden;
}

.scroll-progress {
  width: 100%;
  background: #ff4500;
  transition: height 0.1s ease;
  border-radius: 2px;
}

.scroll-sections {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.scroll-dot {
  display: flex;
  align-items: center;
  gap: 8px;
  text-decoration: none;
  opacity: 0.5;
  transition: opacity 0.2s ease;
}

.scroll-dot:hover,
.scroll-dot.active {
  opacity: 1;
}

.scroll-dot .dot {
  width: 6px;
  height: 6px;
  background: rgba(255, 255, 255, 0.5);
  border-radius: 50%;
  transition: all 0.2s ease;
}

.scroll-dot.active .dot {
  background: #ff4500;
  box-shadow: 0 0 10px #ff4500;
  transform: scale(1.2);
}

.scroll-dot .dot-label {
  font-size: 11px;
  color: white;
  opacity: 0;
  transform: translateX(-5px);
  transition: all 0.2s ease;
  white-space: nowrap;
}

.scroll-dot:hover .dot-label,
.scroll-dot.active .dot-label {
  opacity: 1;
  transform: translateX(0);
}

@media (max-width: 1024px) {
  .wiki-layout {
    grid-template-columns: 1fr;
    gap: 30px;
  }
  
  .wiki-left {
    text-align: center;
    align-items: center;
  }
  
  .wiki-header {
    align-items: center;
  }
  
  .scroll-indicator {
    display: none;
  }
}

@media (max-width: 768px) {
  .wildfire-home {
    padding: 80px 20px 60px;
  }
  
  .wildfire-title {
    font-size: 42px;
  }
  
  .wiki-title {
    font-size: 28px;
  }
  
  .wiki-row {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .wildfire-home {
    padding: 70px 16px 50px;
  }
  
  .wildfire-title {
    font-size: 32px;
  }
  
  .wiki-title {
    font-size: 24px;
  }
  
  .wiki-row {
    grid-template-columns: 1fr;
  }
  
  .home-search-button {
    width: 44px !important;
    min-width: 44px !important;
    max-width: 44px !important;
    padding: 0;
    justify-content: center;
    height: 44px;
  }
  
  .home-search-text,
  .home-search-shortcut {
    display: none;
  }
  
  .search-icon {
    margin-right: 0;
  }
}
</style>