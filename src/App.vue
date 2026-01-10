<template>
  <div class="min-h-screen py-8 px-4">
    <div class="max-w-4xl mx-auto">
      <!-- Header -->
      <div class="text-center mb-8">
        <h1 class="text-4xl font-bold text-white mb-2">數字轉國字大寫</h1>
        <p class="text-white/80 text-lg">新台幣金融表示轉換工具</p>
      </div>

      <!-- Main Card -->
      <div class="bg-white rounded-2xl shadow-2xl p-8 mb-6">
        <!-- Input Section -->
        <div class="mb-6">
          <label class="block text-gray-700 font-medium mb-3 text-lg">
            請輸入金額
          </label>
          <div class="flex items-center gap-3">
            <span class="text-2xl font-bold text-gray-700">NT$</span>
            <input
              v-model.number="amount"
              type="number"
              step="1"
              min="0"
              max="99999999999"
              placeholder="請輸入數字"
              class="flex-1 text-2xl px-6 py-4 border-2 border-purple-200 rounded-xl focus:border-purple-500 focus:outline-none focus:ring-2 focus:ring-purple-200 transition-all"
            />
            <span class="text-2xl font-bold text-gray-700">元</span>
          </div>
        </div>

        <!-- Number Reference Table -->
        <div class="mb-8 bg-gradient-to-r from-amber-50 to-yellow-50 border-2 border-amber-200 rounded-xl p-4">
          <div class="grid grid-cols-5 sm:grid-cols-10 gap-2">
            <div v-for="(big, index) in BIGS" :key="index" 
                 class="flex flex-col items-center justify-center bg-white rounded-lg p-2 border border-amber-300">
              <span class="text-lg font-bold text-purple-700">{{ big }}</span>
              <span class="text-lg text-gray-500 mt-1">{{ index }}</span>
            </div>
          </div>
        </div>

        <!-- Output: 數字大寫 -->
        <div class="mb-8">
          <h2 class="text-xl font-bold text-gray-800 mb-3">數字大寫</h2>
          <div class="bg-gradient-to-r from-purple-50 to-pink-50 border-2 border-purple-200 rounded-xl p-6 cursor-pointer hover:shadow-lg transition-shadow group" 
               @click="selectChineseText"
               title="點擊複製">
            <p ref="chineseTextRef" class="text-2xl text-center text-gray-800 tracking-wider leading-relaxed select-all">
              {{ chineseAmount }}
            </p>
            <p class="text-xs text-center text-gray-500 mt-2 opacity-0 group-hover:opacity-100 transition-opacity">
              點擊文字以全選
            </p>
          </div>
        </div>

        <!-- Output: 填空 (大寫) -->
        <div class="mb-8">
          <h2 class="text-xl font-bold text-gray-800 mb-3">填空 (大寫)</h2>
          <div class="bg-gradient-to-r from-blue-50 to-cyan-50 border-2 border-blue-200 rounded-xl p-6">
            <div class="text-sm text-gray-600 mb-3">新台幣 (大寫)</div>
            <div class="flex flex-wrap items-center justify-center gap-1 text-lg relative">
              <template v-for="(digit, index) in fillBlankDigits" :key="index">
                <span 
                  :class="[
                    'inline-flex items-center justify-center min-w-[2rem] h-10 border-2 rounded px-2 font-bold relative',
                    digit.isStrikethrough 
                      ? 'border-red-400 bg-red-50' 
                      : 'bg-white border-blue-300 text-blue-900'
                  ]"
                >
                  <span v-if="!digit.isStrikethrough">{{ digit.value }}</span>
                  <span v-if="digit.isStrikethrough" class="absolute inset-0 flex items-center justify-center">
                    <span class="block w-full h-1 bg-red-500"></span>
                  </span>
                </span>
                <span :class="['font-medium', digit.isStrikethrough ? 'text-red-400' : 'text-gray-700']">{{ digit.unit }}</span>
              </template>
            </div>
          </div>
        </div>

        <!-- Output: 表格 (大寫) -->
        <div class="mb-8">
          <h2 class="text-xl font-bold text-gray-800 mb-3">新台幣 (大寫)</h2>
          <div class="overflow-x-auto rounded-xl border-2 border-gray-200">
            <table class="w-full text-center border-collapse">
              <thead>
                <tr class="bg-gradient-to-r from-purple-100 to-pink-100">
                  <th class="px-4 py-3 font-bold text-gray-700 border-r-2 border-white">金額</th>
                  <th v-for="(unit, index) in tableUnits" :key="index" 
                      class="px-2 py-3 font-medium text-gray-700 border-r-2 border-white last:border-r-0">
                    <div>{{ unit.main }}</div>
                    <div class="text-xs" v-if="unit.sub">{{ unit.sub }}</div>
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr class="bg-white">
                  <td class="px-4 py-4 font-medium text-gray-700 border-r-2 border-gray-200">
                    <div>新台幣</div>
                    <div class="text-sm">(大寫)</div>
                  </td>
                  <td v-for="(digit, index) in tableDigits" :key="index"
                      :class="[
                        'px-2 py-4 text-xl font-black border-r-2 border-gray-200 last:border-r-0 relative',
                        digit.isStrikethrough ? 'bg-red-50' : ''
                      ]"
                  >
                    <span v-if="!digit.isStrikethrough" class="text-purple-700">{{ digit.value }}</span>
                    <span v-if="digit.isStrikethrough" class="absolute inset-0 flex items-center justify-center pointer-events-none">
                      <span class="block w-full h-1 bg-red-500"></span>
                    </span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- Output: 表格 (小寫) -->
        <div>
          <h2 class="text-xl font-bold text-gray-800 mb-3">新台幣 (小寫)</h2>
          <div class="overflow-x-auto rounded-xl border-2 border-gray-200">
            <table class="w-full text-center border-collapse">
              <thead>
                <tr class="bg-gradient-to-r from-gray-100 to-gray-200">
                  <th class="px-4 py-3 font-bold text-gray-700 border-r-2 border-white">金額</th>
                  <th v-for="(unit, index) in tableUnits" :key="index" 
                      class="px-2 py-3 font-medium text-gray-700 border-r-2 border-white last:border-r-0">
                    <div>{{ unit.main }}</div>
                    <div class="text-xs" v-if="unit.sub">{{ unit.sub }}</div>
                  </th>
                </tr>
              </thead>
              <tbody>
                <tr class="bg-white">
                  <td class="px-4 py-4 font-medium text-gray-700 border-r-2 border-gray-200">
                    <div>新台幣</div>
                    <div class="text-sm">(小寫)</div>
                  </td>
                  <td v-for="(digit, index) in normalDigits" :key="index"
                      :class="[
                        'px-2 py-4 text-xl font-black border-r-2 border-gray-200 last:border-r-0 relative',
                        digit.isStrikethrough ? 'bg-red-50' : ''
                      ]"
                  >
                    <span v-if="!digit.isStrikethrough" class="text-gray-700">{{ digit.value }}</span>
                    <span v-if="digit.isStrikethrough" class="absolute inset-0 flex items-center justify-center pointer-events-none">
                      <span class="block w-full h-1 bg-red-500"></span>
                    </span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const amount = ref(0)
const chineseTextRef = ref(null)

const BIGS = ['零', '壹', '貳', '參', '肆', '伍', '陸', '柒', '捌', '玖']
const UNITS = ['', '拾', '佰', '仟', '萬', '拾', '佰', '仟', '億', '拾', '佰']

// 點擊全選中文文字
const selectChineseText = () => {
  if (chineseTextRef.value) {
    const range = document.createRange()
    range.selectNodeContents(chineseTextRef.value)
    const selection = window.getSelection()
    selection.removeAllRanges()
    selection.addRange(range)
  }
}

// 數字轉國字大寫完整描述
const chineseAmount = computed(() => {
  const num = Math.floor(Number(amount.value) || 0)
  
  if (num === 0) return '零元整'
  if (num > 99999999999) return '金額超過上限（最高佰億）'
  
  const str = num.toString().padStart(11, '0')
  let result = []
  let hasNonZero = false
  let prevZero = false
  
  for (let i = 0; i < str.length; i++) {
    const digit = parseInt(str[i])
    const unitIndex = str.length - 1 - i
    
    if (digit === 0) {
      // 萬位和億位的零要特別處理
      if (unitIndex === 4 && hasNonZero) {
        result.push('萬')
        prevZero = false
      } else if (unitIndex === 8 && hasNonZero) {
        result.push('億')
        prevZero = false
      } else {
        prevZero = true
      }
    } else {
      if (prevZero && hasNonZero) {
        result.push('零')
      }
      result.push(BIGS[digit])
      if (unitIndex > 0) {
        result.push(UNITS[unitIndex])
      }
      hasNonZero = true
      prevZero = false
    }
  }
  
  return result.join('') + '元整'
})

// 填空顯示用的數據（由左到右：大單位到小單位）
const fillBlankDigits = computed(() => {
  const num = Math.floor(Number(amount.value) || 0)
  if (num > 99999999999) return []
  
  const str = num.toString().padStart(11, '0')
  const units = ['佰億', '拾億', '億', '仟萬', '佰萬', '拾萬', '萬', '仟', '佰', '拾', '元']
  const result = []
  
  // 找出第一個非零的位置
  let firstNonZeroIndex = -1
  for (let i = 0; i < str.length; i++) {
    if (str[i] !== '0') {
      firstNonZeroIndex = i
      break
    }
  }
  
  // 如果全為0，firstNonZeroIndex為-1，特殊處理
  if (firstNonZeroIndex === -1) {
    firstNonZeroIndex = str.length - 1 // 只顯示元位的零
  }
  
  for (let i = 0; i < str.length; i++) {
    const digit = parseInt(str[i])
    const isStrikethrough = i < firstNonZeroIndex
    
    // 跳過非元位的零（元位的零要顯示）
    if (digit === 0 && i !== str.length - 1) continue
    
    result.push({
      value: BIGS[digit],
      unit: units[i],
      isStrikethrough
    })
  }
  
  return result
})

// 表格單位標題
const tableUnits = computed(() => {
  return [
    { main: '佰', sub: '億' },
    { main: '拾', sub: '億' },
    { main: '億', sub: '' },
    { main: '仟', sub: '萬' },
    { main: '佰', sub: '萬' },
    { main: '拾', sub: '萬' },
    { main: '萬', sub: '' },
    { main: '仟', sub: '' },
    { main: '佰', sub: '' },
    { main: '拾', sub: '' },
    { main: '元', sub: '' }
  ]
})

// 表格大寫數字（帶刪節線）
const tableDigits = computed(() => {
  const num = Math.floor(Number(amount.value) || 0)
  if (num > 99999999999) return Array(11).fill({ value: '', isStrikethrough: false })
  
  const str = num.toString().padStart(11, '0')
  
  // 找出第一個非零的位置
  let firstNonZeroIndex = -1
  for (let i = 0; i < str.length; i++) {
    if (str[i] !== '0') {
      firstNonZeroIndex = i
      break
    }
  }
  
  // 如果全為0，不刪節任何位置
  if (firstNonZeroIndex === -1) {
    firstNonZeroIndex = 0
  }
  
  return str.split('').map((d, index) => ({
    value: BIGS[parseInt(d)],
    isStrikethrough: index < firstNonZeroIndex
  }))
})

// 表格小寫數字（帶刪節線）
const normalDigits = computed(() => {
  const num = Math.floor(Number(amount.value) || 0)
  if (num > 99999999999) return Array(11).fill({ value: '', isStrikethrough: false })
  
  const str = num.toString().padStart(11, '0')
  
  // 找出第一個非零的位置
  let firstNonZeroIndex = -1
  for (let i = 0; i < str.length; i++) {
    if (str[i] !== '0') {
      firstNonZeroIndex = i
      break
    }
  }
  
  // 如果全為0，不刪節任何位置
  if (firstNonZeroIndex === -1) {
    firstNonZeroIndex = 0
  }
  
  return str.split('').map((d, index) => ({
    value: d,
    isStrikethrough: index < firstNonZeroIndex
  }))
})
</script>
