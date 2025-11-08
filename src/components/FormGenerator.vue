<template>
  <form class="auto-form" @submit.prevent="$emit('submit')">
    <div 
      v-for="field in schema.fields" 
      :key="field.model"
      class="form-field"
    >
      <div v-if="['text', 'email', 'password'].includes(field.type)" class="input-group">
        <label :for="field.model" class="label">
          {{ field.label }}
          <span v-if="field.required" class="required">*</span>
        </label>
        <input
          :id="field.model"
          :type="field.type"
          :value="formValue[field.model]"
          @input="updateField(field.model, $event.target.value)"
          @blur="validateField(field)"
          :class="['input', { 'error': errors[field.model] }]"
          :placeholder="field.placeholder || ''"
        />
        <div v-if="errors[field.model]" class="error-message">
          {{ errors[field.model] }}
        </div>
      </div>

      <div v-else-if="field.type === 'select'" class="input-group">
        <label :for="field.model" class="label">
          {{ field.label }}
          <span v-if="field.required" class="required">*</span>
        </label>
        <select
          :id="field.model"
          :value="formValue[field.model]"
          @change="updateField(field.model, $event.target.value)"
          @blur="validateField(field)"
          :class="['select', { 'error': errors[field.model] }]"
        >
          <option value="" disabled>Выберите опцию</option>
          <option 
            v-for="option in field.options" 
            :key="option" 
            :value="option"
          >
            {{ option }}
          </option>
        </select>
        <div v-if="errors[field.model]" class="error-message">
          {{ errors[field.model] }}
        </div>
      </div>

      <div v-else-if="field.type === 'checkbox'" class="checkbox-simple">
        <label class="checkbox-label">
          <input
            :id="field.model"
            type="checkbox"
            :checked="formValue[field.model]"
            @change="updateField(field.model, $event.target.checked)"
            @blur="validateField(field)"
            :class="{ 'error': errors[field.model] }"
          />
          <span class="checkbox-custom" aria-hidden="true"></span>
          <span class="checkbox-text">
            {{ field.label }}
            <span v-if="field.required" class="required">*</span>
          </span>
        </label>
        <div v-if="errors[field.model]" class="error-message">
          {{ errors[field.model] }}
        </div>
      </div>
    </div>

    <button 
      type="submit" 
      class="submit-button"
      :disabled="!isFormValid"
    >
      Отправить
    </button>
  </form>
</template>

<script>
import { ref, watch, computed } from 'vue'

export default {
  name: 'FormGenerator',
  props: {
    schema: {
      type: Object,
      required: true
    },
    modelValue: {
      type: Object,
      default: () => ({})
    }
  },
  emits: ['update:modelValue', 'submit'],
  setup(props, { emit }) {
    const formValue = ref({ ...props.modelValue })
    const errors = ref({})

    const initializeForm = () => {
      props.schema.fields.forEach(field => {
        if (formValue.value[field.model] === undefined) {
          if (field.type === 'checkbox') {
            formValue.value[field.model] = false
          } else if (field.type === 'select') {
            formValue.value[field.model] = ''
          } else {
            formValue.value[field.model] = ''
          }
        }
      })
    }

    initializeForm()

    const updateField = (fieldName, value) => {
      formValue.value[fieldName] = value
      emit('update:modelValue', { ...formValue.value })
      validateField(props.schema.fields.find(f => f.model === fieldName))
    }

    const validateField = (field) => {
      const value = formValue.value[field.model]
      let error = ''

      if (field.required) {
        if (field.type === 'checkbox' && !value) {
          error = 'Это поле обязательно для заполнения'
        } else if (field.type !== 'checkbox' && (!value || (typeof value === 'string' && value.trim() === ''))) {
          error = 'Это поле обязательно для заполнения'
        }
      }

      if (!error && value && field.minLength && (typeof value === 'string' && value.length < field.minLength)) {
        error = `Минимальная длина: ${field.minLength} символов`
      }

      if (!error && value && field.pattern && typeof value === 'string') {
        const regex = new RegExp(field.pattern)
        if (!regex.test(value)) {
          error = field.patternMessage || 'Неверный формат'
        }
      }

      if (error) {
        errors.value[field.model] = error
      } else {
        delete errors.value[field.model]
      }
    }

    const validateForm = () => {
      props.schema.fields.forEach(field => {
        validateField(field)
      })
    }

    const isFormValid = computed(() => {
        const hasErrors = Object.keys(errors.value).length > 0;
        
        const allRequiredFilled = props.schema.fields.every(field => {
            if (!field.required) return true;
            const value = formValue.value[field.model];
            if (field.type === 'checkbox') return value === true;
            return value !== '' && value !== null && (typeof value !== 'string' || value.trim() !== '');
        });

        return !hasErrors && allRequiredFilled;
    })

    watch(() => props.schema, () => {
      initializeForm()
      validateForm()
    }, { deep: true })

    watch(() => props.modelValue, (newValue) => {
      formValue.value = { ...newValue }
      validateForm()
    }, { deep: true })

    return {
      formValue,
      errors,
      updateField,
      validateField,
      isFormValid
    }
  }
}
</script>

<style scoped>

:root {
    --color-background: #252526;
    --color-text-primary: #cccccc;
    --color-border: #444444;
    --color-placeholder: #888888;
    --color-accent: #FF3333;
    --color-error: #FF5555;
    --color-disabled: #3c3c3c;
    --color-input-bg: #3c3c3c;
}

.auto-form {
    max-width: 500px;
    margin: 40px auto;
    padding: 0;    
    background-color: transparent;
    border: none;
    font-family: 'Inter', 'Arial', sans-serif; 
    color: var(--color-text-primary);
    box-shadow: none;
}

.form-field {
    margin-bottom: 25px;
}

.input-group {
    display: flex;
    flex-direction: column;
}

.label {
    font-weight: 600;
    margin-bottom: 6px;
    font-size: 16px;
    color: var(--color-text-primary);
}

.required {
    color: var(--color-accent);
    font-weight: 800;
}

.input, .select {
    padding: 12px 10px;
    border: 1px solid var(--color-border);
    border-radius: 4px;
    background-color: var(--color-input-bg);
    color: var(--color-text-primary);
    font-size: 16px;
    line-height: 1.5;
    transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
}

.input::placeholder {
    color: var(--color-placeholder);
    opacity: 1;
}

.input:focus, .select:focus {
    outline: none;
    border-color: var(--color-accent);
    box-shadow: 0 0 0 1px var(--color-accent), 0 0 5px rgba(255, 51, 51, 0.5); 
}

.input.error, .select.error {
    border-color: var(--color-error);
}

.select {
    appearance: none;
    background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 20 20' fill='%23cccccc'%3E%3Cpath d='M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z' clip-rule='evenodd' fill-rule='evenodd'/%3E%3C/svg%3E");
    background-repeat: no-repeat;
    background-position: right 10px center;
    padding-right: 30px;
}

.checkbox-simple {
    margin-top: 10px;
    margin-bottom: 30px;
}

.checkbox-label {
    display: flex;
    align-items: center;
    gap: 12px;
    cursor: pointer;
}

.checkbox-label input[type="checkbox"] {
    opacity: 0;
    position: absolute;
    width: 0;
    height: 0;
}

.checkbox-custom {
    width: 28px; 
    height: 28px; 
    flex-shrink: 0; 
    border: 2px solid var(--color-border);
    border-radius: 4px;
    background-color: var(--color-input-bg);
    position: relative;
    transition: background-color 0.15s, border-color 0.15s;
}

.checkbox-custom::after {
    content: '';
    position: absolute;
    top: 50%;
    left: 50%;
    width: 10px; 
    height: 18px;
    border: solid var(--color-background);
    border-width: 0 3px 3px 0;
    transform: translate(-50%, -60%) rotate(45deg) scale(0); 
    transition: transform 0.2s ease-in-out;
}

.checkbox-label input[type="checkbox"]:checked + .checkbox-custom {
    background-color: var(--color-accent);
    border-color: var(--color-accent);
}

.checkbox-label input[type="checkbox"]:checked + .checkbox-custom::after {
    transform: translate(-50%, -60%) rotate(45deg) scale(1);
}

.checkbox-label input[type="checkbox"]:focus-visible + .checkbox-custom {
    box-shadow: 0 0 0 2px rgba(255, 51, 51, 0.5);
}

.checkbox-text {
    font-size: 17px;
    line-height: 1.4;
    color: var(--color-text-primary);
}

.error-message {
    color: var(--color-error);
    font-size: 14px;
    margin-top: 5px;
    font-weight: 400;
}

.submit-button {
    width: 100%;
    padding: 18px;
    background-color: var(--color-accent);
    color: var(--color-background);
    border: none;
    border-radius: 4px;
    font-size: 18px;
    font-weight: 700;
    cursor: pointer;
    text-transform: uppercase;
    letter-spacing: 1px;
    transition: background-color 0.15s ease-in-out, opacity 0.15s ease-in-out;
    box-shadow: 0 4px 15px rgba(255, 51, 51, 0.3); 
}

.submit-button:hover:not(:disabled) {
    background-color: #e62e2e;
    box-shadow: 0 6px 20px rgba(255, 51, 51, 0.4);
}

.submit-button:disabled {
    background-color: var(--color-disabled);
    color: #777777;
    cursor: not-allowed;
    box-shadow: none;
}
</style>