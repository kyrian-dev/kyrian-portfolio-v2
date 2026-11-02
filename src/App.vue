<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref } from 'vue'
import ProjectCard from './components/ProjectCard.vue'
import { categories, projects } from './data/projects.js'
import './assets/portfolio.css'

const menuOpen = ref(false)
const menuButton = ref(null)
const navigationMenu = ref(null)
const activeSection = ref('home')
const theme = ref(document.documentElement.dataset.theme || 'dark')
const query = ref('')
const category = ref('All')
const view = ref('grid')
const scrollProgress = ref(0)
const year = new Date().getFullYear()

const featuredProjects = computed(() => ['Nexora', 'Taskflow Studio', 'Job Application Tracker'].map(title => projects.find(project => project.title === title)).filter(Boolean))
const caseStudyProjects = computed(() => ['Nexora', 'Taskflow Studio'].map(title => projects.find(project => project.title === title)).filter(Boolean))

const skills = [
  'HTML5',
  'CSS3',
  'JavaScript',
  'TypeScript',
  'Vue.js 3',
  'React.js',
  'Tailwind CSS',
  'REST APIs / Fetch',
  'Chrome DevTools',
  'Responsive Design',
  'Git & GitHub'
]

const navigation = [{ id: 'projects', name: 'Projects' }, { id: 'case-studies', name: 'Case Studies' }, { id: 'about', name: 'About' }, { id: 'testimonials', name: 'References' }, { id: 'contact', name: 'Contact' }]
const filteredProjects = computed(() => {
  const search = query.value.trim().toLowerCase()
  return projects.filter(project => (category.value === 'All' || project.category === category.value)
    && [project.title, project.category, project.problem, project.solution, ...project.stack, ...project.features].join(' ').toLowerCase().includes(search))
})

function toggleTheme() {
  theme.value = theme.value === 'dark' ? 'light' : 'dark'
  document.documentElement.dataset.theme = theme.value
  document.querySelector('meta[name="theme-color"]')?.setAttribute('content', theme.value === 'dark' ? '#080d16' : '#f8fafc')
  try { localStorage.setItem('kyrian-theme', theme.value) } catch { /* Keep the toggle usable without storage. */ }
}
function resetFilters() {
  category.value = 'All'
  query.value = ''
}
async function toggleMenu() {
  menuOpen.value = !menuOpen.value
  if (menuOpen.value) {
    await nextTick()
    navigationMenu.value?.querySelector('a')?.focus()
  }
}
async function closeMenuForSection(id) {
  const wasOpen = menuOpen.value
  menuOpen.value = false
  if (wasOpen) {
    await nextTick()
    document.getElementById(id)?.focus({ preventScroll: true })
  }
}
async function closeMenuWithEscape(event) {
  if (event.key !== 'Escape' || !menuOpen.value) return
  menuOpen.value = false
  await nextTick()
  menuButton.value?.focus()
}
let observer
let layoutObserver
let scrollFrame = 0
function updateProgress() {
  if (scrollFrame) return
  scrollFrame = requestAnimationFrame(() => {
    const distance = document.documentElement.scrollHeight - innerHeight
    scrollProgress.value = distance > 0 ? Math.min(1, Math.max(0, scrollY / distance)) : 0
    scrollFrame = 0
  })
}
function observeSections() {
  observer?.disconnect()
  observer = new IntersectionObserver(entries => {
    entries.forEach(entry => { if (entry.isIntersecting) activeSection.value = entry.target.id })
  }, { rootMargin: `-${Math.round(innerHeight * .15)}px 0px -${Math.round(innerHeight * .65)}px 0px` })
  document.querySelectorAll('[data-section]').forEach(section => observer.observe(section))
}
function onResize() {
  observeSections()
  updateProgress()
}
onMounted(() => {
  observeSections()
  layoutObserver = new ResizeObserver(updateProgress)
  layoutObserver.observe(document.getElementById('main'))
  window.addEventListener('scroll', updateProgress, { passive: true })
  window.addEventListener('resize', onResize)
  document.addEventListener('keydown', closeMenuWithEscape)
  updateProgress()
})
onBeforeUnmount(() => {
  observer?.disconnect()
  layoutObserver?.disconnect()
  cancelAnimationFrame(scrollFrame)
  window.removeEventListener('scroll', updateProgress)
  window.removeEventListener('resize', onResize)
  document.removeEventListener('keydown', closeMenuWithEscape)
})
</script>

<template>
  <a class="skip-link" href="#main">Skip to content</a>
  <header class="site-header">
    <div class="reading-progress" :style="{ transform: 'scaleX(' + scrollProgress + ')' }" aria-hidden="true"></div>
    <div class="header-inner">
      <a href="#home" class="brand" aria-label="Kyrian, home" @click="menuOpen = false">
        <span class="brand-mark" aria-hidden="true">KY</span>
        <span class="brand-name">Kyrian.<small>// frontend developer</small></span>
      </a>
      <div class="header-actions">
        <nav ref="navigationMenu" id="main-navigation" :class="{ 'is-open': menuOpen }" aria-label="Main navigation">
          <a v-for="link in navigation" :key="link.id" :href="'#' + link.id" :aria-current="activeSection === link.id ? 'location' : undefined" @click="closeMenuForSection(link.id)">{{ link.name }}</a>
        </nav>
        <button class="icon-button theme-toggle" type="button" @click="toggleTheme" :aria-label="'Switch to ' + (theme === 'dark' ? 'light' : 'dark') + ' theme'" :title="'Switch to ' + (theme === 'dark' ? 'light' : 'dark') + ' theme'">
          <svg v-if="theme === 'dark'" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" aria-hidden="true"><circle cx="12" cy="12" r="4"/><path d="M12 2v2m0 16v2M2 12h2m16 0h2M5 5l1.5 1.5m11 11L19 19M5 19l1.5-1.5m11-11L19 5"/></svg>
          <svg v-else viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" aria-hidden="true"><path d="M20.5 14A8.5 8.5 0 0 1 10 3.5 8.5 8.5 0 1 0 20.5 14Z"/></svg>
        </button>
        <button ref="menuButton" class="icon-button menu-toggle" type="button" :aria-expanded="menuOpen" aria-controls="main-navigation" :aria-label="menuOpen ? 'Close navigation' : 'Open navigation'" @click="toggleMenu">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" aria-hidden="true"><path v-if="menuOpen" d="m6 6 12 12M6 18 18 6"/><path v-else d="M4 7h16M4 12h16M4 17h16"/></svg>
        </button>
      </div>
    </div>
  </header>

  <main id="main" tabindex="-1">
    <section id="home" class="hero section-shell" data-section aria-labelledby="hero-title">
      <div class="availability"><span aria-hidden="true"></span> Open to junior frontend roles &amp; internships</div>
      <h1 id="hero-title">Hi, I'm<br><span class="gradient-text">Kyrian.</span></h1>
      <p class="hero-role">Frontend Development <span>·</span> React <span>·</span> Vue <span>·</span> TypeScript<span class="cursor" aria-hidden="true"></span></p>
      <p class="hero-description">A junior frontend developer based in Stuttgart.<br>I build <strong>responsive, accessible, interactive web applications</strong> with React, Vue and TypeScript.</p>
      <div class="button-row hero-buttons">
        <a class="button button-primary" href="#projects">View Projects <span aria-hidden="true">→</span></a>
        <a class="button" href="#about">About Me</a>
        <a class="button" href="#contact">Contact</a>
        <a class="button" href="https://github.com/kyrian-dev" target="_blank" rel="noopener noreferrer" aria-label="Kyrian's GitHub profile (opens in a new tab)">GitHub <span aria-hidden="true">↗</span></a>
      </div>
      <dl class="hero-stats">
        <div><dt>{{ projects.length }}</dt><dd>Live projects</dd></div>
        <div><dt>Vue 3</dt><dd>Main framework</dd></div>
        <div><dt>ILS</dt><dd>Hamburg training</dd></div>
        <div><dt>Stuttgart</dt><dd>Based in</dd></div>
      </dl>
    </section>

    <section class="flagship section-shell" aria-labelledby="flagship-title">
      <div class="flagship-copy">
        <p class="eyebrow">// Featured build</p>
        <h2 id="flagship-title">Nexora <span class="gradient-text">Project Dashboard</span></h2>
        <p>A React and TypeScript workspace for organising projects, managing tasks, tracking progress and keeping deadlines in view. Data is saved locally in your browser.</p>
        <div class="flagship-stack"><span>React</span><span>TypeScript</span><span>Tailwind CSS</span><span>Vite</span></div>
        <div class="button-row"><a class="button button-primary" href="https://nexora-kyrian4.vercel.app/" target="_blank" rel="noopener noreferrer">Explore Nexora <span aria-hidden="true">↗</span></a><a class="button" href="https://github.com/kyrian-dev/nexora" target="_blank" rel="noopener noreferrer">View source <span aria-hidden="true">↗</span></a></div>
      </div>
      <a class="flagship-visual" href="https://nexora-kyrian4.vercel.app/" target="_blank" rel="noopener noreferrer" aria-label="Open Nexora live dashboard in a new tab"><img src="https://raw.githubusercontent.com/kyrian-dev/nexora/main/screenshots/dashboard.png" alt="Actual Nexora dashboard screenshot showing project management interface" loading="lazy" /></a>
    </section>

    <section class="intro section-shell" aria-labelledby="intro-title">
      <div class="intro-copy">
        <p class="eyebrow">// From idea to interface</p>
        <h2 id="intro-title">Thoughtful interfaces.<br><span class="gradient-text">Practical applications.</span></h2>
        <p>From organising tasks to browsing a storefront, I build interfaces that make everyday actions feel straightforward.</p>
        <div class="button-row"><a class="button button-primary pill-button" href="#projects">Explore My Work <span aria-hidden="true">→</span></a><a class="button pill-button" href="#contact">Get in Touch</a></div>
      </div>
    </section>

    <div class="tech-strip section-shell" aria-label="Technologies I work with">
      <span class="tech-strip-label">Built with tools I know</span>
      
<ul>
  <li><span class="tech-symbol">&lt;/&gt;</span> HTML5</li>
  <li><span class="tech-symbol">#</span> CSS3</li>
  <li><span class="tech-symbol boxed">JS</span> JavaScript</li>
  <li><span class="tech-symbol boxed">TS</span> TypeScript</li>
  <li><span class="tech-symbol vue-symbol">V</span> Vue.js</li>
  <li><span class="tech-symbol">⚛</span> React.js</li>
  <li><span class="tech-symbol">✦</span> Tailwind CSS</li>
  <li><span class="tech-symbol">↔</span> REST APIs / Fetch</li>
  <li><span class="tech-symbol">⌕</span> Chrome DevTools</li>
  <li><span class="tech-symbol">ϟ</span> Vite</li>
  <li><span class="tech-symbol">⌘</span> Git & GitHub</li>
</ul>

    </div>

    <section id="projects" class="section-shell content-section" data-section tabindex="-1" aria-labelledby="projects-title">
      <p class="eyebrow">// Featured projects</p>
      <h2 id="projects-title">Featured Projects</h2>
      <p class="section-description">Three featured applications demonstrating project management, task organisation, and job-search workflows. Explore the demos and source code.</p>
      <div class="project-grid featured-grid"><ProjectCard v-for="project in featuredProjects" :key="project.id" :project="project" /></div>
      <div class="all-projects-heading"><h3>Explore all {{ projects.length }} projects</h3><p>Filter the complete portfolio by category, technology, or feature.</p></div>
      <div class="project-toolbar">
        <div class="search-row">
          <label class="search-box"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.7" aria-hidden="true"><circle cx="10.5" cy="10.5" r="6.5"/><path d="m16 16 5 5"/></svg><span class="sr-only">Search projects</span><input v-model="query" type="search" placeholder="Search by project, feature, or technology..."></label>
          <div class="view-switch" role="group" aria-label="Project layout">
            <button class="icon-button" type="button" :aria-pressed="view === 'grid'" aria-label="Grid view" title="Grid view" @click="view = 'grid'"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" aria-hidden="true"><rect x="4" y="4" width="6" height="6" rx="1"/><rect x="14" y="4" width="6" height="6" rx="1"/><rect x="4" y="14" width="6" height="6" rx="1"/><rect x="14" y="14" width="6" height="6" rx="1"/></svg></button>
            <button class="icon-button" type="button" :aria-pressed="view === 'list'" aria-label="List view" title="List view" @click="view = 'list'"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.6" aria-hidden="true"><path d="M8 6h13M8 12h13M8 18h13M3 6h1M3 12h1M3 18h1"/></svg></button>
          </div>
        </div>
        <div class="filter-row" role="group" aria-label="Filter projects by category"><span class="filter-label">Category:</span><button v-for="item in categories" :key="item" type="button" class="filter-chip" :aria-pressed="category === item" @click="category = item">{{ item }}<span v-if="item === 'All'"> ({{ projects.length }})</span></button></div>
      </div>
      <div class="project-summary"><p aria-live="polite" aria-atomic="true">{{ filteredProjects.length }} {{ filteredProjects.length === 1 ? 'project' : 'projects' }} to explore</p><button v-if="query || category !== 'All'" class="text-button" @click="resetFilters">Reset filters <span aria-hidden="true">↺</span></button><span v-else class="project-demo-note">Personal projects · Live frontend demos</span></div>
      <div v-if="filteredProjects.length" class="project-grid" :class="{ 'list-view': view === 'list' }"><ProjectCard v-for="project in filteredProjects" :key="project.id" :project="project"/></div>
      <div v-else class="empty-state"><h3>No projects found</h3><p>Try another search or choose a different category.</p><button class="button button-primary" @click="resetFilters">Show all projects</button></div>
    </section>

    <section id="case-studies" class="section-shell notes-section" data-section tabindex="-1" aria-labelledby="notes-title">
      <p class="eyebrow">// Behind the interface</p>
      <h2 id="notes-title">Development Case Studies</h2>
      <p class="section-description">A closer look at the problem, approach, implementation and scope of two featured applications.</p>
      <div class="case-study-grid">
        <article v-for="project in caseStudyProjects" :key="project.id" class="case-study-card">
          <img :src="project.image" :alt="project.alt" loading="lazy" />
          <div class="case-study-content">
            <span class="eyebrow">Case study · {{ project.category }}</span>
            <h3>{{ project.title }}</h3>
            <h4>Challenge</h4><p>{{ project.problem }}</p>
            <h4>Approach</h4><p>{{ project.solution }}</p>
            <h4>Implemented features</h4><ul><li v-for="feature in project.features" :key="feature">{{ feature }}</li></ul>
            <p v-if="project.limitation" class="case-study-limitation">{{ project.limitation }}</p>
            <div class="button-row"><a class="button button-primary" :href="project.live" target="_blank" rel="noopener noreferrer">Live demo ↗</a><a class="button" :href="project.source + '#readme'" target="_blank" rel="noopener noreferrer">Code &amp; documentation ↗</a></div>
          </div>
        </article>
      </div>
    </section>

    <section id="about" class="section-shell content-section" data-section tabindex="-1" aria-labelledby="about-title">
      <p class="eyebrow">// Who I am</p><h2 id="about-title">About Me</h2>
      <div class="about-grid">
        <div>
          <h3 class="small-heading">Frontend</h3><ul class="skill-grid"><li v-for="skill in skills.slice(0, 7)" :key="skill">{{ skill }}</li></ul><h3 class="small-heading education-heading">Tools &amp; practices</h3><ul class="skill-grid"><li v-for="skill in skills.slice(7)" :key="skill">{{ skill }}</li></ul>
          
<h3 class="small-heading education-heading">Education & Training</h3>

<div class="timeline">
  <div class="timeline-item">
    <h4>Geprüfter Webentwickler</h4>
    <p class="education-school">ILS Hamburg, Germany</p>
    <p>Professional Training in Web Development</p>
    <p>
      <time datetime="2025-03">March 2025</time> –
      <time datetime="2026-04">April 2026</time>
    </p>
  </div>

  <div class="timeline-item">
    <h4>Frontend Web Development Bootcamp</h4>
    <p class="education-school">Ironhack Berlin, Germany</p>
    <p>Frontend Web Development</p>
    <p>2024</p>
  </div>

  <div class="timeline-item">
    <h4>Higher National Diploma (HND)</h4>
    <p class="education-school">Building Technology, Nigeria</p>
    <p>Higher National Diploma in Building Technology</p>
  </div>
</div>

<div class="location-note">
  <svg viewBox="0 0 24 24" fill="none"
       stroke="currentColor" stroke-width="1.7"
       aria-hidden="true">
    <path d="M20 10c0 6-8 12-8 12S4 16 4 10a8 8 0 1 1 16 0Z"/>
    <circle cx="12" cy="10" r="2.5"/>
  </svg>
  Based in Stuttgart
</div>
</div>

        <div class="about-copy"><h3 class="small-heading">What I bring</h3><p>I'm a junior frontend developer with a strong interest in building modern, responsive, and user-friendly web applications.

After completing my professional web development training at ILS Hamburg, I've gained practical experience through developing real-world-inspired projects, including Nexora, a React and TypeScript project dashboard, alongside a task management application, job application tracker, e-commerce website, and taxi booking demo.

I bring a solid understanding of frontend development, attention to detail, problem-solving skills, and the ability to turn ideas into functional web applications. I focus on writing clean, maintainable code and creating interfaces that work smoothly across different devices.

I'm eager to contribute my technical skills to a professional development team, collaborate with other developers, and continue growing as a frontend developer.</p><div class="button-row"><a class="button button-primary" href="#projects">View My Projects <span aria-hidden="true">→</span></a><a class="button" href="https://www.linkedin.com/in/obiekee-kyrian-ikechukwu-563022308/" target="_blank" rel="noopener noreferrer" aria-label="Kyrian's LinkedIn profile (opens in a new tab)">LinkedIn <span aria-hidden="true">↗</span></a></div></div>
      </div>
    </section>

    <section id="testimonials" class="section-shell content-section testimonials-section" data-section tabindex="-1" aria-labelledby="testimonials-title">
      <p class="eyebrow">// Mentor feedback</p>
      <h2 id="testimonials-title">Testimonials &amp; References</h2>
      <p class="section-description">Feedback from my frontend development training.</p>
      <div class="testimonial-grid">
        <figure class="testimonial-card">
          <span class="testimonial-quote-mark" aria-hidden="true">“</span>
          <blockquote>Kyrian is a dedicated and motivated frontend developer with strong practical skills, attention to detail, and a willingness to learn. His projects demonstrate creativity and great potential for professional growth.</blockquote>
          <figcaption><strong>Sarah Henneberg</strong><span>ILS Hamburg</span></figcaption>
        </figure>
        <figure class="testimonial-card">
          <span class="testimonial-quote-mark" aria-hidden="true">“</span>
          <blockquote>Kyrian approaches technical challenges with enthusiasm and determination. His problem-solving mindset, openness to feedback, and commitment to continuous improvement make him a promising Junior Frontend Developer.</blockquote>
          <figcaption><strong>Mariano Neito</strong><span>Ironhack Berlin</span></figcaption>
        </figure>
      </div>
    </section>

    <section id="contact" class="contact-section" tabindex="-1" data-section aria-labelledby="contact-title"><div class="section-shell contact-inner">
      <div class="availability"><span aria-hidden="true"></span> Open to junior roles &amp; internships</div><h2 id="contact-title">Looking for a <span class="gradient-text">Junior Frontend Developer?</span></h2><p>I'm actively seeking junior frontend developer positions and frontend development internships in Germany. I work with React, Vue.js, JavaScript and TypeScript, and I'm ready to contribute to real-world products, collaborate with experienced developers and continue growing.</p><div class="opportunity-tags"><span>Junior Frontend Developer</span><span>Frontend Internship</span><span>Based in Stuttgart</span></div>
      <a class="contact-email" href="mailto:kyrian.dev.de@gmail.com"><span><small>Email</small><strong>kyrian.dev.de@gmail.com</strong></span><span class="contact-arrow" aria-hidden="true">↗</span></a>
      <div class="button-row contact-buttons"><a class="button" href="https://www.linkedin.com/in/obiekee-kyrian-ikechukwu-563022308/" target="_blank" rel="noopener noreferrer" aria-label="Kyrian's LinkedIn profile (opens in a new tab)">LinkedIn <span aria-hidden="true">↗</span></a><a class="button" href="https://github.com/kyrian-dev" target="_blank" rel="noopener noreferrer" aria-label="Kyrian's GitHub profile (opens in a new tab)">GitHub <span aria-hidden="true">↗</span></a><a class="button button-primary" href="mailto:kyrian.dev.de@gmail.com?subject=Frontend%20Opportunity">Get in Touch <span aria-hidden="true">→</span></a></div>
    </div></section>
  </main>
  <footer class="site-footer"><div class="section-shell footer-inner"><p>© {{ year }} Kyrian. <span>Built with Vue.js · Germany</span></p><div><a href="https://github.com/kyrian-dev" target="_blank" rel="noopener noreferrer">GitHub ↗</a><a href="mailto:kyrian.dev.de@gmail.com">Email</a><a href="#home">Back to top ↑</a></div></div></footer>
</template>
