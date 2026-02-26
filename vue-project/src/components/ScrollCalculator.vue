<template>
  <div class="scroll-calculator-container">
    <div class="glass-card main-card">
      <div class="header-row">
        <h2 class="title">Calculadora de Scroll Optimo</h2>
        <span class="title-separator">-</span>
        <div class="element-selector">
          <button 
            v-for="element in elementList" 
            :key="element.name"
            :class="['element-btn', { active: selectedElement === element.name }]"
            @click="selectElement(element.name)"
          >
            <img :src="element.icon" alt="" class="element-icon" />
            {{ element.name }}
          </button>
        </div>
      </div>

      <div class="prices-section">
        <h3>Precios de los scrolls</h3>
        <div class="input-grid">
          <div class="input-group">
            <div class="scroll-label-container">
              <img :src="getScrollImage('pequeno')" class="scroll-icon" alt="" />
              <label>Pequeños</label>
            </div>
            <div class="kamas-input">
              <span class="currency-symbol">$</span>
              <input type="number" v-model.number="currentPrices.pequeno" @input="savePrices" @keydown="preventComma" />
            </div>
          </div>
          <div class="input-group">
            <div class="scroll-label-container">
              <img :src="getScrollImage('mediano')" class="scroll-icon" alt="" />
              <label>Medianos</label>
            </div>
            <div class="kamas-input">
              <span class="currency-symbol">$</span>
              <input type="number" v-model.number="currentPrices.mediano" @input="savePrices" @keydown="preventComma" />
            </div>
          </div>
          <div class="input-group">
            <div class="scroll-label-container">
              <img :src="getScrollImage('grande')" class="scroll-icon" alt="" />
              <label>Grandes</label>
            </div>
            <div class="kamas-input">
              <span class="currency-symbol">$</span>
              <input type="number" v-model.number="currentPrices.grande" @input="savePrices" @keydown="preventComma" />
            </div>
          </div>
          <div class="input-group">
            <div class="scroll-label-container">
              <img :src="getScrollImage('poderoso')" class="scroll-icon" alt="" />
              <label>Poderosos</label>
            </div>
            <div class="kamas-input">
              <span class="currency-symbol">$</span>
              <input type="number" v-model.number="currentPrices.poderoso" @input="savePrices" @keydown="preventComma" />
            </div>
          </div>
        </div>
      </div>

      <div class="result-section" v-if="optimalResult">
        <h3>Resultado Óptimo</h3>
        <div class="result-grid">
          <div class="result-item">
            <img :src="getScrollImage('pequeno')" class="scroll-icon-result" alt="" />
            <span class="label">Pequeño</span>
            <span class="value">{{ optimalResult.pequeno }}</span>
          </div>
          <div class="result-item">
            <img :src="getScrollImage('mediano')" class="scroll-icon-result" alt="" />
            <span class="label">Mediano</span>
            <span class="value">{{ optimalResult.mediano }}</span>
          </div>
          <div class="result-item">
            <img :src="getScrollImage('grande')" class="scroll-icon-result" alt="" />
            <span class="label">Grande</span>
            <span class="value">{{ optimalResult.grande }}</span>
          </div>
          <div class="result-item">
            <img :src="getScrollImage('poderoso')" class="scroll-icon-result" alt="" />
            <span class="label">Poderoso</span>
            <span class="value">{{ optimalResult.poderoso }}</span>
          </div>
          <div class="result-item highlight">
            <span class="label">Costo Total</span>
            <span class="value kamas">{{ formatKamas(optimalResult.cost) }}</span>
          </div>
        </div>
      </div>
      <div v-else class="result-section no-solution">
        <h3>No es posible alcanzar 101 puntos con estos scrolls.</h3>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import defaultPrices from '../data/prix/precios_scroll.json'
import airIcon from '../data/spellicons/AirBonus.svg'
import vitaIcon from '../data/spellicons/Vita.svg'
import earthIcon from '../data/spellicons/EarthBonus.svg'
import fireIcon from '../data/spellicons/FireBonus.svg'
import waterIcon from '../data/spellicons/WaterBonus.svg'
import wisdomIcon from '../data/spellicons/Wisdom.svg'

const elementList = [
  { name: 'Agilidad', icon: airIcon },
  { name: 'Vitalidad', icon: vitaIcon },
  { name: 'Fuerza', icon: earthIcon },
  { name: 'Inteligencia', icon: fireIcon },
  { name: 'Suerte', icon: waterIcon },
  { name: 'Sabiduría', icon: wisdomIcon }
]
const selectedElement = ref(elementList[0].name)

const pricesData = ref(JSON.parse(JSON.stringify(defaultPrices)))
const currentPrices = ref(pricesData.value[selectedElement.value])

const optimalResult = ref(null)

const selectElement = (elName) => {
  selectedElement.value = elName
  currentPrices.value = pricesData.value[elName]
  calculateOptimal()
}

const savePrices = () => {
  pricesData.value[selectedElement.value] = { ...currentPrices.value }
  calculateOptimal()
}

const preventComma = (e) => {
  if (e.key === ',' || e.key === '.') {
    e.preventDefault()
  }
}

const formatKamas = (value) => {
  return '$ ' + value.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".")
}

const scrollImageCache = {}

const getScrollImage = (type) => {
  const elementPrefixMap = {
    'Agilidad': { prefix: 'agi', max: '101' },
    'Vitalidad': { prefix: 'vita', max: '101' },
    'Fuerza': { prefix: 'fo', max: '101' },
    'Inteligencia': { prefix: 'int', max: '101' },
    'Suerte': { prefix: 'cha', max: '101' },
    'Sabiduría': { prefix: 'sasa', max: '100' }
  }
  
  const elementData = elementPrefixMap[selectedElement.value];
  if (!elementData) return '';
  
  let size = '';
  switch(type) {
    case 'pequeno': size = '25'; break;
    case 'mediano': size = '50'; break;
    case 'grande': size = '80'; break;
    case 'poderoso': size = elementData.max; break;
  }
  
  const imgName = `${elementData.prefix}${size}`;
  if (!scrollImageCache[imgName]) {
    scrollImageCache[imgName] = new URL(`../data/images/${imgName}.svg`, import.meta.url).href;
  }
  return scrollImageCache[imgName];
}

const calculateOptimal = () => {
  const p_s = currentPrices.value.pequeno || 0
  const p_m = currentPrices.value.mediano || 0
  const p_g = currentPrices.value.grande || 0
  const p_p = currentPrices.value.poderoso || 0

  const INF = Infinity
  const dp = new Array(102).fill(INF)
  const choice = new Array(102).fill(null)
  
  dp[0] = 0

  for (let i = 0; i <= 100; i++) {
    if (dp[i] === INF) continue

    if (i < 25 && i + 1 <= 101) {
      if (dp[i] + p_s < dp[i + 1]) {
        dp[i + 1] = dp[i] + p_s
        choice[i + 1] = { from: i, type: 'pequeno', amount: 1 }
      }
    }
    if (i < 50 && i + 1 <= 101) {
      if (dp[i] + p_m < dp[i + 1]) {
        dp[i + 1] = dp[i] + p_m
        choice[i + 1] = { from: i, type: 'mediano', amount: 1 }
      }
    }
    if (i < 80 && i + 1 <= 101) {
      if (dp[i] + p_g < dp[i + 1]) {
        dp[i + 1] = dp[i] + p_g
        choice[i + 1] = { from: i, type: 'grande', amount: 1 }
      }
    }
    if (i < 100 && i + 2 <= 101) {
      if (dp[i] + p_p < dp[i + 2]) {
        dp[i + 2] = dp[i] + p_p
        choice[i + 2] = { from: i, type: 'poderoso', amount: 2 }
      }
    }
  }

  if (dp[101] === INF) {
    optimalResult.value = null
    return
  }

  let curr = 101
  const counts = { pequeno: 0, mediano: 0, grande: 0, poderoso: 0 }
  let totalCost = dp[101]

  while (curr > 0) {
    const c = choice[curr]
    if (c) {
      counts[c.type]++
      curr = c.from
    } else {
      break
    }
  }

  optimalResult.value = {
    ...counts,
    cost: totalCost,
    points: 101
  }
}

onMounted(() => {
  calculateOptimal()
})
</script>

<style scoped>
.scroll-calculator-container {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  min-height: calc(100vh - 80px); /* Account for header */
  padding: 2rem 3rem;
  box-sizing: border-box;
}

.glass-card {
  background: rgba(15, 23, 42, 0.7);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border: 1px solid rgba(14, 165, 233, 0.2);
  border-radius: 24px;
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5), 0 0 20px rgba(14, 165, 233, 0.1);
}

.main-card {
  width: 100%;
  max-width: 100%;
  padding: 2.5rem;
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
  text-shadow: 0 0 10px rgba(14, 165, 233, 0.5);
}

.title-separator {
  color: rgba(14, 165, 233, 0.5);
  font-size: 2.5rem;
  font-weight: 300;
}

.element-selector {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 0.5rem;
  margin: 0;
}

.element-btn {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background: rgba(30, 41, 59, 0.8);
  border: 1px solid rgba(14, 165, 233, 0.3);
  color: #94a3b8;
  padding: 0.6rem 1.2rem;
  border-radius: 12px;
  font-family: 'Outfit', sans-serif;
  font-weight: 600;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
}

.element-icon {
  width: 24px;
  height: 24px;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5));
}

.element-btn:hover {
  background: rgba(14, 165, 233, 0.1);
  color: #e2e8f0;
}

.element-btn.active {
  background: linear-gradient(135deg, rgba(14, 165, 233, 0.4), rgba(56, 189, 248, 0.4));
  border-color: #38bdf8;
  color: #fff;
  box-shadow: 0 0 15px rgba(14, 165, 233, 0.4);
}

.prices-section h3, .result-section h3 {
  color: #38bdf8;
  border-bottom: 1px solid rgba(14, 165, 233, 0.3);
  padding-bottom: 0.8rem;
  margin-bottom: 1.5rem;
  font-size: 1.3rem;
  text-shadow: 0 0 8px rgba(14, 165, 233, 0.3);
}

.input-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.5rem;
  margin-bottom: 3rem;
}

.scroll-label-container {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  margin-bottom: 0.5rem;
}

.scroll-icon {
  width: 24px;
  height: 24px;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.3));
}

.input-group label {
  display: block;
  color: #cbd5e1;
  font-size: 0.95rem;
  margin-bottom: 0;
}

.kamas-input {
  display: flex;
  align-items: center;
  background: rgba(15, 23, 42, 0.8);
  border: 1px solid rgba(14, 165, 233, 0.4);
  border-radius: 10px;
  padding: 0.5rem 1rem;
  transition: all 0.3s;
}

.kamas-input:focus-within {
  border-color: #38bdf8;
  box-shadow: 0 0 10px rgba(56, 189, 248, 0.3);
}

.currency-symbol {
  color: #38bdf8;
  font-weight: bold;
  margin-right: 0.5rem;
}

.kamas-input input {
  background: transparent;
  border: none;
  color: #fff;
  font-size: 1.1rem;
  width: 100%;
  font-family: 'Outfit', sans-serif;
  outline: none;
}

.kamas-input input::-webkit-outer-spin-button,
.kamas-input input::-webkit-inner-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.result-section {
  background: rgba(15, 23, 42, 0.5);
  border-radius: 16px;
  padding: 2rem;
  border: 1px solid rgba(56, 189, 248, 0.2);
}

.no-solution {
  text-align: center;
  color: #ef4444;
}

.result-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(130px, 1fr));
  gap: 1.5rem;
}

.result-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: rgba(30, 41, 59, 0.6);
  padding: 1.2rem;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.scroll-icon-result {
  width: 32px;
  height: 32px;
  margin-bottom: 0.5rem;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5));
}

.result-item.highlight {
  background: linear-gradient(135deg, rgba(14, 165, 233, 0.2), rgba(2, 6, 23, 0.4));
  border-color: rgba(56, 189, 248, 0.4);
}

.result-item .label {
  color: #94a3b8;
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.result-item .value {
  color: #fff;
  font-size: 1.5rem;
  font-weight: 700;
}

.result-item .value.kamas {
  color: #eab308;
  text-shadow: 0 0 10px rgba(234, 179, 8, 0.4);
}

@media (max-width: 200px) {

  .title {
    font-size: 1.5rem;
  }
}
</style>
