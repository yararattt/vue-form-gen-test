<template>
  <div id="app">
    <div class="container">
      <h1 class="main-title">ДЕМОНСТРАЦИОННЫЙ МОДУЛЬ АВТОГЕНЕРАЦИИ ФОРМ</h1>

      <div v-if="loading" class="status-box loading">
        ЗАГРУЗКА СХЕМЫ...
      </div>
      <div v-else-if="error" class="status-box error">
        <p>ОШИБКА: {{ error }}</p>
        <button @click="retryLoad" class="retry-button">ПОВТОРИТЬ</button>
      </div>
      <div v-else class="demo-section">
        <div class="form-section module-card">
          <h2>Модуль 01: Интерфейс ввода</h2>
          <p class="schema-info">
            Источник схемы: <strong>test.json</strong>
          </p>
          <FormGenerator
            :schema="formSchema"
            v-model="formData"
            @submit="handleSubmit"
          />
        </div>
        <div class="data-section module-card">
          <h2>Модуль 02: Контроль данных</h2>
         
          <div class="reactive-data">
             <h3>Реактивные данные (V-MODEL):</h3>
             <pre class="data-preview data-primary">{{ JSON.stringify(formData, null, 2) }}</pre>
          </div>
         
          <div class="schema-section">
            <h3>Текущая JSON-схема:</h3>
            <pre class="data-preview data-secondary">{{ JSON.stringify(formSchema, null, 2) }}</pre>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import FormGenerator from './components/FormGenerator.vue'
import { ref, onMounted } from 'vue'
import formSchemaFromJson from './data/test.json'

export default {
  name: 'App',
  components: {
    FormGenerator
  },
  setup() {
    const formSchema = ref({ fields: [] })
    const formData = ref({})
    const loading = ref(true)
    const error = ref('')
   
    const loadSchema = () => {
      loading.value = true
      error.value = ''
     
      setTimeout(() => {
        try {
          const jsonData = formSchemaFromJson
         
          if (!jsonData.fields || !Array.isArray(jsonData.fields)) {
            throw new Error('Неверная структура JSON: отсутствует массив fields')
          }
         
          const validFields = jsonData.fields.every(field =>
            field.type && field.label && field.model
          )
         
          if (!validFields) {
            throw new Error('Неверная структура полей: каждое поле должно иметь type, label и model')
          }
         
          formSchema.value = jsonData
          console.log('Схема загружена из test.json:', jsonData)
         
        } catch (err) {
          error.value = err.message
          console.error('Ошибка загрузки JSON:', err)
        } finally {
          loading.value = false
        }
      }, 500)
    }
   
    const retryLoad = () => {
      loadSchema()
    }
   
    const handleSubmit = () => {
      console.log('Форма отправлена! Данные формы:', formData.value)
      const message = document.createElement('div');
      message.textContent = 'ФОРМА УСПЕШНО ОТПРАВЛЕНА';
      message.style.cssText = `
        position: fixed; top: 20px; right: 20px; padding: 15px 30px;
        background: #FF3333; color: white; font-weight: 700;
        border-radius: 4px; z-index: 1000;
        text-transform: uppercase; font-size: 16px;
        box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
        animation: fadeout 3s forwards;
      `;
      document.body.appendChild(message);
      setTimeout(() => message.remove(), 3000);
     
      let style = document.getElementById('fadeout-style');
      if (!style) {
          style = document.createElement('style');
          style.id = 'fadeout-style';
          style.textContent = `@keyframes fadeout { 0% { opacity: 1; transform: translateY(0); } 80% { opacity: 1; } 100% { opacity: 0; transform: translateY(-10px); } }`;
          document.head.appendChild(style);
      }
    }
   
    onMounted(() => {
      loadSchema()
    })
   
    return {
      formSchema,
      formData,
      loading,
      error,
      retryLoad,
      handleSubmit
    }
  }
}
</script>
<style>

:root {
    --color-background: #252526;
    --color-text-primary: #cccccc;
    --color-border: #444444;
    --color-secondary-bg: #1e1e1e;
    --color-accent: #FF3333;
    --color-error: #FF5555;
    --color-code-bg: #121212;
    --color-info-bg: #333333;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', 'Arial', sans-serif; 
  background-color: var(--color-secondary-bg);
  color: var(--color-text-primary);
}

#app {
  min-height: 100vh;
  padding: 50px 20px;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
}

.main-title {
  text-align: center;
  margin-bottom: 60px;
  color: var(--color-text-primary);
  font-size: 32px;
  font-weight: 900;
  letter-spacing: normal;
  text-transform: uppercase;
  border-bottom: 4px solid var(--color-accent); 
  padding-bottom: 12px;
  max-width: 800px;
  margin-left: auto;
  margin-right: auto;
}

h2, h3 {
  margin-bottom: 20px;
  color: var(--color-text-primary);
  font-weight: 700;
  text-transform: none;
  letter-spacing: 0;
}

h2 {
  font-size: 24px;
  border-bottom: 1px solid var(--color-border);
  padding-bottom: 10px;
}

h3 {
  font-size: 18px;
  font-weight: 600;
  margin-top: 25px;
  border-bottom: none;
  color: #999;
}


.status-box {
  text-align: center;
  padding: 40px;
  margin: 20px 0;
  font-size: 18px;
  font-weight: 700;
  text-transform: uppercase;
  border-radius: 4px;
}

.loading {
  color: #ffffff;
  background: #333300; 
  border: 1px solid #ffcc00;
}

.error {
  background-color: #550000; 
  color: var(--color-error);
  border: 1px solid var(--color-error);
}

.retry-button {
  margin-top: 15px;
  padding: 10px 20px;
  background-color: var(--color-error);
  color: var(--color-background);
  border: none;
  border-radius: 4px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.15s;
  text-transform: uppercase;
}

.retry-button:hover {
  background-color: var(--color-accent);
}

.demo-section {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 40px;
  align-items: start;
}

.module-card {
  background: var(--color-background);
  padding: 30px;
  border: 1px solid var(--color-border);
  border-radius: 6px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
}

.schema-info {
  background: var(--color-info-bg);
  color: var(--color-text-primary);
  padding: 10px 15px;
  border-left: 4px solid var(--color-accent);
  margin-bottom: 25px;
  font-size: 14px;
}


.data-preview {
  background: var(--color-code-bg); 
  color: #7af8b0;
  padding: 15px;
  border-radius: 4px;
  border: 1px solid var(--color-border);
  font-family: 'Courier New', monospace;
  font-size: 13px;
  max-height: 300px;
  overflow-y: auto;
  white-space: pre-wrap;
  word-wrap: break-word;
  line-height: 1.4;
  margin-bottom: 20px;
}

.data-secondary {
  border-left: 4px solid var(--color-error);
  color: #ff9999;
}

@media (max-width: 768px) {
  .demo-section {
    grid-template-columns: 1fr;
    gap: 30px;
  }
 
  #app {
    padding: 20px 10px;
  }
}
</style>