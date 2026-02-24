<template>
  <header class="app-header">
    <div class="app-header-brand">
      <img src="../data/images/redemption-logo.png" alt="Redemption Logo" class="app-header-logo-img" />
      <span class="app-header-title">Redemption Tools</span>
      <button class="nav-btn active">Calculadora de XP</button>
    </div>
    <span class="app-header-credits">Created by Glifo &amp; Dacero</span>
  </header>
</template>

<script>
export default {
  name: 'AppHeader',
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Outfit:wght@300;400;600;700&display=swap');

.app-header {
  position: sticky;
  top: 0;
  z-index: 200;
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0.8rem 2.5rem;
  background: rgba(8, 14, 36, 0.92);
  border-bottom: 1px solid rgba(14, 165, 233, 0.2);
  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);
}

.app-header-brand {
  display: flex;
  align-items: center;
  gap: 1.2rem;
}

.app-header-logo-img {
  height: 38px;
  width: auto;
  object-fit: contain;
  filter: drop-shadow(0 2px 4px rgba(0,0,0,0.4));
}

.app-header-title {
  font-family: 'Outfit', sans-serif;
  font-size: 1.1rem;
  font-weight: 700;
  color: #ffffff;
  letter-spacing: 0.5px;
  text-transform: uppercase;
}

.nav-btn {
  background: rgba(14, 165, 233, 0.15);
  border: 1px solid rgba(14, 165, 233, 0.4);
  color: #38bdf8;
  padding: 0.4rem 1.2rem;
  border-radius: 20px;
  font-family: 'Outfit', sans-serif;
  font-weight: 600;
  font-size: 0.9rem;
  cursor: pointer;
  transition: all 0.2s;
}

.nav-btn.active {
  background: rgba(14, 165, 233, 0.25);
  border-color: #0ea5e9;
  box-shadow: 0 0 10px rgba(14, 165, 233, 0.3);
  color: #fff;
}

.app-header-credits {
  font-family: 'Outfit', sans-serif;
  font-size: 0.85rem;
  color: #64748b;
  font-weight: 400;
}
</style>
