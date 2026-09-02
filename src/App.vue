<script setup lang="ts">
import { computed, ref } from 'vue'
import { Line } from 'vue-chartjs'
import { Chart as ChartJS, CategoryScale, Filler, Legend, LinearScale, LineElement, PointElement, Tooltip } from 'chart.js'
import overview from './data/overview.json'

ChartJS.register(CategoryScale, Filler, Legend, LinearScale, LineElement, PointElement, Tooltip)

const selectedYear = ref(overview.period.year)
const selectedMonth = ref(overview.period.month)
const activeTab = ref('Overview')
const healthView = ref('Impact')
const showPrototypeNotice = ref(false)
const selectedPartner = ref(overview.diveDeep.partners[0].name)
const selectedGoal = ref(overview.diveDeep.goals[0].label)

const activePartner = computed(() => overview.diveDeep.partners.find((partner) => partner.name === selectedPartner.value) ?? overview.diveDeep.partners[0])
const activeGoal = computed(() => overview.diveDeep.goals.find((goal) => goal.label === selectedGoal.value) ?? overview.diveDeep.goals[0])

const chartData = computed(() => ({
  labels: overview.health.labels,
  datasets: [
    { label: 'Impact delivered', data: overview.health.impact, borderColor: '#087f73', backgroundColor: 'rgba(8, 127, 115, 0.12)', fill: true, tension: 0.38, pointRadius: 4, pointBackgroundColor: '#087f73' },
    { label: 'Expected trajectory', data: overview.health.target, borderColor: '#9da9a7', borderDash: [6, 6], tension: 0.38, pointRadius: 0 },
  ],
}))

const partnerChartData = computed(() => ({
  labels: overview.health.labels,
  datasets: [
    { label: 'Partner progress', data: activePartner.value.impact, borderColor: '#2876a8', backgroundColor: 'rgba(40, 118, 168, 0.12)', fill: true, tension: 0.38, pointRadius: 4, pointBackgroundColor: '#2876a8' },
    { label: 'Expected trajectory', data: [40, 45, 50, 55, 60, 65], borderColor: '#9da9a7', borderDash: [6, 6], tension: 0.38, pointRadius: 0 },
  ],
}))

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  interaction: { mode: 'index' as const, intersect: false },
  plugins: { legend: { display: false }, tooltip: { padding: 12, displayColors: false } },
  scales: {
    y: { beginAtZero: false, grid: { color: '#e5ece9' }, ticks: { color: '#70807b', callback: (value: string | number) => `${Number(value) / 1000}K` } },
    x: { grid: { display: false }, ticks: { color: '#70807b' } },
  },
}

function prototypeAction() {
  showPrototypeNotice.value = true
  window.setTimeout(() => (showPrototypeNotice.value = false), 3000)
}
</script>

<template>
  <v-app>
    <v-app-bar class="app-bar" elevation="0">
      <div class="brand-mark" aria-hidden="true">P3</div>
      <span class="eyebrow app-bar__label">P301 sustainability hub</span>
      <v-spacer />
         <nav class="nav-tabs" aria-label="Dashboard sections">
           <v-btn variant="text" :class="{ 'nav-tabs__button--active': activeTab === 'Overview' }" @click="activeTab = 'Overview'">Overview</v-btn>
           <v-btn variant="text" :class="{ 'nav-tabs__button--active': activeTab === 'Dive Deep' }" @click="activeTab = 'Dive Deep'">Dive Deep</v-btn>
         </nav>
      <v-btn icon variant="text" class="theme-button" aria-label="Toggle dark mode" @click="prototypeAction"><v-icon icon="mdi-weather-night" /></v-btn>
      <v-avatar color="#ffffff" size="40" class="avatar">J</v-avatar>
    </v-app-bar>

    <v-main><v-container class="page-shell">
      <div class="page-heading"><div><p class="overline">Monday, September 2, 2026</p><h1>{{ activeTab === 'Overview' ? 'Good morning, James' : "Let's see how our partners are doing" }}</h1><p class="subtitle">{{ activeTab === 'Overview' ? 'A clear view of momentum, impact, and the decisions that matter next.' : 'Focus on partner momentum, uncover risks early, and turn data into the next right action.' }}</p></div></div>

      <div class="filter-bar"><div class="filter-controls"><v-select v-model="selectedYear" :items="['FY26', 'FY25']" label="Year" density="compact" hide-details /><v-select v-model="selectedMonth" :items="['September', 'August', 'July', 'June']" label="Month" density="compact" hide-details /><v-select v-if="activeTab === 'Dive Deep'" v-model="selectedPartner" :items="overview.diveDeep.partners.map((partner) => partner.name)" label="Partner" density="compact" hide-details /></div><div class="heading-actions"><v-btn variant="text" color="primary" prepend-icon="mdi-file-export-outline" @click="prototypeAction">Export</v-btn><v-btn variant="text" color="primary" prepend-icon="mdi-share-variant-outline" @click="prototypeAction">Share</v-btn></div><span class="updated">Updated 3 seconds ago</span></div>

      <section v-if="activeTab === 'Overview'" class="insights-section"><div class="section-heading"><div><p class="overline">Signal scan</p><h2>What needs your attention</h2></div><span class="section-note">3 fresh insights</span></div><div class="insight-grid"><v-card v-for="insight in overview.insights" :key="insight.title" class="insight-card" elevation="0" :class="`tone-${insight.tone}`"><div class="insight-icon"><v-icon :icon="insight.icon" size="22" /></div><p class="card-kicker">{{ insight.category }}</p><h3>{{ insight.title }}</h3><p>{{ insight.body }}</p><v-btn variant="text" color="primary" class="insight-action" append-icon="mdi-arrow-right" @click="prototypeAction">{{ insight.action }}</v-btn></v-card></div></section>

      <section v-if="activeTab === 'Overview'"><div class="section-heading"><div><p class="overline">Company pulse</p><h2>Key performance indicators</h2></div><span class="section-note">FY26 targets</span></div><div class="kpi-grid"><v-card v-for="kpi in overview.kpis" :key="kpi.label" class="kpi-card" elevation="0"><div class="kpi-top"><span class="trend">{{ kpi.trend }} this month</span></div><p>{{ kpi.label }}</p><strong>{{ kpi.value }}</strong><span class="target">Target: {{ kpi.target }}</span></v-card></div></section>

      <section v-if="activeTab === 'Overview'" class="dashboard-grid"><v-card class="panel health-panel" elevation="0"><div class="panel-heading"><div><p class="overline">ESG health</p><h2>Impact delivered</h2></div><v-btn-toggle v-model="healthView" density="compact" mandatory class="small-toggle"><v-btn value="Impact">Impact</v-btn><v-btn value="Goals">Goals</v-btn></v-btn-toggle></div><div class="chart-summary"><strong>12.5K</strong><span>tCO₂e avoided <b>↑ 8.4%</b> vs. August</span></div><div class="chart-wrap"><Line :data="chartData" :options="chartOptions" /></div><div class="legend-row"><span><i class="legend-line actual"></i>Actual impact</span><span><i class="legend-line expected"></i>Expected trajectory</span></div></v-card><v-card class="panel strategy-panel" elevation="0"><div class="panel-heading"><div><p class="overline">Strategy progress</p><h2>Moving the plan forward</h2></div><v-icon icon="mdi-arrow-top-right" color="primary" /></div><div class="strategy-list"><div v-for="item in overview.strategy" :key="item.label" class="strategy-item"><div class="strategy-label"><span>{{ item.label }}</span><strong>{{ item.value }}%</strong></div><v-progress-linear :model-value="item.value" :color="item.value > 70 ? 'primary' : 'warning'" height="8" rounded /><small>{{ item.detail }}</small></div></div></v-card></section>

      <section v-if="activeTab === 'Overview'"><v-card class="panel goals-panel" elevation="0"><div class="panel-heading"><div><p class="overline">Portfolio roadmap</p><h2>Goals analysis</h2></div><div class="status-legend"><span><i class="status-dot on-track"></i>On track</span><span><i class="status-dot at-risk"></i>At risk</span></div></div><div class="timeline"><div class="timeline-months"><span v-for="month in ['Jan', 'Apr', 'Jul', 'Oct', 'Jan', 'Apr']" :key="month">{{ month }}</span></div><div v-for="goal in overview.goals" :key="goal.label" class="goal-row"><div class="goal-name"><strong>{{ goal.label }}</strong><span :class="goal.status === 'At risk' ? 'risk-text' : ''">{{ goal.status }} · {{ goal.progress }}%</span></div><div class="goal-track"><div class="goal-bar" :class="`bar-${goal.tone}`" :style="{ left: `${goal.start}%`, width: `${goal.duration}%` }"></div></div></div></div></v-card></section>

      <section v-if="activeTab === 'Overview'" class="playground"><div><p class="overline">Coming into focus</p><h2>Playground</h2><p>Compare partner momentum, prioritize actions, and test how a shift in strategy could change the year ahead.</p></div><v-btn color="primary" variant="outlined" append-icon="mdi-arrow-right" @click="prototypeAction">Open Playground</v-btn></section>

      <template v-if="activeTab === 'Dive Deep'"><section class="partner-summary"><div><p class="overline">Partner profile</p><h2>{{ activePartner.name }}</h2><p>{{ activePartner.sector }} <span class="partner-status" :class="activePartner.status === 'On track' ? 'partner-status--good' : 'partner-status--attention'">{{ activePartner.status }}</span></p></div><div class="partner-score"><span>ESG maturity</span><strong>{{ activePartner.score }}</strong><small>{{ activePartner.change }} since April</small></div></section><section class="deep-insights"><v-card class="goal-nav" elevation="0"><p class="overline">Focus area</p><v-btn v-for="goal in overview.diveDeep.goals" :key="goal.label" :class="{ 'goal-nav__button--active': selectedGoal === goal.label }" variant="text" @click="selectedGoal = goal.label"><v-icon :icon="goal.icon" size="19" />{{ goal.label }}</v-btn></v-card><v-card class="goal-insight" elevation="0"><p class="overline">Insight summary</p><h2>{{ activeGoal.label }} momentum</h2><p>{{ activeGoal.summary }}</p><div class="recommended-action"><v-icon icon="mdi-sparkles" color="primary" size="20" /><div><span>Recommended action</span><strong>{{ activeGoal.action }}</strong></div></div><v-btn color="primary" variant="text" append-icon="mdi-arrow-right" @click="prototypeAction">Open action plan</v-btn></v-card></section><section><div class="section-heading"><div><p class="overline">Partner pulse</p><h2>Key performance indicators</h2></div><span class="section-note">{{ activePartner.name }}</span></div><div class="deep-kpi-grid"><v-card v-for="kpi in overview.diveDeep.kpis" :key="kpi.label" class="deep-kpi" elevation="0"><v-icon :icon="kpi.icon" color="primary" size="22" /><p>{{ kpi.label }}</p><strong>{{ kpi.value }}</strong><span>{{ kpi.detail }}</span></v-card></div></section><section class="dashboard-grid"><v-card class="panel" elevation="0"><div class="panel-heading"><div><p class="overline">Performance trend</p><h2>Impact progress</h2></div><span class="section-note">April - September</span></div><div class="chart-wrap"><Line :data="partnerChartData" :options="chartOptions" /></div><div class="legend-row"><span><i class="legend-line actual"></i>{{ activePartner.name }}</span><span><i class="legend-line expected"></i>Expected trajectory</span></div></v-card><v-card class="panel" elevation="0"><div class="panel-heading"><div><p class="overline">Goal progress</p><h2>Where to focus</h2></div></div><div class="strategy-list"><div v-for="metric in [{ label: 'Renewable energy', value: activePartner.renewables }, { label: 'Waste diversion', value: activePartner.waste }, { label: 'Community engagement', value: activePartner.engagement }]" :key="metric.label" class="strategy-item"><div class="strategy-label"><span>{{ metric.label }}</span><strong>{{ metric.value }}%</strong></div><v-progress-linear :model-value="metric.value" :color="metric.value >= 70 ? 'primary' : 'warning'" height="8" rounded /></div></div></v-card></section></template>
    </v-container></v-main>
    <v-snackbar v-model="showPrototypeNotice" color="secondary">Prototype action ready for the next phase.<template #actions><v-btn variant="text" @click="showPrototypeNotice = false">Close</v-btn></template></v-snackbar>
  </v-app>
</template>
