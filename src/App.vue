  <template>
    <div id="app">
      <header class="app-header">
        <h1>Vue 3 Практика - Генератор цветовых палитр</h1>
        <p>Изучаем реактивность, директивы и компоненты</p>
      </header>

      <main class="main-content">
        <ColorPaletteGenerator />
      </main>

      <footer class="app-footer">
        <p>Vue 3 + Vite • Практика 27 • Генератор цветовых палитр</p>
      </footer>
    </div>
  </template>

  <script>
  import ColorPaletteGenerator from './components/ColorPaletteGenerator.vue'

  export default {
    name: 'App',
    
    components: {
      ColorPaletteGenerator
    }
  }
  </script>

  <style scoped>
  .app-header {
    background-color: #ff6b9d; /* Розовый цвет */
    color: white;
    padding: 2rem;
    text-align: center;
  }

  .app-header h1 {
    margin-bottom: 0.5rem;
    font-size: 2.5rem;
  }

  .app-header p {
    opacity: 0.9;
    font-size: 1.1rem;
  }

  .main-content {
    flex: 1;
    padding: 2rem;
    max-width: 1200px;
    margin: 0 auto;
    width: 100%;
  }

  .app-footer {
    background-color: #333;
    color: white;
    text-align: center;
    padding: 1rem;
    margin-top: auto;
  }
  </style>