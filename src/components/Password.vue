<script setup>
const count = ref(0)
const password = ref('')
const copied = ref(false)
const settings = ref({
  minLength: 8,
  maxLength: 64,
  maxDigits: 10,
  maxSymbols: 10,
  maxUppers: 10,
  length: 16,
  digits: 2,
  symbols: 2,
  uppers: 4,
  ambiguous: true,
})

const lengthSlidePosition = computed(() => ((settings.value.length - settings.value.minLength) / (settings.value.maxLength - settings.value.minLength)) * 100)
const digitsSlidePosition = computed(() => (settings.value.digits / settings.value.maxDigits) * 100)
const symbolsSlidePosition = computed(() => (settings.value.symbols / settings.value.maxSymbols) * 100)
const uppersSlidePosition = computed(() => (settings.value.uppers / settings.value.maxUppers) * 100)

const lettersSetArray = 'abcdefghijklmnopqrstuvwxyz'.split('')
const uppersSetArray = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.split('')
const symbolsSetArray = '=+-^?!%&*$#@|'.split('')

const strength = computed(() => {
  const count = {
    excess: 0,
    upperCase: 0,
    numbers: 0,
    symbols: 0,
  }
  const weight = {
    excess: 3,
    upperCase: 4,
    numbers: 5,
    symbols: 5,
    combo: 0,
    flatLower: 0,
    flatNumber: 0,
  }
  const strength = {
    text: '',
    score: 0,
  }
  const baseScore = 30

  let i
  for (i = 0; i < password.value.length; i++) {
    if (password.value.charAt(i).match(/[A-Z]/g))
      count.upperCase++

    if (password.value.charAt(i).match(/[0-9]/g))
      count.numbers++

    if (password.value.charAt(i).match(/(.*[!,@,#,$,%,^,&,*,?,_,~])/))
      count.symbols++
  }

  count.excess = password.value.length - 6

  if (count.upperCase && count.numbers && count.symbols)
    weight.combo = 25
  else if ((count.upperCase && count.numbers) || (count.upperCase && count.symbols) || (count.numbers && count.symbols))
    weight.combo = 15

  if (password.value.match(/^[\sa-z]+$/))
    weight.flatLower = -30

  if (password.value.match(/^[\s0-9]+$/))
    weight.flatNumber = -50

  const score
    = baseScore
    + (count.excess * weight.excess)
    + (count.upperCase * weight.upperCase)
    + (count.numbers * weight.numbers)
    + (count.symbols * weight.symbols)
    + weight.combo + weight.flatLower
    + weight.flatNumber

  if (score < 30) {
    strength.text = 'weak'
    strength.score = 10
    return strength
  }
  else if (score >= 30 && score < 75) {
    strength.text = 'average'
    strength.score = 40
    return strength
  }
  else if (score >= 75 && score < 150) {
    strength.text = 'strong'
    strength.score = 75
    return strength
  }
  else {
    strength.text = 'secure'
    strength.score = 100
    return strength
  }
})

function copyToClipboard() {
  const copyElement = document.createElement('textarea')
  copyElement.style.opacity = '0'
  copyElement.style.position = 'fixed'
  copyElement.textContent = password.value
  const body = document.getElementsByTagName('body')[0]
  body.appendChild(copyElement)
  copyElement.select()
  document.execCommand('copy')
  body.removeChild(copyElement)
  copied.value = true

  const resetInterval = setInterval(() => {
    copied.value = false
    clearInterval(resetInterval)
  }, 100000)
}

function generatePassword() {
  const passwordArray = []
  const digitsPositionArray = Array.from({ length: settings.value.length }, (_, i) => i)

  digitsPositionArray.forEach((index) => {
    passwordArray[index] = lettersSetArray[Math.floor(Math.random() * lettersSetArray.length)]
  })

  for (let i = 0; i < settings.value.digits; i++) {
    const digit = Math.round(Math.random() * 9)
    const numberIndex = digitsPositionArray.splice(Math.floor(Math.random() * digitsPositionArray.length), 1)[0]
    passwordArray[numberIndex] = digit
  }

  for (let i = 0; i < settings.value.symbols; i++) {
    const symbol = symbolsSetArray[Math.floor(Math.random() * symbolsSetArray.length)]
    const symbolIndex = digitsPositionArray.splice(Math.floor(Math.random() * digitsPositionArray.length), 1)[0]
    passwordArray[symbolIndex] = symbol
  }

  for (let i = 0; i < settings.value.uppers; i++) {
    const upper = uppersSetArray[Math.floor(Math.random() * uppersSetArray.length)]
    const upperIndex = digitsPositionArray.splice(Math.floor(Math.random() * digitsPositionArray.length), 1)[0]
    passwordArray[upperIndex] = upper
  }
  password.value = passwordArray.join('')
}

function incrementCount() {
  count.value++
}

function resetCopiedAndCount() {
  copied.value = false
  count.value = 0
}

watch(settings, (newSettings) => {
  const sum = Number.parseInt(newSettings.digits) + Number.parseInt(newSettings.symbols) + Number.parseInt(newSettings.uppers)
  settings.value.minLength = Math.max(sum, 8)
  if (sum > Number.parseInt(newSettings.length))
    newSettings.length = sum
  generatePassword()
}, { deep: true })

onMounted(generatePassword)
</script>

<template>
  <section class="wrapper">
    <h1>Password Generator</h1>
    <div class="password-box">
      <span id="password" class="password" @click="copyToClipboard(); incrementCount()">{{ password }}</span>
      <span class="regenerate-password" @click="generatePassword(); resetCopiedAndCount()">1</span>
      <span class="copy-password" @click="copyToClipboard(); incrementCount()">2</span>
      <span v-if="copied" class="tooltip">Password copied x{{ count }}</span>
    </div>
    <form @keydown.enter.prevent>
      <div class="field-wrap">
        <label>Strength</label>
        <span class="range-value">{{ strength.text }}</span>
        <div class="range-slider_wrapper slider-strength" :class="strength.text">
          <span class="slider-bar" :style="{ width: `${strength.score}%` }" />
          <input v-model="strength.score" type="range" class="range-slider" min="0" max="100" disabled>
        </div>
      </div>
      <div class="seperator" />

      <div class="field-wrap">
        <label>Length</label>
        <span class="range-value">{{ settings.length }}</span>
        <div class="range-slider_wrapper">
          <span class="slider-bar" :style="{ width: `${lengthSlidePosition}%` }" />
          <input
            v-model="settings.length" type="range" class="range-slider" :min="settings.minLength"
            :max="settings.maxLength"
          >
        </div>
      </div>

      <div class="field-wrap">
        <label>Uppercases</label>
        <span class="range-value">{{ settings.uppers }}</span>
        <div class="range-slider_wrapper">
          <span class="slider-bar" :style="{ width: `${uppersSlidePosition}%` }" />
          <input v-model="settings.uppers" type="range" class="range-slider" min="0" :max="settings.maxUppers">
        </div>
      </div>

      <div class="field-wrap">
        <label>Digits</label>
        <span class="range-value">{{ settings.digits }}</span>
        <div class="range-slider_wrapper">
          <span class="slider-bar" :style="{ width: `${digitsSlidePosition}%` }" />
          <input v-model="settings.digits" type="range" class="range-slider" min="0" :max="settings.maxDigits">
        </div>
      </div>

      <div class="field-wrap">
        <label>Symbols</label>
        <span class="range-value">{{ settings.symbols }}</span>
        <div class="range-slider_wrapper">
          <span class="slider-bar" :style="{ width: `${symbolsSlidePosition}%` }" />
          <input v-model="settings.symbols" type="range" class="range-slider" min="0" :max="settings.maxSymbols">
        </div>
      </div>
    </form>
  </section>
</template>
