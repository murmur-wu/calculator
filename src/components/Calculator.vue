<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'

const MAX_DISPLAY_LENGTH = 13
const EXPONENTIAL_PRECISION = 4

const display = ref('0')
const expression = ref('')
const operator = ref(null)
const prevValue = ref(null)
const waitingForOperand = ref(false)
const justCalculated = ref(false)

const displayValue = computed(() => {
  if (display.value.length > MAX_DISPLAY_LENGTH - 1) {
    const num = parseFloat(display.value)
    if (isNaN(num)) return display.value
    return num.toExponential(EXPONENTIAL_PRECISION)
  }
  return display.value
})

function inputDigit(digit) {
  if (justCalculated.value) {
    display.value = String(digit)
    expression.value = String(digit)
    justCalculated.value = false
    waitingForOperand.value = false
    return
  }
  if (waitingForOperand.value) {
    display.value = String(digit)
    waitingForOperand.value = false
  } else {
    if (display.value === '0' && digit !== '.') {
      display.value = String(digit)
    } else {
      if (display.value.length < MAX_DISPLAY_LENGTH) {
        display.value = display.value + digit
      }
    }
  }
  expression.value = (expression.value || '') + digit
}

function inputDecimal() {
  if (justCalculated.value) {
    display.value = '0.'
    expression.value = '0.'
    justCalculated.value = false
    waitingForOperand.value = false
    return
  }
  if (waitingForOperand.value) {
    display.value = '0.'
    waitingForOperand.value = false
  } else if (!display.value.includes('.')) {
    display.value = display.value + '.'
  }
  if (!expression.value.endsWith('.') && !waitingForOperand.value) {
    expression.value = (expression.value || '') + '.'
  }
}

function handleOperator(op) {
  const current = parseFloat(display.value)

  if (prevValue.value !== null && !waitingForOperand.value && !justCalculated.value) {
    const result = calculate(prevValue.value, current, operator.value)
    display.value = String(result)
    prevValue.value = result
    expression.value = String(result) + ' ' + op + ' '
  } else {
    prevValue.value = current
    if (justCalculated.value) {
      expression.value = display.value + ' ' + op + ' '
    } else {
      expression.value = (expression.value || display.value) + ' ' + op + ' '
    }
  }

  operator.value = op
  waitingForOperand.value = true
  justCalculated.value = false
}

function calculate(a, b, op) {
  switch (op) {
    case '+': return parseFloat((a + b).toPrecision(12))
    case '−': return parseFloat((a - b).toPrecision(12))
    case '×': return parseFloat((a * b).toPrecision(12))
    case '÷': return b !== 0 ? parseFloat((a / b).toPrecision(12)) : 'Error'
    default: return b
  }
}

function handleEquals() {
  if (operator.value === null || waitingForOperand.value) return

  const current = parseFloat(display.value)
  const result = calculate(prevValue.value, current, operator.value)

  expression.value = expression.value + ' ='
  display.value = String(result)
  prevValue.value = null
  operator.value = null
  waitingForOperand.value = false
  justCalculated.value = true
}

function handleClear() {
  display.value = '0'
  expression.value = ''
  operator.value = null
  prevValue.value = null
  waitingForOperand.value = false
  justCalculated.value = false
}

function handleToggleSign() {
  if (display.value !== '0') {
    if (display.value.startsWith('-')) {
      display.value = display.value.slice(1)
    } else {
      display.value = '-' + display.value
    }
  }
}

function handlePercent() {
  const value = parseFloat(display.value)
  display.value = String(parseFloat((value / 100).toPrecision(12)))
  justCalculated.value = false
}

function handleKeydown(event) {
  let handled = true
  if (event.key >= '0' && event.key <= '9') {
    inputDigit(event.key)
  } else if (event.key === '.') {
    inputDecimal()
  } else if (event.key === '+') {
    handleOperator('+')
  } else if (event.key === '-') {
    handleOperator('−')
  } else if (event.key === '*') {
    handleOperator('×')
  } else if (event.key === '/') {
    handleOperator('÷')
  } else if (event.key === 'Enter' || event.key === '=') {
    handleEquals()
  } else if (event.key === 'Escape') {
    handleClear()
  } else if (event.key === '%') {
    handlePercent()
  } else {
    handled = false
  }
  if (handled) event.preventDefault()
}

onMounted(() => {
  window.addEventListener('keydown', handleKeydown)
})

onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown)
})
</script>

<template>
  <div class="flex items-center justify-center min-h-screen bg-gray-900 p-4">
    <div class="w-full max-w-xs bg-gray-800 rounded-3xl shadow-2xl overflow-hidden">
      <!-- Display -->
      <div class="px-6 pt-8 pb-4 text-right">
        <div class="text-gray-400 text-sm h-6 truncate">{{ expression || '\u00a0' }}</div>
        <div class="text-white text-5xl font-light tracking-wider mt-1 truncate">
          {{ displayValue }}
        </div>
      </div>

      <!-- Buttons Grid -->
      <div class="grid grid-cols-4 gap-px bg-gray-700 border-t border-gray-700">
        <!-- Row 1: function buttons -->
        <button class="flex items-center justify-center h-20 bg-gray-500 text-gray-900 text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-400 active:brightness-75" @click="handleClear">AC</button>
        <button class="flex items-center justify-center h-20 bg-gray-500 text-gray-900 text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-400 active:brightness-75" @click="handleToggleSign">+/−</button>
        <button class="flex items-center justify-center h-20 bg-gray-500 text-gray-900 text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-400 active:brightness-75" @click="handlePercent">%</button>
        <button :class="['flex items-center justify-center h-20 text-2xl font-medium cursor-pointer select-none transition-all duration-100', operator === '÷' && waitingForOperand ? 'bg-white text-orange-500 hover:bg-gray-100' : 'bg-orange-500 text-white hover:bg-orange-400', 'active:brightness-75']" @click="handleOperator('÷')">÷</button>

        <!-- Row 2 -->
        <button class="flex items-center justify-center h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75" @click="inputDigit('7')">7</button>
        <button class="flex items-center justify-center h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75" @click="inputDigit('8')">8</button>
        <button class="flex items-center justify-center h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75" @click="inputDigit('9')">9</button>
        <button :class="['flex items-center justify-center h-20 text-2xl font-medium cursor-pointer select-none transition-all duration-100', operator === '×' && waitingForOperand ? 'bg-white text-orange-500 hover:bg-gray-100' : 'bg-orange-500 text-white hover:bg-orange-400', 'active:brightness-75']" @click="handleOperator('×')">×</button>

        <!-- Row 3 -->
        <button class="flex items-center justify-center h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75" @click="inputDigit('4')">4</button>
        <button class="flex items-center justify-center h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75" @click="inputDigit('5')">5</button>
        <button class="flex items-center justify-center h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75" @click="inputDigit('6')">6</button>
        <button :class="['flex items-center justify-center h-20 text-2xl font-medium cursor-pointer select-none transition-all duration-100', operator === '−' && waitingForOperand ? 'bg-white text-orange-500 hover:bg-gray-100' : 'bg-orange-500 text-white hover:bg-orange-400', 'active:brightness-75']" @click="handleOperator('−')">−</button>

        <!-- Row 4 -->
        <button class="flex items-center justify-center h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75" @click="inputDigit('1')">1</button>
        <button class="flex items-center justify-center h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75" @click="inputDigit('2')">2</button>
        <button class="flex items-center justify-center h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75" @click="inputDigit('3')">3</button>
        <button :class="['flex items-center justify-center h-20 text-2xl font-medium cursor-pointer select-none transition-all duration-100', operator === '+' && waitingForOperand ? 'bg-white text-orange-500 hover:bg-gray-100' : 'bg-orange-500 text-white hover:bg-orange-400', 'active:brightness-75']" @click="handleOperator('+')">+</button>

        <!-- Row 5 -->
        <button class="col-span-2 flex items-center justify-start pl-8 h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75 rounded-bl-3xl" @click="inputDigit('0')">0</button>
        <button class="flex items-center justify-center h-20 bg-gray-600 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-gray-500 active:brightness-75" @click="inputDecimal">.</button>
        <button class="flex items-center justify-center h-20 bg-orange-500 text-white text-2xl font-medium cursor-pointer select-none transition-all duration-100 hover:bg-orange-400 active:brightness-75 rounded-br-3xl" @click="handleEquals">=</button>
      </div>
    </div>
  </div>
</template>
