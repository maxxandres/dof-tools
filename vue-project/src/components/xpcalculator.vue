<template>
  <div class="xp-calculator-container">

    <!-- Description banner -->
    <!-- <div class="description-banner glass-panel">
      <p class="description-text">Ejemplo</p>
    </div> -->

    <!-- Fila 1: Ingresar Datos -->
    <div class="panel glass-panel inputs-row">
      <!-- Header: título + selector de PL -->
      <div class="inputs-header">
        <h2 class="panel-subtitle-bar">Datos</h2>
        <div class="pl-selection-wrapper">
        <div class="pl-images-container2" v-if="plImages.length">
          <img v-for="(img, idx) in plImages" :key="idx" :src="img.src" :style="img.style" class="pl-image" />
        </div>
        </div>
        <!-- Contenedor del Dropdown + Imágenes -->
        <div class="pl-selection-wrapper">
          <!-- Imágenes de Personaje y Jefe -->


          <!-- Dropdown-style PL selector -->
          <div class="pl-dropdown" @click="showPLModal = true">
          <span class="pl-dropdown-icon">PL</span>
          <span
            class="pl-dropdown-value"
            :style="selectedPL && selectedPL.color ? { color: selectedPL.color } : {}"
          >{{ selectedPL ? selectedPL.nombre : 'Seleccionar PL' }}</span>
          <svg class="pl-dropdown-chevron" :class="{ open: showPLModal }" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5">
            <polyline points="6 9 12 15 18 9"/>
          </svg>
        </div>
      </div>
      </div>
      <div class="inputs-horizontal">
        <div class="input-field">
          <label>Nivel actual</label>
          <input type="number" v-model.number="nivelActual" @blur="limitCurrentLevel" min="1" max="200" :class="{ 'error': currentLevelError }" @keydown="preventNonInteger">
        </div>
        <div class="input-field">
          <label>XP actual <span class="highlight-text">%</span></label>
          <input type="number" v-model.number="xpActualPorcentaje" @blur="limitXp" min="0" max="100" @keydown="preventNonInteger">
        </div>
        <div class="input-field">
          <label>Nivel objetivo</label>
          <input type="number" v-model.number="nivelObjetivo" @blur="limitTargetLevel" min="2" max="200" :class="{ 'error': targetLevelError }" @keydown="preventNonInteger">
        </div>
        <!-- XP por pelea -->
        <div class="input-field">
          <label>XP por pelea
            <span class="tooltip-wrap">
              <span class="tooltip-icon">?</span>
              <span class="tooltip-box">XP que obtienes por cada pelea de PL.<br>Ej: 13.000.000</span>
            </span>
          </label>
          <input
            type="text"
            :value="xpPorPeleaStr"
            @input="formatXpPorPelea"
            :style="selectedPL && selectedPL.color ? { color: selectedPL.color, borderColor: selectedPL.color + '80' } : {}"
            @keydown="preventNonDigit"
          >
        </div>
        <!-- Precio pelea -->
        <div class="input-field">
          <label>Precio pelea
            <span class="tooltip-wrap">
              <span class="tooltip-icon">?</span>
              <span class="tooltip-box">Costo en kamas que pagas por cada pelea.<br>Si pagas una ronda, divide el precio que pagaste por la cantidad de peleas.</span>
            </span>
          </label>
          <input
            type="text"
            :value="precioPeleaStr"
            @input="formatPrecioPelea"
            :style="selectedPL && selectedPL.color ? { color: selectedPL.color, borderColor: selectedPL.color + '80' } : {}"
            @keydown="preventNonDigit"
          >
        </div>
        <div class="input-field">
          <label>Min. por pelea
            <span class="tooltip-wrap">
              <span class="tooltip-icon">?</span>
              <span class="tooltip-box">Duración aproximada de cada pelea en minutos.<br>Ej: 7</span>
            </span>
          </label>
          <input
            type="number"
            v-model.number="tiempoPelea"
            min="1"
            :style="selectedPL && selectedPL.color ? { color: selectedPL.color, borderColor: selectedPL.color + '80' } : {}"
            @keydown="preventNonInteger"
          >
        </div>

      </div>
    </div>

    <!-- Fila 2: Información PL + Barra de Progreso -->
    <div class="bottom-grid">

      <!-- Información de la PL -->
       
      <div class="panel glass-panel autocompletado">
       
        <div class="result-row highlight">
          <span>XP Restante</span>
          <span class="value">{{ formatNumber(xpRestante) }}</span>
        </div>
        
        <div class="result-row">
          <span>Niveles que subirás</span>
          <span class="value">{{ nivelesSubiras > 0 ? nivelesSubiras : 0 }}</span>
        </div>
        
        <div class="result-row highlight-action">
          <span>Peleas necesarias</span>
          <span class="value">{{ formatNumber(peleasNecesarias) }}</span>
        </div>
        
        <div class="result-row kamas-row featured-row">
          <span>Costo peleas (Kamas)</span>
          <span class="value featured-value">{{ formatNumber(costoPeleas) }}</span>
        </div>
        
        <div class="result-row tiempo-row featured-row">
          <span>Tiempo en horas</span>
          <span class="value featured-value">{{ tiempoHoras }} h</span>
        </div>
      </div>

      <!-- Barra de Progreso (columna derecha) -->
      <div class="progress-section glass-panel progress-glow">
        <div class="progress-header">
          <h2 class="panel-subtitle">Progreso hacia Nivel {{ nivelObjetivo }} </h2>
          <span class="progress-value">{{ Math.min(100, Math.max(0, porcentajeProgreso)).toFixed(4) }}%</span>
        </div>
        <div class="progress-bar-container">
          <div class="progress-bar-fill" :style="{ width: Math.min(100, Math.max(0, porcentajeProgreso)) + '%' }">
            <span class="progress-label">{{ Math.min(100, Math.max(0, porcentajeProgreso)).toFixed(2) }}%</span>
          </div>
        </div>
      </div>

    </div>

    <!-- ===================== MODAL PL ===================== -->
    <teleport to="body">
      <transition name="modal-fade">
        <div v-if="showPLModal" class="modal-overlay" @click.self="closePLModal">
          <div class="modal-box glass-panel">

            <!-- Header modal -->
            <div class="modal-header">
              <h2 class="modal-title">⚔ Seleccionar tipo de PL</h2>
              <button class="modal-close" @click="closePLModal">×</button>
            </div>

            <!-- Formulario editar / agregar -->
            <div v-if="editModal.open" class="modal-edit-form">
              <h3 class="modal-edit-title">{{ editModal.isNew ? 'Agregar nueva PL' : 'Editar PL' }}</h3>
              <div class="modal-edit-fields">
                <div class="modal-field">
                  <label>Nombre</label>
                  <input type="text" v-model="editModal.data.nombre" placeholder="Ej: Bworker" :disabled="!editModal.isNew && !editModal.item.custom">
                </div>
                <div class="modal-field">
                  <label>Precio (Kamas)</label>
                  <input type="text" :value="formatNumber(editModal.data.precio)" @input="e => editModal.data.precio = parseNumber(e.target.value)" placeholder="12.000" @keydown="preventNonDigit">
                </div>
                <div class="modal-field">
                  <label>XP Estimada</label>
                  <input type="text" :value="formatNumber(editModal.data.xp)" @input="e => editModal.data.xp = parseNumber(e.target.value)" placeholder="13.000.000" @keydown="preventNonDigit">
                </div>
                <div class="modal-field">
                  <label>Min. por pelea</label>
                  <input type="number" v-model.number="editModal.data.minutos" min="0" placeholder="7" @keydown="preventNonInteger">
                </div>
              </div>
              <div class="modal-edit-actions">
                <button class="btn-modal-cancel" @click="editModal.open = false">Cancelar</button>
                <button class="btn-modal-save" @click="saveEditModal">Guardar</button>
              </div>
            </div>

            <!-- Lista PLs -->
            <div v-else>
              <div class="modal-pl-list">
                <div
                  v-for="(item, index) in datosExtras"
                  :key="index"
                  class="modal-pl-row"
                  :class="{ active: selectedPL && selectedPL.nombre === item.nombre }"
                  :style="item.color ? { borderLeftColor: item.color + '55' } : {}"
                  @click="applyExtraData(item)"
                >
                  <div class="modal-pl-info">
                    <span class="modal-pl-name" :style="{ color: item.color || '#38bdf8' }">{{ item.nombre }}</span>
                    <span class="modal-pl-stats" :style="{ color: item.color ? item.color + 'bb' : '#94a3b8' }">
                      {{ formatNumber(item.xp) }} XP &nbsp;·&nbsp; {{ formatNumber(item.precio) }} kamas
                    </span>
                  </div>
                  <div class="modal-pl-actions" @click.stop>
                    <button class="btn-icon-edit" @click="openEditModal(item, index)" title="Editar">✎</button>
                    <button v-if="item.custom" class="btn-icon-delete" @click="deleteExtraData(index)" title="Eliminar">×</button>
                  </div>
                </div>
              </div>

              <!-- Agregar nueva -->
              <button class="btn-add-new-pl" @click="openAddModal">
                <span>+</span> Agregar nueva PL
              </button>
            </div>

          </div>
        </div>
      </transition>
    </teleport>

  </div>
</template>
      <!-- Ingresar Datos -->

<script>
import tablaXpData from '../data/tabla_xp_acumulada.json';
import datosExtrasData from '../data/datos_extras.json';

// Image imports
import bworkerImg from '../data/images/Bworker.webp';
import minototImg from '../data/images/Minotot.webp';
import kimboImg from '../data/images/Kimbo.webp';
import robleImg from '../data/images/roble.png';
import sfinterImg from '../data/images/sphincter-cell.png';
import jalatoImg from '../data/images/bouftouroyal.png';
import craImg from '../data/images/Cra.svg';
import fecaImg from '../data/images/Feca.svg';
import pandaImg from '../data/images/Pandawa.svg';
import sacriImg from '../data/images/Sacrieur.svg';
import sadiImg from '../data/images/Sadida.svg';

export default {
  name: 'XpCalculator',
  data() {
    return {
      nivelActual: 1,
      xpActualPorcentaje: 0,
      nivelObjetivo: 65,
      xpPorPeleaRaw: 0,
      precioPeleaRaw: 0,
      tiempoPelea: 0,
      datosExtras: [],
      newExtra: { nombre: '', precioStr: '', xpStr: '' },
      selectedPL: null,
      showPLModal: false,
      editModal: {
        open: false,
        isNew: false,
        index: null,
        item: null,
        data: { nombre: '', precio: 0, xp: 0 }
      }
    }
  },
  computed: {
    plImages() {
      if (!this.selectedPL) return [];
      const name = this.selectedPL.nombre.toLowerCase();
      // Adjust the `height` values below to change the sizes of the images:
      if (name.includes('bworker')) return [
        { src: sacriImg, style: { height: '35px' } },
        { src: bworkerImg, style: { height: '45px', marginLeft: '5px' } }
      ];
      if (name.includes('minotot')) return [
        { src: craImg, style: { height: '28px' } }, 
        { src: pandaImg, style: { height: '30px' } }, 
        { src: minototImg, style: { height: '48px', marginLeft: '5px' } }
      ];
      if (name.includes('sfinter')) return [
        { src: fecaImg, style: { height: '35px' } }, 
        { src: sfinterImg, style: { height: '48px', marginLeft: '5px' } }
      ];
      if (name.includes('roble')) return [
        { src: sacriImg, style: { height: '35px' } }, 
        { src: robleImg, style: { height: '48px', marginLeft: '5px' } }
      ];
      if (name.includes('kimbo')) return [
        { src: pandaImg, style: { height: '35px' } }, 
        { src: kimboImg, style: { height: '48px', marginLeft: '5px' } }
      ];
      if (name.includes('jalató')) return [
        { src: sadiImg, style: { height: '45px' } }, 
        { src: jalatoImg, style: { height: '48px', marginLeft: '5px' } }
      ];
      return [];
    },
    tablaXp() {
      const map = {};
      tablaXpData.forEach(item => {
        map[item.Nivel] = item.XP;
      });
      return map;
    },
    xpPorPeleaStr: {
      get() { return this.formatNumber(this.xpPorPeleaRaw); },
      set(val) { this.xpPorPeleaRaw = this.parseNumber(val); }
    },
    precioPeleaStr: {
      get() { return this.formatNumber(this.precioPeleaRaw); },
      set(val) { this.precioPeleaRaw = this.parseNumber(val); }
    },
    currentLevelError() {
      return this.nivelActual < 1 || this.nivelActual > 200;
    },
    targetLevelError() {
      return this.nivelObjetivo <= this.nivelActual || this.nivelObjetivo > 200;
    },
    xpTotalObjetivo() {
      if(this.targetLevelError && this.nivelObjetivo > 200) return 0;
      return this.tablaXp[this.nivelObjetivo] || 0;
    },
    xpActualTotal() {
      let lvl = this.nivelActual;
      if(lvl < 1) lvl = 1;
      if(lvl >= 200) return this.tablaXp[200];
      
      const xpBase = this.tablaXp[lvl] || 0;
      const xpNext = this.tablaXp[lvl + 1] || 0;
      
      const xpRequiredForNext = xpNext - xpBase;
      const xpFromPercent = xpRequiredForNext * (this.xpActualPorcentaje / 100);
      
      return xpBase + xpFromPercent;
    },
    xpRestante() {
      if (this.currentLevelError || this.targetLevelError) return 0;
      return Math.max(0, this.xpTotalObjetivo - this.xpActualTotal);
    },
    nivelesSubiras() {
      return this.nivelObjetivo - Math.max(1, this.nivelActual);
    },
    peleasNecesarias() {
      if (this.xpPorPeleaRaw <= 0) return 0;
      return Math.ceil(this.xpRestante / this.xpPorPeleaRaw);
    },
    costoPeleas() {
      return this.peleasNecesarias * this.precioPeleaRaw;
    },
    tiempoHoras() {
      const min = this.peleasNecesarias * this.tiempoPelea;
      return Math.round(min / 60);
    },
    porcentajeProgreso() {
      if (this.xpTotalObjetivo <= 0) return 0;
      return (this.xpActualTotal / this.xpTotalObjetivo) * 100;
    }
  },
  watch: {
    $data: {
      handler() {
        this.saveToStorage();
      },
      deep: true
    }
  },
  mounted() {
    this.loadFromStorage();
  },
  methods: {
    preventNonInteger(e) {
      if (['.', ',', 'e', 'E', '+', '-'].includes(e.key)) {
        e.preventDefault();
      }
    },
    preventNonDigit(e) {
      if (
        ['Backspace', 'Delete', 'Tab', 'Escape', 'Enter', 'ArrowLeft', 'ArrowRight', 'ArrowUp', 'ArrowDown', 'Home', 'End'].includes(e.key) ||
        e.ctrlKey || e.metaKey || e.altKey
      ) {
        return;
      }
      if (!/^\d$/.test(e.key)) {
        e.preventDefault();
      }
    },
    saveToStorage() {
      const stateToSave = {
        nivelActual: this.nivelActual,
        xpActualPorcentaje: this.xpActualPorcentaje,
        nivelObjetivo: this.nivelObjetivo,
        xpPorPeleaRaw: this.xpPorPeleaRaw,
        precioPeleaRaw: this.precioPeleaRaw,
        tiempoPelea: this.tiempoPelea,
        datosExtras: this.datosExtras
      };
      localStorage.setItem('dofusCalcState', JSON.stringify(stateToSave));
    },
    loadFromStorage() {
      const savedState = localStorage.getItem('dofusCalcState');
      // Build a color lookup from the source JSON
      const colorMap = {};
      datosExtrasData.forEach(d => { if(d.color) colorMap[d.nombre] = d.color; });

      if (savedState) {
        try {
          const parsed = JSON.parse(savedState);
          this.nivelActual = parsed.nivelActual ?? 1;
          this.xpActualPorcentaje = parsed.xpActualPorcentaje ?? 0;
          this.nivelObjetivo = parsed.nivelObjetivo ?? 65;
          this.xpPorPeleaRaw = parsed.xpPorPeleaRaw ?? 13000000;
          this.precioPeleaRaw = parsed.precioPeleaRaw ?? 12000;
          this.tiempoPelea = parsed.tiempoPelea ?? 7;

          if (parsed.datosExtras && parsed.datosExtras.length > 0) {
            // Merge color from JSON for non-custom items that are missing it
            this.datosExtras = parsed.datosExtras.map(item => {
              if (!item.color && colorMap[item.nombre]) {
                return { ...item, color: colorMap[item.nombre] };
              }
              return item;
            });
          } else {
            this.datosExtras = JSON.parse(JSON.stringify(datosExtrasData));
          }
        } catch (e) {
          console.error('Error al leer el localStorage', e);
          this.datosExtras = JSON.parse(JSON.stringify(datosExtrasData));
        }
      } else {
        this.datosExtras = JSON.parse(JSON.stringify(datosExtrasData));
      }
    },
    restoreDefaults() {
      if (confirm('¿Restaurar los datos extras por defecto? Perderás tus modificaciones.')) {
        this.datosExtras = JSON.parse(JSON.stringify(datosExtrasData));
      }
    },
    addExtraData() {
      if (!this.newExtra.nombre) return;
      this.datosExtras.push({
        nombre: this.newExtra.nombre,
        precio: this.parseNumber(this.newExtra.precioStr),
        xp: this.parseNumber(this.newExtra.xpStr),
        custom: true
      });
      this.newExtra = { nombre: '', precioStr: '', xpStr: '' };
    },
    deleteExtraData(index) {
      this.datosExtras.splice(index, 1);
    },
    formatInputStr(e) {
      const raw = e.target.value.replace(/\D/g, '');
      return this.formatNumber(parseInt(raw) || 0);
    },
    updateTableFormat(e, item, field) {
      const val = e.target.value.replace(/\D/g, '');
      item[field] = parseInt(val) || 0;
      e.target.value = this.formatNumber(item[field]);
    },
    limitCurrentLevel() {
      if (this.nivelActual < 1) this.nivelActual = 1;
      if (this.nivelActual > 200) this.nivelActual = 200;
    },
    limitTargetLevel() {
      if (this.nivelObjetivo <= this.nivelActual) this.nivelObjetivo = this.nivelActual + 1;
      if (this.nivelObjetivo > 200) this.nivelObjetivo = 200;
    },
    limitXp() {
      if (this.xpActualPorcentaje < 0) this.xpActualPorcentaje = 0;
      if (this.xpActualPorcentaje > 100) this.xpActualPorcentaje = 100;
      this.xpActualPorcentaje = Math.round(this.xpActualPorcentaje);
    },
    formatNumber(num) {
      if (!num && num !== 0) return '';
      return num.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
    },
    parseNumber(str) {
      if (!str) return 0;
      return parseInt(str.toString().replace(/\./g, ''), 10) || 0;
    },
    formatXpPorPelea(e) {
      let val = e.target.value.replace(/\D/g, '');
      this.xpPorPeleaRaw = parseInt(val) || 0;
    },
    formatPrecioPelea(e) {
      let val = e.target.value.replace(/\D/g, '');
      this.precioPeleaRaw = parseInt(val) || 0;
    },
    applyExtraData(item) {
      if (item.precio >= 0) this.precioPeleaRaw = item.precio;
      if (item.xp >= 0) this.xpPorPeleaRaw = item.xp;
      if (item.minutos > 0) this.tiempoPelea = item.minutos;
      this.selectedPL = item;
      this.showPLModal = false;
    },
    closePLModal() {
      this.showPLModal = false;
      this.editModal.open = false;
    },
    openEditModal(item, index) {
      this.editModal = {
        open: true,
        isNew: false,
        index,
        item,
        data: { nombre: item.nombre, precio: item.precio, xp: item.xp, minutos: item.minutos ?? 0 }
      };
    },
    openAddModal() {
      this.editModal = {
        open: true,
        isNew: true,
        index: null,
        item: null,
        data: { nombre: '', precio: 0, xp: 0, minutos: 0 }
      };
    },
    saveEditModal() {
      const d = this.editModal.data;
      if (!d.nombre) return;
      if (this.editModal.isNew) {
        this.datosExtras.push({ nombre: d.nombre, precio: d.precio, xp: d.xp, minutos: d.minutos ?? 0, custom: true });
      } else {
        const item = this.datosExtras[this.editModal.index];
        item.precio = d.precio;
        item.xp = d.xp;
        item.minutos = d.minutos ?? 0;
        if (item.custom) item.nombre = d.nombre;
        // update selectedPL reference if it was the active one
        if (this.selectedPL && this.selectedPL.nombre === (this.editModal.item?.nombre || d.nombre)) {
          this.selectedPL = { ...item };
        }
      }
      this.editModal.open = false;
    }
  }
}
</script>


<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;600;700&display=swap');

.xp-calculator-container {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  font-family: 'Outfit', sans-serif;
  color: #e2e8f0;
  box-sizing: border-box;
}


.glass-panel {
  background: rgba(30, 41, 59, 0.4);
  backdrop-filter: blur(16px);
  -webkit-backdrop-filter: blur(16px);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 20px;
  padding: 1.8rem;
  box-shadow: 0 4px 30px rgba(0, 0, 0, 0.3), inset 0 1px 0 rgba(255,255,255,0.1);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.panel-subtitle-wrapper {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid rgba(14, 165, 233, 0.3);
  margin-bottom: 1.5rem;
  padding-bottom: 0.8rem;
}

.panel-subtitle {
  font-size: 1.3rem;
  font-weight: 600;
  color: #f8fafc;
  margin: 0;
  display: flex;
  align-items: center;
  gap: 10px;
}

.btn-restore {
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  color: #e2e8f0;
  border-radius: 8px;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: all 0.2s;
  font-size: 1.1rem;
}

.btn-restore:hover {
  background: #0ea5e9;
  border-color: #0ea5e9;
  color: white;
  transform: rotate(-180deg);
}

.hint {
  font-size: 0.8rem;
  color: #94a3b8;
  font-weight: 400;
  text-transform: none;
}

/* Layout */
.inputs-row {
  margin-bottom: 1.5rem;
}



.panel-subtitle-bar {
  font-size: 1.1rem;
  font-weight: 600;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 1px;
  margin: 0;
}

.inputs-horizontal {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 1rem;
}

.input-field {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.input-field label {
  font-size: 0.85rem;
  font-weight: 500;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.input-field input {
  width: 100%;
  box-sizing: border-box;
  background: rgba(2, 6, 23, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #38bdf8;
  padding: 0.7rem;
  border-radius: 10px;
  font-size: 1rem;
  font-weight: 600;
  text-align: center;
  transition: all 0.2s ease;
  outline: none;
  font-family: 'Outfit', sans-serif;
}

.input-field input:focus {
  border-color: #0ea5e9;
  box-shadow: 0 0 0 3px rgba(14, 165, 233, 0.2);
}

.input-field input.error {
  border-color: #f43f5e;
  color: #f43f5e;
}

.bottom-grid {
  display: grid;
  grid-template-columns: 1fr 1.5fr;
  gap: 1.5rem;
  margin-bottom: 1.5rem;
  align-items: stretch;
}

/* Progress inside grid: remove outer margin */
.bottom-grid .progress-section {
  margin-top: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

/* Description banner */
.description-banner {
  margin-bottom: 1.2rem;
  padding: 1rem 1.5rem;
  border-left: 3px solid rgba(14, 165, 233, 0.5);
}

.description-text {
  margin: 0;
  color: #64748b;
  font-size: 0.9rem;
  line-height: 1.6;
  font-style: italic;
}

.highlight-text {
  color: #0ea5e9;
  font-weight: 600;
}

/* RESULTS STYLES */
.autocompletado .result-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.2rem;
  padding: 0.8rem 1.2rem;
  border-radius: 12px;
  background: rgba(15, 23, 42, 0.5);
  font-size: 1rem;
  border-left: 4px solid transparent;
}

.autocompletado .result-row:hover {
  background: rgba(30, 41, 59, 0.8);
}

.result-row .value {
  font-weight: 700;
  font-size: 1.15rem;
  color: #f8fafc;
  letter-spacing: 0.5px;
}

.result-row.highlight {
  border-left-color: #fbbf24;
  background: linear-gradient(90deg, rgba(251, 191, 36, 0.05), transparent);
}
.result-row.highlight .value {
  color: #fcd34d;
}

.result-row.highlight-action {
  border-left-color: #f43f5e;
  background: linear-gradient(90deg, rgba(244, 63, 94, 0.05), transparent);
}
.result-row.highlight-action .value {
  color: #fda4af;
}

.kamas-row .value {
  color: #34d399;
}

/* Featured rows (Costo y Tiempo) */
.featured-row {
  padding: 1.1rem 1.2rem !important;
  border-radius: 14px !important;
  background: rgba(15, 23, 42, 0.7) !important;
}

.kamas-row.featured-row {
  border-left-color: #34d399 !important;
  background: linear-gradient(90deg, rgba(52, 211, 153, 0.08), transparent) !important;
}

.tiempo-row {
  border-left-color: #a78bfa !important;
  background: linear-gradient(90deg, rgba(167, 139, 250, 0.08), transparent) !important;
}

.tiempo-row .value {
  color: #a78bfa;
}

.featured-value {
  font-size: 1.5rem !important;
  font-weight: 800 !important;
  letter-spacing: 0.5px;
  text-shadow: 0 0 12px currentColor;
}

/* TABLE CONTROLS */
.add-row-form {
  display: flex;
  gap: 0.5rem;
  margin-bottom: 1rem;
  background: rgba(15, 23, 42, 0.5);
  padding: 0.7rem;
  border-radius: 12px;
}

.add-input {
  flex: 1;
  background: rgba(2, 6, 23, 0.6);
  border: 1px solid rgba(255, 255, 255, 0.1);
  color: #e2e8f0;
  padding: 0.5rem 0.6rem;
  border-radius: 8px;
  font-size: 0.9rem;
  font-family: inherit;
  outline: none;
  min-width: 0;
}

.add-input:focus { border-color: #0ea5e9; }

.btn-add {
  background: #0ea5e9;
  color: white;
  border: none;
  border-radius: 8px;
  width: 36px;
  min-width: 36px;
  font-size: 1.4rem;
  cursor: pointer;
  transition: background 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}

.btn-add:hover { background: #0284c7; }

.btn-delete {
  background: rgba(244, 63, 94, 0.15);
  color: #f43f5e;
  border: 1px solid rgba(244, 63, 94, 0.25);
  border-radius: 6px;
  width: 26px;
  height: 26px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1rem;
  line-height: 1;
  cursor: pointer;
  transition: all 0.2s;
  padding: 0;
}

.btn-delete:hover { background: #f43f5e; color: white; }

.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: separate;
  border-spacing: 0 5px;
}

th, td {
  padding: 0.5rem 0.6rem;
  text-align: center;
}

th {
  font-weight: 600;
  color: #94a3b8;
  text-transform: uppercase;
  font-size: 0.85rem;
  letter-spacing: 1px;
}

td {
  font-weight: 400;
  color: #e2e8f0;
  background: rgba(15, 23, 42, 0.4);
}

tr.clickable-row {
  cursor: pointer;
  transition: transform 0.2s ease;
}

tr.clickable-row:hover td {
  background: rgba(56, 189, 248, 0.15);
  transform: scale(1.01);
}

tr td:first-child { border-top-left-radius: 8px; border-bottom-left-radius: 8px; font-weight: 600; color: #38bdf8; }
tr td:last-child { border-top-right-radius: 8px; border-bottom-right-radius: 8px; }

.item-name {
  cursor: pointer;
}

.table-input {
  background: transparent;
  border: 1px solid transparent;
  color: #e2e8f0;
  font-family: inherit;
  font-size: 1rem;
  text-align: center;
  width: 100%;
  padding: 0.2rem;
  border-radius: 6px;
  outline: none;
  transition: all 0.2s;
}

.table-input:hover, .table-input:focus {
  background: rgba(2, 6, 23, 0.6);
  border-color: #0ea5e9;
  cursor: text;
}

/* PROGRESS BAR STYLES */
.progress-section {
  margin-top: 1rem;
}

.progress-glow {
  border-color: rgba(16, 185, 129, 0.3) !important;
  box-shadow: 0 4px 30px rgba(0,0,0,0.3), 0 0 30px rgba(16, 185, 129, 0.15), inset 0 1px 0 rgba(255,255,255,0.1) !important;
}

.progress-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.2rem;
}

.progress-header .panel-subtitle {
  margin-bottom: 0;
  border: none;
  padding: 0;
}

.progress-value {
  font-size: 1.8rem;
  font-weight: 800;
  color: #10b981;
  text-shadow: 0 0 20px rgba(16, 185, 129, 0.8);
  letter-spacing: 1px;
}

.progress-bar-container {
  width: 100%;
  height: 46px;
  background: rgba(2, 6, 23, 0.8);
  border-radius: 23px;
  overflow: hidden;
  border: 1px solid rgba(16, 185, 129, 0.2);
  box-shadow: inset 0 2px 8px rgba(0,0,0,0.6), 0 0 20px rgba(16, 185, 129, 0.1);
}

.progress-bar-fill {
  height: 100%;
  background: linear-gradient(90deg, #047857, #059669, #34d399, #10b981);
  background-size: 300% 100%;
  border-radius: 23px;
  box-shadow: 0 0 25px rgba(16, 185, 129, 0.8), inset 0 1px 0 rgba(255,255,255,0.2);

  display: flex;
  align-items: center;
  justify-content: flex-end;
  padding-right: 1rem;
  min-width: 3.5rem;
}

.progress-label {
  color: rgba(255,255,255,0.9);
  font-size: 0.9rem;
  font-weight: 700;
  white-space: nowrap;
  text-shadow: 0 1px 3px rgba(0,0,0,0.5);
}

@keyframes shimmer {
  0% { background-position: 100% 0; }
  100% { background-position: -100% 0; }
}

/* Responsive */
@media (max-width: 1024px) {
  .inputs-horizontal {
    grid-template-columns: repeat(3, 1fr);
  }
  .bottom-grid {
    grid-template-columns: 1fr;
  }
}

@media (max-width: 640px) {
  .inputs-horizontal {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* ===== INPUTS HEADER ===== */
.inputs-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.2rem;
}

/* ===== PL DROPDOWN (select-style) ===== */
.pl-dropdown {
  display: flex;
  align-items: center;
  gap: 0.55rem;
  background: rgba(2, 6, 23, 0.6);
  border: 1px solid rgba(14, 165, 233, 0.4);
  border-radius: 10px;
  padding: 0.5rem 0.9rem;
  cursor: pointer;
  font-family: 'Outfit', sans-serif;
  font-size: 0.95rem;
  font-weight: 600;
  color: #38bdf8;
  transition: all 0.2s ease;
  min-width: 180px;
  user-select: none;
}

.pl-dropdown:hover {
  border-color: #0ea5e9;
  box-shadow: 0 0 10px rgba(14, 165, 233, 0.25);
  background: rgba(14, 165, 233, 0.08);
}

.pl-dropdown-icon {
  font-size: 0.9rem;
  opacity: 0.7;
}

.pl-dropdown-value {
  flex: 1;
  font-family: 'Outfit', sans-serif;
}

.pl-dropdown-chevron {
  width: 16px;
  height: 16px;
  color: #94a3b8;
  transition: transform 0.2s;
  flex-shrink: 0;
}

.pl-dropdown-chevron.open {
  transform: rotate(180deg);
}

/* ===== LOCKED INPUT ===== */
.input-locked {
  border-color: rgba(14, 165, 233, 0.5) !important;
  background: rgba(14, 165, 233, 0.06) !important;
  color: #38bdf8 !important;
  cursor: not-allowed !important;
  opacity: 0.85;
}

.lock-icon {
  font-size: 0.75rem;
  opacity: 0.7;
  vertical-align: middle;
}

/* =========== MODAL =========== */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(2, 6, 23, 0.75);
  backdrop-filter: blur(6px);
  z-index: 9999;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 1rem;
}

.modal-box {
  width: 100%;
  max-width: 520px;
  max-height: 80vh;
  overflow-y: auto;
  padding: 2rem;
  font-family: 'Outfit', sans-serif;
  background: rgba(15, 23, 42, 0.95) !important;
  border: 1px solid rgba(14, 165, 233, 0.25);
  box-shadow: 0 25px 60px rgba(0,0,0,0.7), 0 0 40px rgba(14, 165, 233, 0.1);
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  padding-bottom: 1rem;
  border-bottom: 1px solid rgba(255,255,255,0.08);
}

.modal-title {
  font-family: 'Outfit', sans-serif;
  font-size: 0.95rem;
  font-weight: 700;
  color: #38bdf8;
  text-transform: uppercase;
  letter-spacing: 1px;
  margin: 0;
}

.modal-close {
  background: transparent;
  border: none;
  color: #94a3b8;
  font-size: 1.6rem;
  cursor: pointer;
  line-height: 1;
  transition: color 0.2s;
  padding: 0;
}
.modal-close:hover { color: #f43f5e; }

/* PL List */
.modal-pl-list {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
  margin-bottom: 1.2rem;
}

.modal-pl-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.85rem 1rem 0.85rem 1.2rem;
  border-radius: 12px;
  background: rgba(30, 41, 59, 0.5);
  border: 1px solid transparent;
  border-left-width: 3px;
  cursor: pointer;
  transition: all 0.2s;
  font-family: 'Outfit', sans-serif;
}

.modal-pl-row:hover {
  background: rgba(14, 165, 233, 0.1);
  border-color: rgba(14, 165, 233, 0.3);
}

.modal-pl-row.active {
  background: rgba(14, 165, 233, 0.15);
  border-color: #0ea5e9;
  box-shadow: 0 0 12px rgba(14, 165, 233, 0.2);
}

.modal-pl-info {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}

.modal-pl-name {
  font-family: 'Outfit', sans-serif;
  font-weight: 600;
  font-size: 0.95rem;
  /* color set inline via :style */
}

.modal-pl-stats {
  font-family: 'Outfit', sans-serif;
  font-size: 0.82rem;
  font-weight: 400;
  /* color set inline via :style */
}

.modal-pl-actions {
  display: flex;
  gap: 0.4rem;
  align-items: center;
}

.btn-icon-edit, .btn-icon-delete {
  width: 30px;
  height: 30px;
  border-radius: 8px;
  border: none;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.2s;
}

.btn-icon-edit {
  background: rgba(99, 102, 241, 0.2);
  color: #818cf8;
  border: 1px solid rgba(99, 102, 241, 0.3);
}
.btn-icon-edit:hover { background: #6366f1; color: white; }

.btn-icon-delete {
  background: rgba(244, 63, 94, 0.15);
  color: #f43f5e;
  border: 1px solid rgba(244, 63, 94, 0.25);
}
.btn-icon-delete:hover { background: #f43f5e; color: white; }

.btn-add-new-pl {
  width: 100%;
  padding: 0.8rem;
  background: rgba(14, 165, 233, 0.08);
  border: 1px dashed rgba(14, 165, 233, 0.4);
  border-radius: 12px;
  color: #38bdf8;
  font-family: 'Outfit', sans-serif;
  font-size: 0.95rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.5rem;
}
.btn-add-new-pl:hover {
  background: rgba(14, 165, 233, 0.15);
  border-color: #0ea5e9;
}

/* Edit form inside modal */
.modal-edit-form {
  padding: 1rem;
  background: rgba(30, 41, 59, 0.5);
  border-radius: 14px;
  border: 1px solid rgba(255,255,255,0.06);
}

.modal-edit-title {
  font-size: 1rem;
  font-weight: 700;
  color: #f8fafc;
  margin: 0 0 1.2rem 0;
}

.modal-edit-fields {
  display: flex;
  flex-direction: column;
  gap: 0.9rem;
  margin-bottom: 1.2rem;
}

.modal-field label {
  display: block;
  font-size: 0.8rem;
  font-weight: 600;
  color: #94a3b8;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  margin-bottom: 0.4rem;
}

.modal-field input {
  width: 100%;
  box-sizing: border-box;
  background: rgba(2, 6, 23, 0.7);
  border: 1px solid rgba(255,255,255,0.1);
  color: #e2e8f0;
  padding: 0.65rem 0.9rem;
  border-radius: 9px;
  font-size: 0.95rem;
  font-family: inherit;
  outline: none;
  transition: border-color 0.2s;
}
.modal-field input:focus { border-color: #0ea5e9; }
.modal-field input:disabled { opacity: 0.45; cursor: not-allowed; }

.modal-edit-actions {
  display: flex;
  gap: 0.8rem;
  justify-content: flex-end;
}

.btn-modal-cancel {
  background: transparent;
  border: 1px solid rgba(255,255,255,0.15);
  color: #94a3b8;
  padding: 0.6rem 1.2rem;
  border-radius: 9px;
  cursor: pointer;
  font-family: inherit;
  transition: all 0.2s;
}
.btn-modal-cancel:hover { background: rgba(255,255,255,0.05); color: #e2e8f0; }

.btn-modal-save {
  background: #0ea5e9;
  border: none;
  color: white;
  padding: 0.6rem 1.4rem;
  border-radius: 9px;
  cursor: pointer;
  font-family: inherit;
  font-weight: 700;
  transition: background 0.2s;
}
.btn-modal-save:hover { background: #0284c7; }

/* Modal transition */
.modal-fade-enter-active, .modal-fade-leave-active {
  transition: opacity 0.2s ease;
}
.modal-fade-enter-from, .modal-fade-leave-to {
  opacity: 0;
}

/* ===== TOOLTIPS ===== */
.tooltip-wrap {
  position: relative;
  display: inline-flex;
  align-items: center;
  margin-left: 0.35rem;
  vertical-align: middle;
}

.tooltip-icon {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 15px;
  height: 15px;
  border-radius: 50%;
  background: rgba(14, 165, 233, 0.18);
  border: 1px solid rgba(14, 165, 233, 0.4);
  color: #38bdf8;
  font-size: 0.65rem;
  font-weight: 700;
  cursor: default;
  line-height: 1;
  transition: background 0.2s;
  user-select: none;
}

.tooltip-wrap:hover .tooltip-icon {
  background: rgba(14, 165, 233, 0.35);
}

.tooltip-box {
  visibility: hidden;
  opacity: 0;
  pointer-events: none;
  position: absolute;
  bottom: calc(100% + 10px);
  left: 50%;
  transform: translateX(-50%);
  min-width: 200px;
  background: rgba(15, 23, 42, 0.97);
  border: 1px solid rgba(14, 165, 233, 0.3);
  border-radius: 10px;
  padding: 0.6rem 0.9rem;
  font-size: 0.78rem;
  font-weight: 400;
  color: #cbd5e1;
  line-height: 1.5;
  white-space: normal;
  text-transform: none;
  letter-spacing: 0;
  box-shadow: 0 8px 32px rgba(0,0,0,0.5), 0 0 12px rgba(14,165,233,0.1);
  transition: opacity 0.18s ease, visibility 0.18s ease;
  z-index: 9999;
}

/* Arrow pointing down */
.tooltip-box::after {
  content: '';
  position: absolute;
  top: 100%;
  left: 50%;
  transform: translateX(-50%);
  border: 6px solid transparent;
  border-top-color: rgba(14, 165, 233, 0.3);
}

.tooltip-wrap:hover .tooltip-box {
  visibility: visible;
  opacity: 1;
}

.pl-images-container2 {
  margin-left: 600px;
  display: flex;
  justify-content: center;
  align-items: center;

}
</style>
