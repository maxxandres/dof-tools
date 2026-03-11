<template>
  <div class="calculator-container">
    <div class="calculator-header">
      <div class="header-left">
        <div class="header-content">
          <h2 class="title">Recetas</h2>
        </div>

        <div class="search-container">
          <div class="search-box">
            <input 
              v-model="searchQuery" 
              type="text" 
              placeholder="Buscar equipamiento..."
              class="search-input"
              @focus="showDropdown = true"
              @blur="handleBlur"
            />
          </div>
          
          <div v-if="showDropdown && filteredItems.length > 0" class="dropdown">
            <div 
              v-for="item in filteredItems" 
              :key="item.nombre" 
              class="dropdown-item"
              @mousedown="selectItem(item)"
            >
              <div class="dropdown-item-image-container" v-if="item.imagen">
                <img :src="getItemImage(item.imagen)" :alt="item.nombre" class="dropdown-item-image" />
              </div>
              <span class="item-name">{{ item.nombre }}</span>
              <span class="item-level">Nv. {{ item.nivel }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div v-if="selectedItem" class="recipe-content">
      <div class="panels-wrapper">
        <!-- Left Panel: Item Stats -->
        <div class="panel item-details-panel">
          <div class="item-header">
            <div class="item-visuals">
              <div class="item-image-container" v-if="selectedItem.imagen">
                <img :src="getItemImage(selectedItem.imagen)" :alt="selectedItem.nombre" class="item-image" />
              </div>
              <div class="item-quantity-wrapper">
                <span class="qty-label">x</span>
                <input 
                  type="text" 
                  v-model="itemQuantityDisplay" 
                  @input="handleItemQuantityInput"
                  class="item-quantity-input"
                />
              </div>
            </div>
            <div class="item-info">
              <h3 class="item-title">{{ selectedItem.nombre }}</h3>
              <div class="item-level-badge">Nivel {{ selectedItem.nivel }}</div>
            </div>
          </div>
          
          <div class="stats-container">
            <div v-for="(stat, index) in selectedItem.stats" :key="index" class="stat-row">
              <img v-if="getStatIcon(stat)" :src="getStatIcon(stat)" alt="" class="stat-icon" />
              <span class="stat-name">{{ stat }}</span>
            </div>
          </div>
        </div>

        <!-- Right Panel: Recipe Needs -->
        <div class="panel recipe-panel">
          <div class="recipe-header">
            <h3>Receta ({{ selectedItem.recetas ? selectedItem.recetas.length : 0 }} ingredientes)</h3>
          </div>
          
          <div class="ingredients-list" v-if="selectedItem.recetas && selectedItem.recetas.length > 0">
            <div class="ingredient-header">
              <div class="col-ing">Ingrediente</div>
              <div class="col-qty">Cant.</div>
              <div class="col-price">Precio/u</div>
              <div class="col-total">Total</div>
            </div>
            
            <div v-for="ing in selectedItem.recetas" :key="ing.recursoId" class="ingredient-row">
              <div class="col-ing">
                <div class="ing-image-container" v-if="ing.imagen">
                  <img :src="getItemImage(ing.imagen)" :alt="ing.nombre" class="ing-image" />
                </div>
                <span 
                  :class="['ing-name', { 'has-tooltip': ing.drops && ing.drops.length > 0 }]"
                  @mouseenter="e => handleTooltipEnter(e, ing)"
                  @mousemove="handleTooltipMove"
                  @mouseleave="handleTooltipLeave"
                >
                  {{ ing.nombre }}
                </span>
              </div>
              <div class="col-qty">x{{ ing.cantidad * itemQuantity }}</div>
              <div class="col-price input-wrapper">
                <input 
                  type="text" 
                  :value="formatNumber(ingredientPrices[ing.recursoId])" 
                  @input="e => updatePrice(e, 'ingredient', ing.recursoId)"
                  class="price-input" 
                  placeholder="0"
                />
              </div>
              <div class="col-total highlight">{{ formatKamas(calculateIngredientTotal(ing)) }}</div>
            </div>
            
            <div class="recipe-summary">
              <div class="summary-row">
                <span>Costo del Craft:</span>
                <span class="cost-value">{{ formatKamas(totalCraftCost) }}</span>
              </div>
              <div class="summary-row">
                <span>Precio Mercadillo:</span>
                <div class="input-wrapper">
                  <input 
                    type="text" 
                    :value="formatNumber(currentMarketPrice)" 
                    @input="e => updatePrice(e, 'market')"
                    class="price-input large" 
                    placeholder="0"
                  />
                </div>
              </div>
              <div class="summary-row profit-row" :class="profitClass">
                <span>Ganancia / Pérdida:</span>
                <span class="profit-value">{{ formatKamas(profit) }}</span>
              </div>
            </div>
          </div>
          
          <div v-else class="no-recipe">
            Este objeto no tiene receta (Drop o premio).
          </div>
        </div>
      </div>
      <!-- Global Tooltip for Ingredients -->
      <div 
        v-if="hoveredIng && hoveredIng.drops && hoveredIng.drops.length > 0" 
        class="ing-tooltip active"
        :style="{ top: tooltipY + 'px', left: tooltipX + 'px' }"
      >
        <div class="tooltip-title">Dropeado por:</div>
        <div v-for="drop in hoveredIng.drops" :key="drop.monstruo" class="tooltip-drop">
          <span class="drop-monster">{{ drop.monstruo }}</span>
          <span class="drop-info">{{ drop.porcentaje }} - {{ drop.pp }} PP</span>
        </div>
      </div>
    </div>
    
    <div v-else class="empty-state">
      <p>Busca un equipamiento arriba para ver su receta y costos</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'

// Data sources
import itemsDofusRetro from '../data/itemlist/itemsfinal.json'
import ingredientesData from '../data/itemlist/ingredientes.json'

// Load spell icons
const imageModulesSpellIcons = import.meta.glob('../data/spellicons/*.svg', { eager: true, import: 'default' })
const spellIconsMap = new Map()

for (const path in imageModulesSpellIcons) {
  const filename = path.split('/').pop()
  spellIconsMap.set(filename, imageModulesSpellIcons[path])
}

// Load item images (equipment)
const imageModulesItems = import.meta.glob('../data/itemlist/img/*.svg', { eager: true, import: 'default' })
const itemImagesMap = new Map()

for (const path in imageModulesItems) {
  const filename = path.split('/').pop()
  itemImagesMap.set(filename, imageModulesItems[path])
}

// Load weapon images (support SVG and PNG)
const imageModulesArmas = import.meta.glob(['../data/itemlist/armas/*.svg', '../data/itemlist/armas/*.png'], { eager: true, import: 'default' })
const armasImagesMap = new Map()

for (const path in imageModulesArmas) {
  const filename = path.split('/').pop()
  armasImagesMap.set(filename, imageModulesArmas[path])
}

// Load system icons (notfound, etc.)
const imageModulesIcons = import.meta.glob(['../data/icons/*.png', '../data/icons/*.svg'], { eager: true, import: 'default' })
const iconsMap = new Map()

for (const path in imageModulesIcons) {
  const filename = path.split('/').pop()
  iconsMap.set(filename, imageModulesIcons[path])
}

// Load resource images
const imageModulesResources = import.meta.glob('../data/itemlist/recursos/*.svg', { eager: true, import: 'default' })
const resourceImagesMap = new Map()

for (const path in imageModulesResources) {
  const filename = path.split('/').pop()
  resourceImagesMap.set(filename, imageModulesResources[path])
}

// Load consumable images
const imageModulesConsumables = import.meta.glob('../data/itemlist/consumibles/*.svg', { eager: true, import: 'default' })
const consumableImagesMap = new Map()

for (const path in imageModulesConsumables) {
  const filename = path.split('/').pop()
  consumableImagesMap.set(filename, imageModulesConsumables[path])
}

const items = ref([])
const recursosMap = ref(new Map())
const recetasMap = ref(new Map())

const searchQuery = ref('')
const showDropdown = ref(false)
const selectedItem = ref(null)
const itemQuantity = ref(1)
const itemQuantityDisplay = ref('1')

const ingredientPrices = ref({})
const equipmentMarketPrices = ref({})

// Tooltip State
const hoveredIng = ref(null)
const tooltipX = ref(0)
const tooltipY = ref(0)

const handleTooltipEnter = (event, ing) => {
  if (ing.drops && ing.drops.length > 0) {
    hoveredIng.value = ing
    updateTooltipPos(event)
  }
}

const handleTooltipMove = (event) => {
  if (hoveredIng.value) {
    updateTooltipPos(event)
  }
}

const handleTooltipLeave = () => {
  hoveredIng.value = null
}

const updateTooltipPos = (event) => {
  // Offset to avoid cursor overlapping
  tooltipX.value = event.clientX + 15
  tooltipY.value = event.clientY + 15
}

// Format kamas helper
const formatKamas = (value) => {
  return formatNumber(value || 0) + ' k'
}

const formatNumber = (val) => {
  if (val === undefined || val === null || val === '') return ''
  return new Intl.NumberFormat('es-ES').format(val)
}

// Initialization
onMounted(() => {
  // Initialize items
  items.value = itemsDofusRetro

  // Build resource map (by ID and by Name for parsing)
  const mapRec = new Map()
  const mapRecByName = new Map()
  ingredientesData.forEach(r => {
    mapRec.set(r.id, r)
    mapRecByName.set(r.nombre.toLowerCase(), r)
  })
  recursosMap.value = mapRec

  // Build items map by name for ingredient lookup
  const mapItemsByName = new Map()
  itemsDofusRetro.forEach(item => {
    mapItemsByName.set(item.nombre.toLowerCase(), item)
  })

  // Build recipes map from the items_craft_completo structure
  const mapRecetas = new Map()
  itemsDofusRetro.forEach(item => {
    if (item.ingredientes && Array.isArray(item.ingredientes)) {
      const parsedIngredients = item.ingredientes.map(ingStr => {
        // Parse "1x Item Name"
        const match = ingStr.match(/^(\d+)x\s+(.+)$/)
        if (match) {
          const cantidad = parseInt(match[1])
          const nombre = match[2]
          const resource = mapRecByName.get(nombre.toLowerCase())
          const itemIngredient = mapItemsByName.get(nombre.toLowerCase())
          
          return {
            nombre: nombre,
            cantidad: cantidad,
            recursoId: resource ? resource.id : (itemIngredient ? `item-${itemIngredient.nombre}` : `missing-${nombre}`),
            imagen: resource ? resource.imagen : (itemIngredient ? itemIngredient.imagen : null),
            drops: resource ? resource.drops : null
          }
        }
        return null
      }).filter(i => i !== null)
      
      mapRecetas.set(item.nombre, parsedIngredients)
    }
  })
  recetasMap.value = mapRecetas

  // Load cached prices
  const cachedIng = localStorage.getItem('dofus_ingredient_prices')
  if (cachedIng) {
    try {
      ingredientPrices.value = JSON.parse(cachedIng)
    } catch (e) { console.error('Error parsing ingredient prices') }
  }

  const cachedEq = localStorage.getItem('dofus_equipment_prices')
  if (cachedEq) {
    try {
      equipmentMarketPrices.value = JSON.parse(cachedEq)
    } catch (e) { console.error('Error parsing equipment prices') }
  }
})

// Search logic
const filteredItems = computed(() => {
  if (!searchQuery.value) return []
  const query = searchQuery.value.toLowerCase()
  return items.value
    .filter(item => !item.desactivado && item.nombre.toLowerCase().includes(query))
    .slice(0, 50) // Limit to 50 for performance
})

const handleBlur = () => {
  // Delay so mousedown on item registers first
  setTimeout(() => {
    showDropdown.value = false
  }, 200)
}

const selectItem = (item) => {
  // Attach recipe data from map before setting
  const matchingRecipes = recetasMap.value.get(item.nombre) || []
  selectedItem.value = { ...item, recetas: matchingRecipes }
  searchQuery.value = item.nombre
  showDropdown.value = false
}

const handleItemQuantityInput = (e) => {
  const rawValue = e.target.value.replace(/\D/g, '')
  const numericValue = rawValue ? parseInt(rawValue, 10) : 0
  
  if (numericValue < 1) {
    itemQuantity.value = 1
    itemQuantityDisplay.value = '1'
  } else {
    itemQuantity.value = numericValue
    itemQuantityDisplay.value = rawValue
  }
}

const updatePrice = (e, type, id = null) => {
  const input = e.target
  const rawValue = input.value.replace(/\D/g, '')
  const numericValue = rawValue ? parseInt(rawValue, 10) : 0
  
  if (type === 'ingredient') {
    ingredientPrices.value[id] = numericValue
    savePrices()
  } else if (type === 'market') {
    currentMarketPrice.value = numericValue
  }
  
  // Force update display
  input.value = formatNumber(numericValue)
}

// Stat icon mapping
const getStatIcon = (statStr) => {
  const s = statStr.toLowerCase()
  if (s.includes('vitalidad')) return spellIconsMap.get('Vita.svg')
  if (s.includes('sabiduría') || s.includes('sabiduria')) return spellIconsMap.get('Wisdom.svg')
  if (s.includes('fuerza')) return spellIconsMap.get('EarthBonus.svg')
  if (s.includes('inteligencia')) return spellIconsMap.get('FireBonus.svg')
  if (s.includes('agilidad')) return spellIconsMap.get('AirBonus.svg')
  if (s.includes('suerte')) return spellIconsMap.get('WaterBonus.svg')
  if (s.includes('golpes críticos') || s.includes('críticos')) return spellIconsMap.get('cc.svg')
  if (s.includes('% de daños') || s.includes('daños en un') || s.includes('potencia')) return spellIconsMap.get('pu.svg')
  if (s.includes('daños')) return spellIconsMap.get('dmg.svg')
  if (s.includes('alcance')) return spellIconsMap.get('po.svg')
  if (s.includes('pa')) return spellIconsMap.get('pa.svg')
  if (s.includes('pm')) return spellIconsMap.get('pm.svg')
  if (s.includes('iniciativa')) return spellIconsMap.get('ic.svg')
  if (s.includes('prospección') || s.includes('prospeccion')) return spellIconsMap.get('pp.svg')
  if (s.includes('curaciones')) return spellIconsMap.get('so.svg')
  if (s.includes('invocable')) return spellIconsMap.get('ic.svg')
  return null
}

const getItemImage = (imgRelPath) => {
  if (!imgRelPath) return iconsMap.get('notfound.png')
  const filename = imgRelPath.split('/').pop()
  
  if (imgRelPath.includes('/icons/')) {
    return iconsMap.get(filename) || iconsMap.get('notfound.png')
  }
  if (imgRelPath.includes('/recursos/')) {
    return resourceImagesMap.get(filename) || iconsMap.get('notfound.png')
  }
  if (imgRelPath.includes('/consumibles/')) {
    return consumableImagesMap.get(filename) || iconsMap.get('notfound.png')
  }
  if (imgRelPath.includes('/armas/')) {
    return armasImagesMap.get(filename) || iconsMap.get('notfound.png')
  }
  
  // Default to item images or notfound
  return itemImagesMap.get(filename) || iconsMap.get('notfound.png')
}

// Calculations
const calculateIngredientTotal = (ing) => {
  const price = ingredientPrices.value[ing.recursoId] || 0
  return price * ing.cantidad * itemQuantity.value
}

const totalCraftCost = computed(() => {
  if (!selectedItem.value || !selectedItem.value.recetas) return 0
  return selectedItem.value.recetas.reduce((total, ing) => {
    return total + calculateIngredientTotal(ing)
  }, 0)
})

const currentMarketPrice = computed({
  get() {
    if (!selectedItem.value) return 0
    const unitPrice = equipmentMarketPrices.value[selectedItem.value.imagen] || 0
    return unitPrice * itemQuantity.value
  },
  set(val) {
    if (!selectedItem.value) return
    // Store as unit price
    equipmentMarketPrices.value[selectedItem.value.imagen] = Math.floor(val / itemQuantity.value)
    saveMarketPrice()
  }
})

const profit = computed(() => {
  return currentMarketPrice.value - totalCraftCost.value
})

const profitClass = computed(() => {
  if (profit.value > 0) return 'profit-positive'
  if (profit.value < 0) return 'profit-negative'
  return ''
})

// Savers
const savePrices = () => {
  localStorage.setItem('dofus_ingredient_prices', JSON.stringify(ingredientPrices.value))
}

const saveMarketPrice = () => {
  localStorage.setItem('dofus_equipment_prices', JSON.stringify(equipmentMarketPrices.value))
}
</script>

<style scoped>
.calculator-container {
  padding: 1rem 2rem;
  max-width: 1400px;
  margin: 0 auto;
  color: #e2e8f0;
}

.calculator-header {
  margin-bottom: 1.5rem;
  background: rgba(15, 23, 42, 0.7);
  backdrop-filter: blur(16px);
  padding: 1.5rem 2.5rem;
  border-radius: 20px;
  border: 1px solid rgba(14, 165, 233, 0.2);
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.5);
  position: relative;
  z-index: 100;
}

.header-left {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 2rem;
}

.header-content {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.title-separator {
  color: rgba(14, 165, 233, 0.5);
  font-size: 2.5rem;
  font-weight: 300;
}

.title {
  color: #fff;
  font-size: 2rem;
  margin: 0;
  text-transform: uppercase;
  letter-spacing: 1px;
  text-shadow: 0 0 10px rgba(56, 189, 248, 0.5);
}


.search-container {
  position: relative;
  width: 400px;
}

.search-box {
  position: relative;
}

.search-input {
  width: 100%;
  padding: 1rem;
  background: rgba(15, 23, 42, 0.6);
  border: 1px solid rgba(56, 189, 248, 0.3);
  border-radius: 10px;
  color: #e2e8f0;
  font-size: 0.95rem;
  transition: all 0.3s ease;
}

.search-input:focus {
  outline: none;
  border-color: #38bdf8;
  box-shadow: 0 0 0 3px rgba(56, 189, 248, 0.1);
  background: rgba(15, 23, 42, 0.9);
}

.dropdown {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  margin-top: 0.5rem;
  background: #1e293b;
  border: 1px solid rgba(148, 163, 184, 0.2);
  border-radius: 12px;
  max-height: 400px;
  overflow-y: auto;
  z-index: 1000;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
}

.dropdown-item {
  display: flex;
  align-items: center;
  padding: 0.75rem 1rem;
  cursor: pointer;
  transition: background 0.2s;
  border-bottom: 1px solid rgba(148, 163, 184, 0.1);
}

.dropdown-item:hover {
  background: rgba(56, 189, 248, 0.1);
}

.dropdown-item-image-container {
  width: 32px;
  height: 32px;
  margin-right: 12px;
  background: rgba(15, 23, 42, 0.4);
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid rgba(148, 163, 184, 0.1);
}

.dropdown-item-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.item-name {
  flex: 1;
  font-weight: 500;
}

.item-level {
  font-size: 0.85rem;
  color: #94a3b8;
  background: rgba(15, 23, 42, 0.6);
  padding: 0.2rem 0.6rem;
  border-radius: 12px;
}

.panels-wrapper {
  display: grid;
  grid-template-columns: 320px 1fr;
  gap: 1.5rem;
}

.panel h3 {
  color: #38bdf8;
  border-bottom: 1px solid rgba(56, 189, 248, 0.3);
  padding-bottom: 0.8rem;
  margin-bottom: 1.5rem;
  font-size: 1.3rem;
  text-shadow: 0 0 8px rgba(56, 189, 248, 0.3);
  text-transform: uppercase;
  letter-spacing: 1px;
}

.panel {
  background: rgba(30, 41, 59, 0.5);
  border: 1px solid rgba(148, 163, 184, 0.1);
  border-radius: 16px;
  overflow: hidden;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.2);
}

.item-details-panel {
  padding: 1.5rem;
}

.item-header {
  text-align: center;
  margin-bottom: 1rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid rgba(148, 163, 184, 0.1);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.item-image-container {
  width: 100px;
  height: 100px;
  background: rgba(15, 23, 42, 0.6);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid rgba(148, 163, 184, 0.1);
  padding: 10px;
}

.item-visuals {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.item-quantity-wrapper {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  background: rgba(15, 23, 42, 0.6);
  padding: 0.5rem 0.75rem;
  border-radius: 10px;
  border: 1px solid rgba(56, 189, 248, 0.3);
}

.qty-label {
  color: #38bdf8;
  font-weight: 700;
  font-size: 1.2rem;
}

.item-quantity-input {
  width: 50px;
  background: transparent;
  border: none;
  color: #fff;
  font-size: 1.2rem;
  font-weight: 700;
  text-align: center;
  outline: none;
}

.item-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
  filter: drop-shadow(0 4px 6px rgba(0, 0, 0, 0.3));
}

.item-title {
  font-size: 1.3rem;
  color: #f8fafc;
  margin: 0 0 0.25rem 0;
}

.item-level-badge {
  display: inline-block;
  background: rgba(56, 189, 248, 0.15);
  color: #38bdf8;
  padding: 0.3rem 1rem;
  border-radius: 20px;
  font-size: 0.9rem;
  font-weight: 600;
  border: 1px solid rgba(56, 189, 248, 0.3);
}

.stats-container {
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.stat-row {
  display: flex;
  align-items: center;
  gap: 0.8rem;
  padding: 0.5rem;
  background: rgba(15, 23, 42, 0.4);
  border-radius: 8px;
}

.stat-icon {
  width: 20px;
  height: 20px;
  object-fit: contain;
}

.stat-value {
  color: #10b981;
  font-weight: 600;
  min-width: 65px;
}

.stat-name {
  color: #cbd5e1;
}

.recipe-header {
  padding: 1rem 1.5rem;
  background: rgba(15, 23, 42, 0.6);
  border-bottom: 1px solid rgba(148, 163, 184, 0.1);
}

.recipe-header h3 {
  margin: 0;
  display: flex;
  align-items: center;
  gap: 0.8rem;
  font-size: 1.1rem;
  color: #f8fafc;
}

.ingredients-list {
  padding: 1rem 1.5rem;
}

.ingredient-header {
  display: flex;
  padding: 0.25rem 1rem;
  color: #94a3b8;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: 600;
  border-bottom: 1px solid rgba(148, 163, 184, 0.1);
  margin-bottom: 0.5rem;
}

.ingredient-row {
  display: flex;
  align-items: center;
  padding: 0.5rem 1rem;
  background: rgba(15, 23, 42, 0.3);
  border: 1px solid rgba(148, 163, 184, 0.05);
  border-radius: 8px;
  margin-bottom: 0.4rem;
  transition: background 0.2s;
}

.ingredient-row:hover {
  background: rgba(15, 23, 42, 0.6);
}

.col-ing {
  flex: 2;
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.ing-name {
  font-weight: 500;
}

.ing-image-container {
  width: 28px;
  height: 28px;
  background: rgba(15, 23, 42, 0.4);
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid rgba(148, 163, 184, 0.1);
  flex-shrink: 0;
}

.ing-image {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.col-qty {
  flex: 0.5;
  color: #94a3b8;
  font-weight: 600;
  text-align: center;
}

.col-price {
  flex: 1;
  display: flex;
  justify-content: center;
}

.col-total {
  flex: 1;
  text-align: right;
  font-variant-numeric: tabular-nums;
  font-weight: 600;
}

.col-total.highlight {
  color: #f8fafc;
}

.price-input {
  background: rgba(15, 23, 42, 0.8);
  border: 1px solid rgba(148, 163, 184, 0.2);
  color: #e2e8f0;
  padding: 0.35rem 0.5rem;
  border-radius: 6px;
  width: 100px;
  text-align: right;
  font-family: inherit;
  transition: all 0.2s;
}

.price-input:focus {
  outline: none;
  border-color: #38bdf8;
  background: #1e293b;
}

.price-input.large {
  width: 130px;
  font-size: 1.05rem;
  font-weight: 600;
  color: #fbbf24;
}

.recipe-summary {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px dashed rgba(148, 163, 184, 0.1);
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  align-items: flex-end;
}

.summary-row {
  display: flex;
  align-items: center;
  gap: 1rem;
  font-size: 1rem;
}

.cost-value {
  font-weight: 700;
  color: #f87171;
  font-size: 1.1rem;
  min-width: 120px;
  text-align: right;
}

.profit-row {
  margin-top: 0.5rem;
  padding: 1rem;
  background: rgba(15, 23, 42, 0.8);
  border-radius: 10px;
  border: 1px solid rgba(148, 163, 184, 0.1);
  width: 100%;
  justify-content: space-between;
}

.profit-row span:first-child {
  font-weight: 600;
  color: #94a3b8;
  font-size: 1.1rem;
}

.profit-value {
  font-size: 1.5rem;
  font-weight: 700;
  text-shadow: 0 2px 4px rgba(0,0,0,0.5);
}

.profit-positive .profit-value {
  color: #10b981;
}

.profit-negative .profit-value {
  color: #ef4444;
}

.no-recipe {
  padding: 4rem;
  text-align: center;
  color: #94a3b8;
  font-size: 1.1rem;
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 6rem;
  background: rgba(30, 41, 59, 0.3);
  border: 1px dashed rgba(148, 163, 184, 0.2);
  border-radius: 16px;
  color: #64748b;
  text-align: center;
}

/* Scrollbar styling for dropdown */
.dropdown::-webkit-scrollbar {
  width: 8px;
}
.dropdown::-webkit-scrollbar-track {
  background: rgba(15, 23, 42, 0.5);
}
.dropdown::-webkit-scrollbar-thumb {
  background: rgba(148, 163, 184, 0.3);
  border-radius: 4px;
}
.dropdown::-webkit-scrollbar-thumb:hover {
  background: rgba(148, 163, 184, 0.5);
}
/* Tooltip styles */
.ing-name {
  position: relative;
  transition: color 0.2s;
}

.ing-name.has-tooltip {
  cursor: help;
  border-bottom: 1px dotted rgba(148, 163, 184, 0.4);
}

.ing-name.has-tooltip:hover {
  color: #38bdf8;
}

.ing-tooltip {
  display: none;
  width: 250px;
  background: #1e293b;
  color: #f8fafc;
  text-align: left;
  border-radius: 8px;
  padding: 12px;
  position: fixed;
  z-index: 9999;
  opacity: 0;
  box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.6), 0 8px 10px -6px rgba(0, 0, 0, 0.6);
  border: 1px solid rgba(56, 189, 248, 0.3);
  pointer-events: none;
  font-weight: normal;
  line-height: 1.4;
  transform: translate(0, 0); /* Reset any previous transform */
}

.ing-tooltip.active {
  display: block;
  opacity: 1;
}

.tooltip-title {
  color: #38bdf8;
  font-size: 0.85rem;
  font-weight: 700;
  margin-bottom: 8px;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  border-bottom: 1px solid rgba(56, 189, 248, 0.2);
  padding-bottom: 4px;
}

.tooltip-drop {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 4px;
  font-size: 0.85rem;
  gap: 10px;
}

.drop-monster {
  color: #e2e8f0;
  font-weight: 500;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.drop-info {
  color: #fbbf24;
  font-weight: 600;
  font-variant-numeric: tabular-nums;
  white-space: nowrap;
}
</style>
