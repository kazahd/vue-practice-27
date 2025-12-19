<template>
  <div class="palette-generator">
    <h2> Генератор цветовых палитр</h2>
    <p class="subtitle">Создавайте, настраивайте и сохраняйте гармоничные цветовые схемы</p>

    <!-- Панель управления -->
    <div class="controls">
      <button @click="generateRandomPalette" class="generate-btn">
        Случайная палитра
      </button>
      
      <div class="control-group">
        <label>Количество цветов:</label>
        <div class="count-selector">
          <button 
            v-for="count in [3, 5, 7]" 
            :key="count"
            @click="colorCount = count"
            :class="{ active: colorCount === count }"
            class="count-btn"
          >
            {{ count }}
          </button>
        </div>
      </div>
      
      <div class="control-group">
        <label>Формат отображения:</label>
        <div class="format-selector">
          <button 
            @click="displayFormat = 'hex'"
            :class="{ active: displayFormat === 'hex' }"
            class="format-btn"
          >
            HEX
          </button>
          <button 
            @click="displayFormat = 'rgb'"
            :class="{ active: displayFormat === 'rgb' }"
            class="format-btn"
          >
            RGB
          </button>
        </div>
      </div>
    </div>

    <!-- Отображение палитры -->
    <div class="palette-container">
      <div 
        v-for="(color, index) in palette" 
        :key="index"
        class="color-card"
        :style="{ backgroundColor: color.hex }"
        @click="copyToClipboard(color)"
        :class="{ locked: color.locked }"
      >
        <div class="color-info">
          <div class="color-value">
            {{ displayFormat === 'hex' ? color.hex : color.rgb }}
          </div>
          <button 
            @click.stop="toggleLock(index)"
            class="lock-btn"
            :title="color.locked ? 'Разблокировать' : 'Закрепить'"
          >
            {{ color.locked ? '🔒' : '🔓' }}
          </button>
        </div>
        <div v-if="color.copied" class="copy-notification">
          ✓ Скопировано!
        </div>
      </div>
    </div>

    <!-- Превью интерфейса -->
    <div class="preview-section">
      <h3>Превью палитры в интерфейсе</h3>
      
      <div class="preview-controls">
        <button 
          @click="previewBackground = previewBackground === 'light' ? 'dark' : 'light'"
          class="preview-toggle"
        >
          Фон: {{ previewBackground === 'light' ? 'Светлый' : 'Тёмный' }}
        </button>
      </div>
      
      <div class="preview-container" :class="previewBackground">
        <div class="preview-mockup">
          <div class="mockup-header" :style="{ backgroundColor: palette[0]?.hex || '#667eea' }">
            <h3>Заголовок приложения</h3>
            <p>Описание интерфейса с использованием цветов палитры</p>
          </div>
          
          <div class="mockup-content">
            <div class="mockup-card" :style="{ 
              backgroundColor: palette[1]?.hex || '#764ba2',
              color: getTextColor(palette[1]?.hex || '#764ba2')
            }">
              <h4>Карточка</h4>
              <p>Пример контента с контрастным текстом</p>
              <button class="mockup-btn" :style="{ 
                backgroundColor: palette[2]?.hex || '#f093fb',
                color: getTextColor(palette[2]?.hex || '#f093fb')
              }">
                Кнопка действия
              </button>
            </div>
            
            <button class="mockup-primary-btn" :style="{ 
              backgroundColor: palette[3]?.hex || '#4facfe'
            }">
              Основная кнопка
            </button>
            
            <div class="mockup-footer" :style="{ backgroundColor: palette[4]?.hex || '#f093fb' }">
              <p>Футер интерфейса</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Информация о сохранении -->
    <div class="save-info">
      <p v-if="isSaved">Палитра сохранена в localStorage</p>
      <p v-else>Изменения будут сохранены автоматически</p>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, watch } from 'vue'

export default {
  name: 'ColorPaletteGenerator',
  
  setup() {
    // Реактивные переменные
    const colorCount = ref(5)
    const displayFormat = ref('hex')
    const previewBackground = ref('light')
    const palette = ref([])
    const isSaved = ref(false)

    // Генерация случайного цвета в формате HEX
    const generateRandomColor = () => {
      return '#' + Math.floor(Math.random() * 16777215).toString(16).padStart(6, '0')
    }

    // Преобразование HEX в RGB
    const hexToRgb = (hex) => {
      const r = parseInt(hex.slice(1, 3), 16)
      const g = parseInt(hex.slice(3, 5), 16)
      const b = parseInt(hex.slice(5, 7), 16)
      return `rgb(${r}, ${g}, ${b})`
    }

    // Генерация гармоничной палитры
    const generateHarmoniousPalette = () => {
      const baseHue = Math.floor(Math.random() * 360)
      const newPalette = []
      
      for (let i = 0; i < colorCount.value; i++) {
        // Пропускаем закреплённые цвета
        if (palette.value[i]?.locked) {
          newPalette.push(palette.value[i])
          continue
        }
        
        // Генерация гармоничного цвета
        const hue = (baseHue + (i * 60)) % 360
        const saturation = 50 + Math.random() * 30
        const lightness = 40 + Math.random() * 30
        
        // Преобразование HSL в HEX
        const hex = hslToHex(hue, saturation, lightness)
        const rgb = hexToRgb(hex)
        
        newPalette.push({
          hex,
          rgb,
          locked: false,
          copied: false
        })
      }
      
      palette.value = newPalette
      isSaved.value = false
    }

    // Преобразование HSL в HEX
    const hslToHex = (h, s, l) => {
      h /= 360
      s /= 100
      l /= 100
      
      let r, g, b
      
      if (s === 0) {
        r = g = b = l
      } else {
        const hue2rgb = (p, q, t) => {
          if (t < 0) t += 1
          if (t > 1) t -= 1
          if (t < 1/6) return p + (q - p) * 6 * t
          if (t < 1/2) return q
          if (t < 2/3) return p + (q - p) * (2/3 - t) * 6
          return p
        }
        
        const q = l < 0.5 ? l * (1 + s) : l + s - l * s
        const p = 2 * l - q
        
        r = hue2rgb(p, q, h + 1/3)
        g = hue2rgb(p, q, h)
        b = hue2rgb(p, q, h - 1/3)
      }
      
      const toHex = x => {
        const hex = Math.round(x * 255).toString(16)
        return hex.length === 1 ? '0' + hex : hex
      }
      
      return `#${toHex(r)}${toHex(g)}${toHex(b)}`
    }

    // Копирование в буфер обмена
    const copyToClipboard = async (color) => {
      const text = displayFormat.value === 'hex' ? color.hex : color.rgb
      
      try {
        await navigator.clipboard.writeText(text)
        color.copied = true
        
        // Сброс уведомления через 2 секунды
        setTimeout(() => {
          color.copied = false
        }, 2000)
      } catch (err) {
        console.error('Ошибка копирования:', err)
      }
    }

    // Закрепление/разблокировка цвета
    const toggleLock = (index) => {
      palette.value[index].locked = !palette.value[index].locked
    }

    // Определение цвета текста для контраста
    const getTextColor = (hexColor) => {
      // Простая проверка яркости цвета
      const r = parseInt(hexColor.slice(1, 3), 16)
      const g = parseInt(hexColor.slice(3, 5), 16)
      const b = parseInt(hexColor.slice(5, 7), 16)
      const brightness = (r * 299 + g * 587 + b * 114) / 1000
      return brightness > 128 ? '#000' : '#fff'
    }

    // Сохранение в localStorage
    const saveToLocalStorage = () => {
      try {
        const saveData = {
          palette: palette.value,
          colorCount: colorCount.value,
          displayFormat: displayFormat.value,
          timestamp: new Date().getTime()
        }
        localStorage.setItem('colorPalette', JSON.stringify(saveData))
        isSaved.value = true
        
        // Сброс флага сохранения через 3 секунды
        setTimeout(() => {
          isSaved.value = false
        }, 3000)
      } catch (err) {
        console.error('Ошибка сохранения:', err)
      }
    }

    // Загрузка из localStorage
    const loadFromLocalStorage = () => {
      try {
        const saved = localStorage.getItem('colorPalette')
        if (saved) {
          const data = JSON.parse(saved)
          palette.value = data.palette || []
          colorCount.value = data.colorCount || 5
          displayFormat.value = data.displayFormat || 'hex'
        } else {
          generateHarmoniousPalette()
        }
      } catch (err) {
        console.error('Ошибка загрузки:', err)
        generateHarmoniousPalette()
      }
    }

    // Watchers для автоматического сохранения
    watch(palette, saveToLocalStorage, { deep: true })
    watch(colorCount, generateHarmoniousPalette)
    watch(displayFormat, saveToLocalStorage)

    // Инициализация
    onMounted(() => {
      loadFromLocalStorage()
    })

    // Публичные методы
    const generateRandomPalette = () => {
      generateHarmoniousPalette()
    }

    return {
      colorCount,
      displayFormat,
      previewBackground,
      palette,
      isSaved,
      generateRandomPalette,
      copyToClipboard,
      toggleLock,
      getTextColor
    }
  }
}
</script>

<style scoped>
.palette-generator {
  max-width: 1000px;
  margin: 0 auto;
  padding: 20px;
}

.subtitle {
  color: #666;
  margin-bottom: 30px;
  text-align: center;
}

/* Панель управления */
.controls {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
  margin-bottom: 30px;
  padding: 20px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.control-group {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

.generate-btn {
  padding: 12px 24px;
  background-color: #ff6b9d;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  transition: transform 0.3s ease;
}

.generate-btn:hover {
  transform: translateY(-2px);
}

.count-selector, .format-selector {
  display: flex;
  gap: 10px;
}

.count-btn, .format-btn {
  padding: 8px 16px;
  border: 2px solid #667eea;
  background: white;
  color: #667eea;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.count-btn.active, .format-btn.active {
  background: #667eea;
  color: white;
}

/* Отображение палитры */
.palette-container {
  display: flex;
  gap: 10px;
  margin-bottom: 40px;
  height: 150px;
}

.color-card {
  flex: 1;
  border-radius: 8px;
  position: relative;
  cursor: pointer;
  transition: transform 0.3s ease, box-shadow 0.3s ease;
  overflow: hidden;
}

.color-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 20px rgba(0,0,0,0.2);
}

.color-card.locked {
  border: 3px solid #333;
}

.color-info {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  background: rgba(0, 0, 0, 0.7);
  color: white;
  padding: 10px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.color-value {
  font-family: 'Courier New', monospace;
  font-size: 14px;
  font-weight: bold;
}

.lock-btn {
  background: none;
  border: none;
  color: white;
  cursor: pointer;
  font-size: 16px;
  padding: 5px;
}

.copy-notification {
  position: absolute;
  top: 10px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(255, 255, 255, 0.9);
  color: #333;
  padding: 5px 10px;
  border-radius: 15px;
  font-size: 12px;
  animation: fadeInOut 2s ease;
}

@keyframes fadeInOut {
  0%, 100% { opacity: 0; }
  20%, 80% { opacity: 1; }
}

/* Превью интерфейса */
.preview-section {
  background: white;
  border-radius: 10px;
  padding: 20px;
  margin-bottom: 30px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

.preview-controls {
  margin-bottom: 20px;
}

.preview-toggle {
  padding: 10px 20px;
  background: #f0f0f0;
  border: none;
  border-radius: 20px;
  cursor: pointer;
  font-size: 14px;
}

.preview-container {
  border-radius: 8px;
  overflow: hidden;
}

.preview-container.light {
  background: #f8f9fa;
}

.preview-container.dark {
  background: #333;
}

.preview-mockup {
  width: 100%;
}

.mockup-header {
  padding: 20px;
  color: white;
}

.mockup-content {
  padding: 20px;
}

.mockup-card {
  padding: 20px;
  border-radius: 8px;
  margin-bottom: 20px;
}

.mockup-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  margin-top: 10px;
}

.mockup-primary-btn {
  padding: 12px 24px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  font-weight: bold;
  display: block;
  margin: 20px auto;
}

.mockup-footer {
  padding: 15px;
  color: white;
  text-align: center;
  margin-top: 20px;
}

/* Информация о сохранении */
.save-info {
  text-align: center;
  padding: 15px;
  background: #e9ecef;
  border-radius: 8px;
  font-size: 14px;
}
</style>