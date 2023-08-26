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
      <span class="regenerate-password" @click="generatePassword(); resetCopiedAndCount()">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path
            fill-rule="evenodd" clip-rule="evenodd"
            d="M21 1C21.5523 1 22 1.44772 22 2V8C22 8.55228 21.5523 9 21 9H15C14.4477 9 14 8.55228 14 8C14 7.44772 14.4477 7 15 7H20V2C20 1.44772 20.4477 1 21 1Z"
            fill="currentColor"
          />
          <path
            fill-rule="evenodd" clip-rule="evenodd"
            d="M13.2337 4.10388C11.7084 3.86582 10.1468 4.07415 8.73734 4.70373C7.32784 5.33331 6.13059 6.35725 5.29 7.65204C4.44942 8.94683 4.00141 10.4572 4 12.0009C3.9995 12.5532 3.55137 13.0005 2.99909 13C2.4468 12.9995 1.9995 12.5514 2 11.9991C2.00176 10.0694 2.56178 8.18149 3.61251 6.563C4.66324 4.94451 6.1598 3.66459 7.92168 2.87762C9.68356 2.09064 11.6355 1.83023 13.5421 2.1278C15.4486 2.42536 17.2284 3.26819 18.6667 4.55463L18.669 4.55669L21.669 7.25669C22.0795 7.62615 22.1128 8.25844 21.7433 8.66895C21.3738 9.07946 20.7415 9.11274 20.331 8.74328L17.3333 6.04534C17.333 6.04501 17.3326 6.04469 17.3322 6.04436C16.1818 5.01576 14.7584 4.34185 13.2337 4.10388Z"
            fill="currentColor"
          />
          <path
            fill-rule="evenodd" clip-rule="evenodd"
            d="M2 16C2 15.4477 2.44772 15 3 15H9C9.55228 15 10 15.4477 10 16C10 16.5523 9.55228 17 9 17H4V22C4 22.5523 3.55228 23 3 23C2.44772 23 2 22.5523 2 22V16Z"
            fill="currentColor"
          />
          <path
            fill-rule="evenodd" clip-rule="evenodd"
            d="M21.0009 11C21.5532 11.0005 22.0005 11.4486 22 12.0009C21.9982 13.9306 21.4382 15.8185 20.3875 17.437C19.3368 19.0555 17.8402 20.3354 16.0783 21.1224C14.3164 21.9093 12.3645 22.1698 10.4579 21.8722C8.55136 21.5746 6.7716 20.7318 5.33333 19.4454L5.33103 19.4433L5.33103 19.4433L2.33103 16.7433C1.92052 16.3738 1.88724 15.7415 2.2567 15.331C2.62616 14.9205 3.25845 14.8872 3.66896 15.2567L6.66666 17.9546C6.66704 17.955 6.66742 17.9553 6.66779 17.9557C7.81822 18.9842 9.24159 19.6581 10.7663 19.8961C12.2916 20.1342 13.8531 19.9258 15.2627 19.2963C16.6722 18.6667 17.8694 17.6427 18.71 16.3479C19.5506 15.0532 19.9986 13.5428 20 11.9991C20.0005 11.4468 20.4486 10.9995 21.0009 11Z"
            fill="currentColor"
          />
        </svg>
      </span>
      <span class="copy-password" @click="copyToClipboard(); incrementCount()">
        <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
          <path
            fill-rule="evenodd" clip-rule="evenodd"
            d="M3.87868 3.87868C4.44129 3.31607 5.20435 3 6 3H8C8.55228 3 9 3.44772 9 4C9 4.55228 8.55228 5 8 5H6C5.73478 5 5.48043 5.10536 5.29289 5.29289C5.10536 5.48043 5 5.73478 5 6V20C5 20.2652 5.10536 20.5196 5.29289 20.7071C5.48043 20.8946 5.73478 21 6 21H18C18.2652 21 18.5196 20.8946 18.7071 20.7071C18.8946 20.5196 19 20.2652 19 20V18C19 17.4477 19.4477 17 20 17C20.5523 17 21 17.4477 21 18V20C21 20.7957 20.6839 21.5587 20.1213 22.1213C19.5587 22.6839 18.7957 23 18 23H6C5.20435 23 4.44129 22.6839 3.87868 22.1213C3.31607 21.5587 3 20.7957 3 20V6C3 5.20435 3.31607 4.44129 3.87868 3.87868ZM15.1183 3.52814C15.2923 3.20298 15.6312 3 16 3H18C18.7956 3 19.5587 3.31607 20.1213 3.87868C20.6839 4.44129 21 5.20435 21 6V10C21 10.4407 20.7115 10.8294 20.2898 10.9571C19.868 11.0848 19.4124 10.9214 19.1679 10.5547L15.1679 4.5547C14.9634 4.24784 14.9443 3.8533 15.1183 3.52814ZM17.8685 5L19 6.69722V6C19 5.73478 18.8946 5.48043 18.7071 5.29289C18.5196 5.10536 18.2652 5 18 5H17.8685Z"
            fill="currentColor"
          />
          <path
            fill-rule="evenodd" clip-rule="evenodd"
            d="M7 3C7 1.89543 7.89543 1 9 1H15C16.1046 1 17 1.89543 17 3V5C17 6.10457 16.1046 7 15 7H9C7.89543 7 7 6.10457 7 5V3ZM15 3H9V5H15V3Z"
            fill="currentColor"
          />
          <path
            fill-rule="evenodd" clip-rule="evenodd"
            d="M10 14C10 13.4477 10.4477 13 11 13H21C21.5523 13 22 13.4477 22 14C22 14.5523 21.5523 15 21 15H11C10.4477 15 10 14.5523 10 14Z"
            fill="currentColor"
          />
          <path
            fill-rule="evenodd" clip-rule="evenodd"
            d="M15.7071 9.29289C16.0976 9.68342 16.0976 10.3166 15.7071 10.7071L12.4142 14L15.7071 17.2929C16.0976 17.6834 16.0976 18.3166 15.7071 18.7071C15.3166 19.0976 14.6834 19.0976 14.2929 18.7071L10.2929 14.7071C9.90237 14.3166 9.90237 13.6834 10.2929 13.2929L14.2929 9.29289C14.6834 8.90237 15.3166 8.90237 15.7071 9.29289Z"
            fill="currentColor"
          />
        </svg>
      </span>
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
