<template>
  <div class="spell-calculator-container">
    <div class="main-card">
      <!-- Header and Class Selector integrated -->
      <div class="header-row">
        <h2 class="title">Calculadora de Hechizos <span style="color:#64748b">-</span></h2>
        <div class="in-title-selector">
          <button 
            v-for="cls in Object.keys(classesMap)" 
            :key="cls"
            :class="['mini-class-btn', { active: selectedClass === cls }]"
            @click="selectClass(cls)"
            :title="capitalize(cls)"
          >
            <img :src="getClassIcon(cls)" alt="" class="mini-class-icon" />
          </button>
        </div>
      </div>

      <!-- Main Layout -->
      <div v-if="selectedClass">
        <!-- Calculation Summary -->
        <div class="calculation-summary">
          <div class="summary-item">
            <span class="summary-label">
              Nivel del Jugador
              <input type="number" class="player-lvl-input" v-model.number="playerLevel" min="1" max="200" @keydown="preventNonDigit" />
            </span>
            <span class="summary-value">{{ availablePoints }} pts</span>
          </div>
          <div class="summary-item">
            <span class="summary-label">Puntos Invertidos Totales</span>
            <span class="summary-value">{{ totalInvestedPoints }}</span>
          </div>
          <div class="summary-item highlight">
            <span class="summary-label">Pergaminos Necesarios</span>
            <span class="summary-value">{{ scrollsNeeded }}</span>
          </div>
          <div class="summary-item">
            <span class="summary-label">
              Precio Pergamino
            </span>
            <div class="price-input-wrapper">
              <img src="../data/images/spellparcho.svg" alt="" class="price-scroll-icon" />
              <input 
                type="text" 
                class="scroll-price-input" 
                :value="scrollPriceStr"
                @input="formatScrollPrice"
                @keydown="preventNonDigit"
                placeholder="0"
              />
            </div>
          </div>
          <div class="summary-item total-cost-item">
            <span class="summary-label">Costo Total (Kamas)</span>
            <span class="summary-value kamas-value">{{ formatNumber(totalKamasCost) }}</span>
          </div>
          <div class="summary-actions">
            <button class="btn-reset" @click="resetLevels">Reiniciar todo</button>
          </div>
        </div>

        <div class="content-layout">
          
          <!-- Left Column: Spell List -->
          <div class="spell-list-pane">
            <h3 class="pane-title">Hechizos</h3>
            <div class="spells-grid">
              <div 
                v-for="spell in currentSpells" 
                :key="spell.id"
                :class="['spell-list-item', { active: selectedSpell && selectedSpell.id === spell.id }]"
                @click="selectSpell(spell)"
              >
                <div class="spell-list-left">
                  <img :src="getIconPath(spell.icono)" alt="" class="spell-icon-thumb" />
                  <div class="spell-list-info">
                    <span class="spell-list-name">{{ spell.nombre }}</span>
                    <span class="spell-list-stats">
                      {{ spell.niveles['nivel_' + (investedSpells[selectedClass][spell.id] || 1)].coste_pa }} PA &nbsp; 
                      {{ spell.niveles['nivel_' + (investedSpells[selectedClass][spell.id] || 1)].alcance }} AL
                    </span>
                  </div>
                </div>
                <div class="spell-list-right">
                  <div class="spell-list-level-info">
                    <span class="spell-list-niv">Niv. {{ investedSpells[selectedClass][spell.id] }}</span>
                    <span v-if=" investedSpells[selectedClass][spell.id] < 6 " class="spell-list-cost">Coste sig. nivel: {{ investedSpells[selectedClass][spell.id] }}</span>
                  </div>
                  <div class="spell-list-actions">
                    <button class="lvl-action-btn minus" title="Reducir nivel" @click="decrementSpellLevel($event, spell.id)">-</button>
                    <button class="lvl-action-btn plus" title="Aumentar nivel" @click="incrementSpellLevel($event, spell.id)">+</button>
                  </div>
                </div>
              </div>
            </div>
          </div>

        <!-- Right Column: Spell Details -->
        <div class="spell-details-pane" v-if="selectedSpell">
          <div class="details-header">
            <div class="details-header-left">
              <img :src="getIconPath(selectedSpell.icono)" alt="" class="spell-icon-large" />
              <div class="details-title-group">
                <h4 class="spell-name-large">{{ selectedSpell.nombre }}</h4>
                <div class="spell-req-level">Nivel requerido: {{ currentLevelData.nivel_requerido }}</div>
              </div>
            </div>
            
            <div class="details-header-right">
              <div class="level-selector">
                <span class="level-label">Niveles del hechizo: </span>
                <button 
                  v-for="lvl in 6" 
                  :key="lvl"
                  :class="['lvl-btn', { active: selectedSpellLevel === lvl }]"
                  @click="selectedSpellLevel = lvl"
                >
                  {{ lvl }}
                </button>
              </div>
              <div class="spell-basic-stats">
                <div>{{ currentLevelData.alcance }} AL</div>
                <div>{{ currentLevelData.coste_pa }} PA</div>
              </div>
            </div>
          </div>
          
          <div class="spell-description">
            <p>{{ selectedSpell.descripcion }}</p>
            <p><strong>Estado(s) requerido(s):</strong> Ninguno</p>
            <p><strong>Estado(s) prohibido(s):</strong> Ninguno</p>
          </div>
          
          <div class="spell-effects-section">
            <h5 class="section-title">Efectos</h5>
            <div class="effects-tabs">
              <button 
                :class="['effect-tab', { active: activeTab === 'normales' }]" 
                @click="activeTab = 'normales'"
              >Normales</button>
              <button 
                :class="['effect-tab', { active: activeTab === 'criticos', disabled: !currentLevelData.efectos_criticos || currentLevelData.efectos_criticos.length === 0 }]"
                @click="(currentLevelData.efectos_criticos && currentLevelData.efectos_criticos.length > 0) ? activeTab = 'criticos' : null"
              >Críticos</button>
            </div>
            <div class="effects-list">
              <template v-if="activeTab === 'normales'">
                <div v-for="(ef, index) in currentLevelData.efectos" :key="'n_'+index" class="effect-row">
                  <img v-if="getEffectIcon(ef)" :src="getEffectIcon(ef)" class="effect-list-img-icon" alt="" />
                  <span v-else class="effect-list-icon" style="color:#d32f2f">▲</span>
                  <span>{{ ef }}</span>
                </div>
              </template>
              <template v-else-if="activeTab === 'criticos' && currentLevelData.efectos_criticos">
                <div v-for="(efc, index) in currentLevelData.efectos_criticos" :key="'c_'+index" class="effect-row critical">
                  <img v-if="getEffectIcon(efc)" :src="getEffectIcon(efc)" class="effect-list-img-icon" alt="" />
                  <span v-else class="effect-list-icon" style="color:#d32f2f">▲</span>
                  <span>{{ efc }} (Crítico)</span>
                </div>
              </template>
            </div>
          </div>
          
          <div class="spell-other-stats">
            <h5 class="section-title">Otras características</h5>
            <div class="stats-grid">
              <div class="stat-row">
                <span>Probabilidad de golpe crítico</span>
                <span class="stat-val">{{ currentLevelData.probabilidad_critico || '-' }}</span>
              </div>
              <div class="stat-row">
                <span>Probabilidad de fallo</span>
                <span class="stat-val">{{ currentLevelData.probabilidad_fallo || '-' }}</span>
              </div>
            </div>
          </div>
          
        </div>
        
          <div class="spell-details-pane empty" v-else>
            <p>Selecciona un hechizo para ver sus detalles</p>
          </div>
          
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'

// Import all JSONs statically for now, alternatively you could fetch them dynamically.
import aniripsa from '../data/clases/aniripsa_spells.json'
import anutrof from '../data/clases/anutrof_spells.json'
import feca from '../data/clases/feca_spells.json'
import ocra from '../data/clases/ocra_spells.json'
import osamodas from '../data/clases/osamodas_spells.json'
import pandawa from '../data/clases/pandawa_spells.json'
import sacrogrito from '../data/clases/sacrogrito_spells.json'
import sadida from '../data/clases/sadida_spells.json'
import sram from '../data/clases/sram_spells.json'
import xelor from '../data/clases/xelor_spells.json'
import yopuka from '../data/clases/yopuka_spells.json'
import zurcarak from '../data/clases/zurcarak_spells.json'

const classesMap = {
  aniripsa, anutrof, feca, ocra, osamodas, 
  pandawa, sacrogrito, sadida, sram, xelor, yopuka, zurcarak
}

import scrollPrices from '../data/prix/precios_scroll.json'

const selectedClass = ref(null)
const currentSpells = ref([])
const selectedSpell = ref(null)
const selectedSpellLevel = ref(1)
const activeTab = ref('normales')

// Tracking invested points
// Structure: { 'feca': { 1: 1, 2: 6, ... }, ... }
const investedSpells = ref({})
const playerLevel = ref(200)
const scrollPriceRaw = ref(scrollPrices.Hechizos ? scrollPrices.Hechizos.pequeno : 150000)

watch(playerLevel, (newVal) => {
  if (newVal > 200) {
    playerLevel.value = 200
  } else if (newVal !== '' && newVal < 1) {
    playerLevel.value = 1
  }
})

const availablePoints = computed(() => {
  let lvl = typeof playerLevel.value === 'number' ? playerLevel.value : 1
  return Math.max(0, lvl - 1)
})

const formatNumber = (num) => {
  if (!num && num !== 0) return ''
  return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".")
}

const parseNumber = (str) => {
  if (!str) return 0
  return parseInt(str.toString().replace(/\./g, ''), 10) || 0
}

const scrollPriceStr = computed({
  get() { return formatNumber(scrollPriceRaw.value) },
  set(val) { scrollPriceRaw.value = parseNumber(val) }
})

const formatScrollPrice = (e) => {
  let val = e.target.value.replace(/\D/g, '')
  scrollPriceRaw.value = parseInt(val) || 0
}

const preventNonDigit = (e) => {
  if (
    ['Backspace', 'Delete', 'Tab', 'Escape', 'Enter', 'ArrowLeft', 'ArrowRight', 'ArrowUp', 'ArrowDown', 'Home', 'End'].includes(e.key) ||
    e.ctrlKey || e.metaKey || e.altKey
  ) {
    return
  }
  if (!/^\d$/.test(e.key)) {
    e.preventDefault()
  }
}

const initInvestedClass = (cls) => {
  if (!investedSpells.value[cls]) {
    investedSpells.value[cls] = {}
    classesMap[cls].forEach(s => {
      investedSpells.value[cls][s.id] = 1
    })
  }
}

const loadPlayerLevel = () => {
  try {
    const savedState = localStorage.getItem('dofusCalcState')
    if (savedState) {
      const parsed = JSON.parse(savedState)
      if (parsed.nivelObjetivo) {
        playerLevel.value = parsed.nivelObjetivo
      }
    }
  } catch (e) {}
}

onMounted(() => {
  loadPlayerLevel()
  selectClass('feca')
})

const capitalize = (str) => {
  if (!str) return ''
  return str.charAt(0).toUpperCase() + str.slice(1)
}

const selectClass = (cls) => {
  selectedClass.value = cls
  currentSpells.value = classesMap[cls]
  selectedSpell.value = null
  initInvestedClass(cls)
}

const selectSpell = (spell) => {
  selectedSpell.value = spell
  selectedSpellLevel.value = investedSpells.value[selectedClass.value][spell.id] || 1
  activeTab.value = 'normales'
}

const incrementSpellLevel = (e, spellId) => {
  e.stopPropagation()
  const cls = selectedClass.value
  if (investedSpells.value[cls][spellId] < 6) {
    investedSpells.value[cls][spellId]++
    // Sync active view
    if (selectedSpell.value && selectedSpell.value.id === spellId) {
      selectedSpellLevel.value = investedSpells.value[cls][spellId]
    }
  }
}

const decrementSpellLevel = (e, spellId) => {
  e.stopPropagation()
  const cls = selectedClass.value
  if (investedSpells.value[cls][spellId] > 1) {
    investedSpells.value[cls][spellId]--
    // Sync active view
    if (selectedSpell.value && selectedSpell.value.id === spellId) {
      selectedSpellLevel.value = investedSpells.value[cls][spellId]
    }
  }
}

const resetLevels = () => {
  if (!selectedClass.value) return
  const cls = selectedClass.value
  Object.keys(investedSpells.value[cls]).forEach(k => {
    investedSpells.value[cls][k] = 1
  })
  if (selectedSpell.value) {
    selectedSpellLevel.value = 1
  }
}

const costMap = { 1: 0, 2: 1, 3: 3, 4: 6, 5: 10, 6: 15 }

const totalInvestedPoints = computed(() => {
  if (!selectedClass.value) return 0
  let total = 0
  const clsLevels = investedSpells.value[selectedClass.value]
  if (clsLevels) {
    for (const [id, level] of Object.entries(clsLevels)) {
      total += costMap[level] || 0
    }
  }
  return total
})

const scrollsNeeded = computed(() => {
  return Math.max(0, totalInvestedPoints.value - availablePoints.value)
})

const totalKamasCost = computed(() => {
  return scrollsNeeded.value * scrollPriceRaw.value
})

const currentLevelData = computed(() => {
  if (!selectedSpell.value) return {}
  const lvlKey = `nivel_${selectedSpellLevel.value}`
  return selectedSpell.value.niveles[lvlKey] || {}
})

// Since we cannot load dynamic SVG paths easily if they don't exist yet, 
// using a placeholder function to map icon paths.
const getIconPath = (iconName) => {
  if (!iconName) return ''
  // Vite requires static strings or explicit globs for new URL() 
  // We can try to use standard templating.
  const cleanPath = iconName.replace('icons/', '').replace('.svg', '')
  
  try {
    return new URL(`../data/icons/${cleanPath}.svg`, import.meta.url).href
  } catch (e) {
    return ''
  }
}

const getClassIcon = (cls) => {
  const iconMap = {
    'aniripsa': 'Eniripsa',
    'anutrof': 'Enutrof',
    'feca': 'Feca',
    'ocra': 'Cra',
    'osamodas': 'Osamodas',
    'pandawa': 'Pandawa',
    'sacrogrito': 'Sacrieur',
    'sadida': 'Sadida',
    'sram': 'Sram', // Assuming Sram.svg exists or will be added soon
    'xelor': 'Xelor',
    'yopuka': 'Iop',
    'zurcarak': 'Ecaflip'
  }
  const iconName = iconMap[cls] || 'Feca'
  try {
    return new URL(`../data/images/${iconName}.svg`, import.meta.url).href
  } catch (e) {
    return ''
  }
}

const getEffectIcon = (effectText) => {
  if (!effectText) return null
  const t = effectText.toLowerCase()
  
  let iconName = null
  
  // Specific checks first
  if (t.includes('daños') && t.includes('fuego')) iconName = 'FireDamage'
  else if (t.includes('daños') && t.includes('tierra')) iconName = 'EarthDamage'
  else if (t.includes('daños') && t.includes('agua')) iconName = 'WaterDamage'
  else if (t.includes('daños') && t.includes('aire')) iconName = 'AirDamage'
  else if (t.includes('daños') && t.includes('neutral')) iconName = 'NeutralDamage'
  else if (t.includes('pdv devueltos') || t.includes('pdv rendidos')) iconName = 'so'
  else if (t.includes('curas') || t.includes('cura')) iconName = 'so'
  else if (t.includes('vitalidad') || (t.includes('aument') && t.includes('vida'))) iconName = 'Vita'
  else if (t.includes('crítico') && !t.includes('probabilidad')) iconName = 'cc' // Except base probabilities
  else if (t.includes('criaturas') || t.includes('criatura')) iconName = 'ic'
  else if (t.includes('alcance')) iconName = 'po'
  else if (t.includes('reenvío de daños') || t.includes('reenvio de daños')) iconName = 'rdmg'
  else if (t.includes('fuerza')) iconName = 'EarthBonus'
  else if (t.includes('inteligencia')) iconName = 'FireBonus'
  else if (t.includes('suerte')) iconName = 'WaterBonus'
  else if (t.includes('agilidad')) iconName = 'AirBonus'
  else if (t.includes('sabiduría') || t.includes('sabiduria')) iconName = 'Wisdom'
  else if (t.includes('pa') || t.includes('puntos de acción')) iconName = 'pa'
  else if (t.includes('pm') || t.includes('puntos de movimiento')) iconName = 'pm'
  else if (t.includes('poder')) iconName = 'pu'
  else if (t.includes('daños')) iconName = 'dmg' // Fallback for general damage missing element text

  if (iconName) {
    try {
      return new URL(`../data/spellicons/${iconName}.svg`, import.meta.url).href
    } catch (e) {
      return null
    }
  }
  return null
}
</script>

<style scoped>
.spell-calculator-container {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  min-height: calc(100vh - 80px);
  padding: 2rem 3rem;
  box-sizing: border-box;
  font-family: 'Outfit', sans-serif;
}

.main-card {
  width: 100%;
  max-width: 1200px;
  background: rgba(15, 23, 42, 0.7);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border: 1px solid rgba(14, 165, 233, 0.2);
  border-radius: 20px;
  padding: 2.5rem;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
  min-height: auto;
}

.header-row {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1.5rem;
  margin-bottom: 2rem;
}

.title {
  color: #fff;
  font-size: 2rem;
  margin: 0;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.in-title-selector {
  display: flex;
  gap: 0.4rem;
  background: rgba(30, 41, 59, 0.6);
  padding: 0.5rem;
  border-radius: 12px;
  border: 1px solid rgba(14, 165, 233, 0.3);
}

.mini-class-btn {
  background: transparent;
  border: 1px solid transparent;
  padding: 0.3rem;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.mini-class-btn:hover {
  background: rgba(14, 165, 233, 0.2);
}

.mini-class-btn.active {
  background: linear-gradient(135deg, rgba(14, 165, 233, 0.5), rgba(56, 189, 248, 0.3));
  border-color: #38bdf8;
}

.mini-class-icon {
  width: 26px;
  height: 26px;
  filter: drop-shadow(0 2px 2px rgba(0,0,0,0.5));
}

.calculation-summary {
  display: flex;
  justify-content: space-around;
  align-items: center;
  background: rgba(30, 41, 59, 0.95);
  backdrop-filter: blur(8px);
  border: 1px solid rgba(14, 165, 233, 0.5);
  border-radius: 12px;
  padding: 1rem;
  margin-bottom: 1.5rem;
  color: #e2e8f0;
  position: sticky;
  top: 70px;
  z-index: 100;
  box-shadow: 0 10px 25px rgba(0,0,0,0.5);
}

.summary-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.3rem;
}

.summary-label {
  font-size: 0.85rem;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.summary-value {
  font-size: 1.5rem;
  font-weight: bold;
  color: #fff;
}

.summary-item.highlight .summary-value {
  color: #38bdf8;
}

.player-lvl-input {
  width: 50px;
  background: rgba(2, 6, 23, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #38bdf8;
  padding: 0.2rem 0.4rem;
  border-radius: 6px;
  font-family: 'Outfit', sans-serif;
  font-weight: bold;
  text-align: center;
  margin-left: 0.3rem;
  outline: none;
}

.price-input-wrapper {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-top: 0.3rem;
}

.price-scroll-icon {
  width: 24px;
  height: 24px;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.3));
}

.scroll-price-input {
  width: 100px;
  background: rgba(2, 6, 23, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #34d399;
  padding: 0.4rem;
  border-radius: 6px;
  font-family: 'Outfit', sans-serif;
  font-weight: bold;
  text-align: center;
  outline: none;
  font-size: 1.1rem;
}

.scroll-price-input:focus, .player-lvl-input:focus {
  border-color: #0ea5e9;
  box-shadow: 0 0 0 2px rgba(14, 165, 233, 0.2);
}

.total-cost-item {
  border-left: 2px solid rgba(255,255,255,0.1);
  padding-left: 1rem;
}

.kamas-value {
  color: #34d399 !important;
  text-shadow: 0 0 8px rgba(52, 211, 153, 0.4);
}

.btn-reset {
  background: rgba(244, 63, 94, 0.15);
  border: 1px solid rgba(244, 63, 94, 0.4);
  color: #fda4af;
  padding: 0.5rem 1rem;
  border-radius: 8px;
  cursor: pointer;
  font-family: 'Outfit', sans-serif;
  font-weight: 600;
  transition: all 0.2s;
}

.btn-reset:hover {
  background: rgba(244, 63, 94, 0.3);
  color: #fff;
}

/* Layout */
.content-layout {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  background: #fdf5e6; 
  border-radius: 12px;
  padding: 1.5rem;
  border: 4px solid #d4c3ae;
  color: #4b3e2a;
}

/* Left Pane */
.spell-list-pane {
  background: #e2d4bd;
  border: 2px solid #cdba9f;
  padding: 1rem;
  border-radius: 8px;
  height: auto;
}

.pane-title {
  text-align: center;
  margin-top: 0;
  color: #5c4e3a;
  border-bottom: 1px solid #cdba9f;
  padding-bottom: 0.5rem;
}

.spells-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.5rem;
}

.spell-list-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.5rem;
  background: #cfbeaa;
  border: 1px solid #bba68d;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.2s;
}

.spell-list-item:hover {
  background: #d6cab6;
}

.spell-list-item.active {
  background: #e38661;
  color: white;
  border-color: #c0633b;
}

.spell-list-left {
  display: flex;
  align-items: center;
  gap: 0.8rem;
}

.spell-list-info {
  display: flex;
  flex-direction: column;
}

.spell-list-name {
  font-weight: bold;
  font-size: 0.95rem;
}

.spell-list-stats {
  font-size: 0.8rem;
  color: #5c4e3a;
}
.spell-list-item.active .spell-list-stats {
  color: #ffe6d5;
}

.spell-list-right {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.spell-list-level-info {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  font-size: 0.75rem;
}

.spell-list-niv {
  font-weight: bold;
}

.spell-list-cost {
  font-size: 0.65rem;
  color: #7b6f63;
}

.spell-list-item.active .spell-list-cost {
  color: #ffe6d5;
}

.spell-list-actions {
  display: flex;
  flex-direction: column;
  gap: 0.15rem;
}

.lvl-action-btn {
  width: 18px;
  height: 18px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bold;
  color: white;
  border: 1px solid rgba(0,0,0,0.3);
  border-radius: 3px;
  cursor: pointer;
  font-size: 0.9rem;
  line-height: 1;
}

.lvl-action-btn.plus {
  background: linear-gradient(to bottom, #f37d2f, #d55310);
}
.lvl-action-btn.plus:hover {
  background: linear-gradient(to bottom, #ff944d, #ea661e);
}
.lvl-action-btn.minus {
  background: linear-gradient(to bottom, #9ca3af, #6b7280);
}
.lvl-action-btn.minus:hover {
  background: linear-gradient(to bottom, #d1d5db, #9ca3af);
}

.spell-icon-thumb {
  width: 32px;
  height: 32px;
  background: #a44848;
  border: 1px solid #fff;
  border-radius: 4px;
}

.spell-name-thumb {
  font-size: 0.85rem;
  font-weight: 600;
}

/* Right Pane */
.spell-details-pane {
  background: #faf6f0;
  border: 2px solid #fff;
  box-shadow: 0 0 10px rgba(0,0,0,0.1);
  padding: 1.5rem;
  border-radius: 4px;
  display: flex;
  flex-direction: column;
}

.spell-details-pane.empty {
  justify-content: center;
  align-items: center;
  color: #948875;
  font-style: italic;
}

.details-header {
  display: flex;
  justify-content: space-between;
  border-bottom: 1px solid #e2d4bd;
  padding-bottom: 1rem;
  margin-bottom: 1rem;
}

.details-header-left {
  display: flex;
  gap: 1rem;
}

.spell-icon-large {
  width: 64px;
  height: 64px;
  background: #a44848;
  border: 2px solid #fff;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2);
  border-radius: 4px;
}

.spell-name-large {
  margin: 0 0 0.5rem 0;
  font-size: 1.2rem;
  color: #3f3121;
}

.spell-req-level {
  font-size: 0.9rem;
  color: #6a5840;
}

.details-header-right {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 0.5rem;
}

.level-selector {
  display: flex;
  align-items: center;
  gap: 0.2rem;
}

.level-label {
  font-size: 0.85rem;
  margin-right: 0.5rem;
}

.lvl-btn {
  background: none;
  border: none;
  font-size: 0.9rem;
  cursor: pointer;
  color: #6a5840;
  padding: 0.2rem 0.4rem;
}

.lvl-btn.active {
  color: #e35f42;
  font-weight: bold;
}

.spell-basic-stats {
  text-align: right;
  font-size: 0.9rem;
  color: #6a5840;
}

.spell-description {
  font-size: 0.9rem;
  line-height: 1.5;
  margin-bottom: 1.5rem;
}

.section-title {
  margin-top: 0;
  margin-bottom: 0.5rem;
  font-size: 1rem;
  color: #4b3e2a;
}

.effects-tabs {
  display: flex;
  margin-bottom: 0.5rem;
}

.effect-tab {
  background: #cfbeaa;
  border: none;
  padding: 0.3rem 1rem;
  font-size: 0.85rem;
  cursor: pointer;
  color: #5c4e3a;
}

.effect-tab.active {
  background: #e2d4bd;
  font-weight: bold;
}

.effect-tab.disabled {
  background: #a69a8b;
  color: #7b6f63;
  cursor: not-allowed;
}

.effects-list {
  background: #e9e0d1;
  padding: 0.5rem;
  border-radius: 4px;
  margin-bottom: 1.5rem;
}

.effect-row {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  font-size: 0.9rem;
  padding: 0.3rem 0;
}

.effect-list-img-icon {
  width: 18px;
  height: 18px;
  object-fit: contain;
}

.stats-grid {
  display: flex;
  flex-direction: column;
  gap: 0.3rem;
  font-size: 0.9rem;
}

.stat-row {
  display: flex;
  justify-content: space-between;
}

.stat-val {
  font-weight: bold;
}

/* Custom Scrollbar for list */
.spell-list-pane::-webkit-scrollbar {
  width: 6px;
}
.spell-list-pane::-webkit-scrollbar-track {
  background: #cdba9f;
}
.spell-list-pane::-webkit-scrollbar-thumb {
  background: #a8947b;
  border-radius: 3px;
}
</style>
