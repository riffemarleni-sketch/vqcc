<template>
  <div id="app">
    <div class="sparkle-extra" style="top:22%;left:18%;animation-delay:0.5s">✦</div>
    <div class="sparkle-extra" style="bottom:28%;left:14%;animation-delay:1.3s;font-size:1rem">✦</div>
    <div class="sparkle-extra" style="top:18%;right:15%;animation-delay:0.8s">✦</div>
    <div class="sparkle-extra" style="bottom:20%;right:18%;animation-delay:0.2s;font-size:1rem">✦</div>

    <div class="app">
      <div class="logo-pill">
        <img class="logo" src="/img/vxcc/logo.png" alt="" />
      </div>

      <div class="title">
        <h1>Vòng Quay</h1>
        <h2>May Mắn</h2>
        <p>Quà nào cũng thích</p>
      </div>

      <div class="wheel-wrapper">
        <div class="wheel-ring"></div>
        <canvas ref="wheelCanvas" id="wheel" width="400" height="400"></canvas>

        <div class="loading-overlay" v-if="!imagesLoaded">
          <div class="loading-spinner"></div>
        </div>

        <div class="pointer">
          <svg viewBox="0 0 44 52" fill="none" xmlns="http://www.w3.org/2000/svg">
            <polygon points="22,52 0,8 44,8" fill="#ffd60a" stroke="white" stroke-width="3" stroke-linejoin="round" />
            <polygon points="22,46 4,10 40,10" fill="#ffb700" />
          </svg>
        </div>

        <div class="center-hub">
          <div class="hub-text">concung<br />.com</div>
        </div>
      </div>

      <button class="spin-btn" @click="spin" :disabled="spinning || !imagesLoaded || hasSpun">
        {{
          !imagesLoaded
            ? 'Đang tải...'
            : hasSpun
              ? 'Đã quay rồi'
              : spinning
                ? 'Đang quay...'
                : 'Quay Ngay'
        }}
      </button>
    </div>

    <transition name="fade">
      <div class="overlay" v-if="showResult" @click.self="closeModal">
        <div class="modal">
          <div class="popup-top">
            <div class="popup-top-title">
              <img class="logo-modal" src="/img/vxcc/logopupop.png" alt="" />
            </div>
          </div>

          <div class="popup-body">
            <div class="popup-product-box">
              <img class="modal-img" :src="currentPrize.img" :alt="currentPrize.label" />
            </div>

            <div class="popup-note">
              <p>CHÚC MỪNG BẠN ĐÃ QUAY TRÚNG ƯU ĐÃI ĐẶC BIỆT</p>
              {{ currentPrize.label }}
            </div>

            <div class="popup-instruction">
              VUI LÒNG LIÊN HỆ GIÁM ĐỐC ĐỂ ĐƯỢC HƯỚNG DẪN NHẬN THƯỞNG
            </div>
          </div>
        </div>
      </div>
    </transition>

    <div v-for="c in confetti" :key="c.id" class="confetti-piece" :style="{
      left: c.x + '%',
      top: '-20px',
      background: c.color,
      borderRadius: c.round ? '50%' : '2px',
      animationDuration: c.dur + 's',
      animationDelay: c.delay + 's',
      width: c.size + 'px',
      height: c.size * 1.3 + 'px',
      transform: 'rotate(' + c.rot + 'deg)'
    }"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const wheelCanvas = ref(null)
const spinning = ref(false)
const showResult = ref(false)
const currentPrize = ref({})
const confetti = ref([])
const imagesLoaded = ref(false)
const hasSpun = ref(false)

const STORAGE_KEY = 'vxcc-wheel-has-spun'
const RESULT_KEY = 'vxcc-wheel-result'

const segments = [
  { label: 'Voucher 5.000.000 Vnd', img: '/img/vxcc/voucher.png', color: '#fce4ec', textColor: '#c9184a' },
  { label: 'Voucher 25.000.000 Vnd', img: '/img/vxcc/voucher.png', color: '#fff0f3', textColor: '#e91e63' },
  { label: 'Voucher 15.000.000 Vnd', img: '/img/vxcc/voucher.png', color: '#fce4ec', textColor: '#c9184a' },
  { label: 'Voucher 10.000.000 Vnd', img: '/img/vxcc/voucher.png', color: '#fff0f3', textColor: '#e91e63' },
  { label: 'Voucher 35.000.000 Vnd', img: '/img/vxcc/voucher.png', color: '#fce4ec', textColor: '#c9184a' },
  { label: 'Voucher 45.000.000 Vnd', img: '/img/vxcc/voucher.png', color: '#fff0f3', textColor: '#e91e63' }
]

const FIXED_LABEL = 'Voucher 15.000.000 Vnd'

const n = segments.length
const arcAngle = (2 * Math.PI) / n
let currentRotation = 0
let ctx = null
const loadedImages = []

function preloadImages() {
  const promises = segments.map(
    (seg, i) =>
      new Promise((resolve) => {
        const el = new Image()
        el.crossOrigin = 'anonymous'
        el.onload = () => {
          loadedImages[i] = el
          resolve()
        }
        el.onerror = () => {
          loadedImages[i] = null
          resolve()
        }
        el.src = seg.img
      })
  )
  return Promise.all(promises)
}

function drawWheel(rotation) {
  if (!ctx || !wheelCanvas.value) return

  const canvas = wheelCanvas.value
  const cx = canvas.width / 2
  const cy = canvas.height / 2
  const R = cx - 6

  ctx.clearRect(0, 0, canvas.width, canvas.height)

  ctx.save()
  ctx.beginPath()
  ctx.arc(cx, cy, R + 4, 0, Math.PI * 2)
  ctx.fillStyle = 'white'
  ctx.fill()
  ctx.strokeStyle = '#f8c8d8'
  ctx.lineWidth = 2
  ctx.stroke()
  ctx.restore()

  ctx.save()
  ctx.beginPath()
  ctx.arc(cx, cy, R + 2, 0, Math.PI * 2)
  ctx.strokeStyle = 'rgba(249,168,196,0.6)'
  ctx.lineWidth = 1
  ctx.setLineDash([6, 6])
  ctx.stroke()
  ctx.restore()

  segments.forEach((seg, i) => {
    const startAngle = rotation + i * arcAngle - Math.PI / 2
    const endAngle = startAngle + arcAngle
    const midAngle = startAngle + arcAngle / 2

    ctx.save()
    ctx.beginPath()
    ctx.moveTo(cx, cy)
    ctx.arc(cx, cy, R, startAngle, endAngle)
    ctx.closePath()

    const grad = ctx.createRadialGradient(cx, cy, R * 0.1, cx, cy, R)
    grad.addColorStop(0, '#ffffff')
    grad.addColorStop(1, seg.color)
    ctx.fillStyle = grad
    ctx.fill()
    ctx.strokeStyle = 'rgba(249,168,196,0.5)'
    ctx.lineWidth = 1.5
    ctx.stroke()
    ctx.restore()

    const imgEl = loadedImages[i]
    if (imgEl) {
      const imgR = R * 0.48
      const imgSize = R * 0.28
      const ix = cx + Math.cos(midAngle) * imgR
      const iy = cy + Math.sin(midAngle) * imgR

      ctx.save()
      ctx.beginPath()
      ctx.moveTo(cx, cy)
      ctx.arc(cx, cy, R - 2, startAngle, endAngle)
      ctx.closePath()
      ctx.clip()
      ctx.drawImage(imgEl, ix - imgSize / 2, iy - imgSize / 2, imgSize, imgSize)
      ctx.restore()
    }

    ctx.save()
    ctx.beginPath()
    ctx.arc(cx, cy, R - 3, 0, Math.PI * 2)
    ctx.clip()

    const lines = seg.label.split('\n')
    const fontSize = lines.length > 2 ? 9 : 10
    const lineH = fontSize + 4

    const textR = R * 0.82
    const tx = cx + Math.cos(midAngle) * textR
    const ty = cy + Math.sin(midAngle) * textR

    ctx.translate(tx, ty)
    ctx.rotate(midAngle + Math.PI / 2)

    ctx.font = `bold ${fontSize}px "Nunito", sans-serif`
    ctx.fillStyle = seg.textColor
    ctx.textAlign = 'center'
    ctx.textBaseline = 'middle'

    lines.forEach((line, li) => {
      const yOff = (li - (lines.length - 1) / 2) * lineH
      ctx.fillText(line.toUpperCase(), 0, yOff)
    })

    ctx.restore()
  })

  segments.forEach((_, i) => {
    const angle = rotation + i * arcAngle - Math.PI / 2
    ctx.save()
    ctx.beginPath()
    ctx.moveTo(cx, cy)
    ctx.lineTo(cx + Math.cos(angle) * R, cy + Math.sin(angle) * R)
    ctx.strokeStyle = 'rgba(249,168,196,0.85)'
    ctx.lineWidth = 1.5
    ctx.stroke()
    ctx.restore()
  })
}

function getFixedWinnerIndex() {
  return segments.findIndex((item) => item.label === FIXED_LABEL)
}

function saveSpinResult(prize) {
  localStorage.setItem(STORAGE_KEY, 'true')
  localStorage.setItem(RESULT_KEY, JSON.stringify(prize))
  hasSpun.value = true
}

function getSavedResult() {
  const saved = localStorage.getItem(RESULT_KEY)
  if (!saved) return null

  try {
    return JSON.parse(saved)
  } catch {
    return null
  }
}

function spin() {
  if (spinning.value || !imagesLoaded.value || hasSpun.value) return

  const winnerIndex = getFixedWinnerIndex()

  if (winnerIndex === -1) {
    alert('Không tìm thấy Voucher 15.000.000 Vnd trong segments')
    return
  }

  spinning.value = true

  const targetAngle = -(winnerIndex * arcAngle + arcAngle / 2)
  const fullSpins = 8 * 2 * Math.PI
  const totalRotation =
    currentRotation + fullSpins + (targetAngle - (currentRotation % (2 * Math.PI)))

  const duration = 4500
  const start = performance.now()
  const startRot = currentRotation
  const easeOut = (t) => 1 - Math.pow(1 - t, 4)

  const animate = (now) => {
    const t = Math.min((now - start) / duration, 1)
    const rot = startRot + (totalRotation - startRot) * easeOut(t)
    drawWheel(rot)

    if (t < 1) {
      requestAnimationFrame(animate)
    } else {
      currentRotation = totalRotation
      spinning.value = false
      currentPrize.value = segments[winnerIndex]

      saveSpinResult(segments[winnerIndex])
      launchConfetti()

      setTimeout(() => {
        showResult.value = true
      }, 400)
    }
  }

  requestAnimationFrame(animate)
}

function launchConfetti() {
  const colors = ['#f72585', '#ffd60a', '#4cc9f0', '#7209b7', '#06d6a0', '#ff6b6b', '#ffffff']

  confetti.value = Array.from({ length: 80 }, (_, id) => ({
    id,
    x: Math.random() * 100,
    color: colors[Math.floor(Math.random() * colors.length)],
    dur: 2 + Math.random() * 2,
    delay: Math.random() * 1.2,
    size: 6 + Math.random() * 8,
    rot: Math.random() * 360,
    round: Math.random() > 0.5
  }))

  setTimeout(() => {
    confetti.value = []
  }, 5000)
}

function closeModal() {
  showResult.value = false
}

onMounted(async () => {
  hasSpun.value = localStorage.getItem(STORAGE_KEY) === 'true'

  const savedPrize = getSavedResult()
  if (savedPrize) {
    currentPrize.value = savedPrize
  }

  if (!wheelCanvas.value) return
  ctx = wheelCanvas.value.getContext('2d')

  drawWheel(0)
  await preloadImages()
  imagesLoaded.value = true
  drawWheel(0)
})
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Nunito:wght@700;800;900&family=Baloo+2:wght@700;800&display=swap');

*,
*::before,
*::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  --pink: #f72585;
  --pink-dark: #c9184a;
  --pink-light: #ffccd5;
  --gold: #ffd60a;
}

body {
  font-family: 'Nunito', sans-serif;
  background: radial-gradient(ellipse at 30% 40%, #ff4d9e 0%, #e5006e 40%, #b5004f 100%);
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  position: relative;
}

body::before,
body::after {
  content: '✦';
  position: absolute;
  color: rgba(255, 255, 255, 0.25);
  font-size: 2rem;
  animation: twinkle 2s ease-in-out infinite alternate;
}

body::before {
  top: 15%;
  left: 8%;
}

body::after {
  bottom: 18%;
  right: 8%;
  animation-delay: 1s;
}

@keyframes twinkle {
  from {
    opacity: 0.2;
    transform: scale(0.8) rotate(0deg);
  }

  to {
    opacity: 0.8;
    transform: scale(1.3) rotate(20deg);
  }
}

.logo {
  width: 125px;
}

.sparkle-extra {
  position: absolute;
  color: rgba(255, 214, 10, 0.55);
  font-size: 1.4rem;
  animation: twinkle 2.5s ease-in-out infinite alternate;
  pointer-events: none;
}

.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  position: relative;
  z-index: 1;
  margin-top: 10px;
}

.logo-pill {
  background: white;
  border-radius: 999px;
  padding: 7px 28px;
  font-family: 'Baloo 2', cursive;
  font-size: 1.15rem;
  font-weight: 800;
  color: var(--pink);
  box-shadow: 0 4px 18px rgba(0, 0, 0, 0.18);
  margin-bottom: 14px;
  max-height: 60px;
}

.logo-pill span {
  color: #444;
}

.title {
  text-align: center;
  line-height: 1.4;
  margin-bottom: 6px;
  margin-top: -15px;
}

.title h1 {
  font-family: 'Baloo 2', cursive;
  font-size: 2.8rem;
  font-weight: 900;
  color: white;
  text-transform: uppercase;
  letter-spacing: 1px;
  text-shadow: 0 2px 12px rgba(0, 0, 0, 0.2);
}

.title h2 {
  font-family: 'Baloo 2', cursive;
  font-size: 2.8rem;
  font-weight: 900;
  color: var(--gold);
  text-transform: uppercase;
  letter-spacing: 1px;
  text-shadow: 0 2px 12px rgba(0, 0, 0, 0.2);
  margin-top: -12px;
}

.title p {
  color: white;
  font-size: 0.95rem;
  font-weight: 700;
  letter-spacing: 3px;
  text-transform: uppercase;
  margin-top: 4px;
  opacity: 0.9;
}

.wheel-wrapper {
  position: relative;
  width: 400px;
  height: 400px;
  margin: 18px 0 8px;
  filter: drop-shadow(0 12px 40px rgba(0, 0, 0, 0.35));
}

.wheel-ring {
  position: absolute;
  inset: -8px;
  border-radius: 50%;
  border: 2px dashed rgba(255, 255, 255, 0.4);
  animation: rotate-ring 12s linear infinite;
}

@keyframes rotate-ring {
  to {
    transform: rotate(360deg);
  }
}

canvas#wheel {
  width: 400px;
  height: 400px;
  border-radius: 50%;
  display: block;
}

.loading-overlay {
  position: absolute;
  inset: 0;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.55);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 5;
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 4px solid var(--pink-light);
  border-top-color: var(--pink);
  border-radius: 50%;
  animation: spin-anim 0.8s linear infinite;
}

@keyframes spin-anim {
  to {
    transform: rotate(360deg);
  }
}

.center-hub {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 72px;
  height: 72px;
  border-radius: 50%;
  background: radial-gradient(circle at 35% 35%, #ff6db2, var(--pink) 60%, var(--pink-dark));
  border: 4px solid white;
  box-shadow: 0 0 0 3px var(--pink), 0 4px 20px rgba(0, 0, 0, 0.3);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 10;
  pointer-events: none;
}

.hub-text {
  font-family: 'Baloo 2', cursive;
  font-weight: 900;
  color: white;
  font-size: 0.72rem;
  line-height: 1;
  text-align: center;
  text-shadow: 0 1px 4px rgba(0, 0, 0, 0.3);
}

.pointer {
  position: absolute;
  top: -22px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 20;
  filter: drop-shadow(0 3px 6px rgba(0, 0, 0, 0.35));
}

.pointer svg {
  width: 44px;
  height: 52px;
}

.spin-btn {
  background: white;
  color: var(--pink);
  font-family: 'Baloo 2', cursive;
  font-size: 1.4rem;
  font-weight: 900;
  letter-spacing: 3px;
  text-transform: uppercase;
  border: none;
  border-radius: 999px;
  padding: 16px 80px;
  cursor: pointer;
  box-shadow: 0 6px 28px rgba(0, 0, 0, 0.25);
  transition: transform 0.15s, box-shadow 0.15s;
  position: relative;
  overflow: hidden;
}

.spin-btn::after {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.4) 0%, transparent 60%);
  border-radius: 999px;
  pointer-events: none;
}

.spin-btn:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 10px 32px rgba(0, 0, 0, 0.3);
}

.spin-btn:active:not(:disabled) {
  transform: translateY(1px);
}

.spin-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.28);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 9999;
  padding: 0;
}

.popup-top {
  width: 100%;
  height: 170px;
  background: url("/img/vxcc/banner-popup.png") center/cover no-repeat;
  position: relative;
  border-bottom-left-radius: 28px;
  border-bottom-right-radius: 28px;
  overflow: hidden;
}

.popup-top-title {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  padding: 0;
  margin: 0;
  background: none;
  box-shadow: none;
  border-radius: 0;
}

.logo-modal {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.popup-body {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 18px 20px 28px;
  margin-top: -20px;
  font-family: 'Inter', sans-serif;
}

.popup-product-box {
  width: 134px;
  height: 134px;
  border-radius: 13px;
  background: #f3e8df;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 14px;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.06);
  margin-bottom: 20px;
}

.popup-divider {
  display: none;
}

.popup-note {
  width: 100%;
  text-align: center;
  color: #e3277a;
  font-weight: 900;
  font-size: 24px;
  line-height: 1.25;
  margin-bottom: 24px;
}

.prize-name {
  color: #e3277a;
  font-size: 28px;
  font-weight: 900;
  line-height: 1.2;
  margin-top: 8px;
}

.popup-note br {
  display: none;
}

.popup-note::before {
  content: "Ba mẹ đã nhận được quà";
  display: none;
}

.popup-instruction {
  width: 100%;
  background: #f6efea;
  border: 2px dashed #f2ba63;
  border-radius: 26px;
  padding: 20px 18px;
  text-align: center;
  color: #b67a1e;
  font-size: 16px;
  font-weight: 800;
  line-height: 1.5;
  margin-bottom: 26px;
}

.modal {
  background: white;
  border-radius: 28px;
  text-align: center;
  max-width: 382px;
  width: 90%;
  overflow: hidden;
  animation: popIn 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 24px 80px rgba(0, 0, 0, 0.4);
}

@keyframes popIn {
  from {
    transform: scale(0.6);
    opacity: 0;
  }

  to {
    transform: scale(1);
    opacity: 1;
  }
}

.modal-img {
  width: 110px;
  height: 110px;
  object-fit: contain;
  display: block;
  filter: drop-shadow(0 4px 12px rgba(247, 37, 133, 0.25));
}

.close-btn {
  background: linear-gradient(135deg, var(--pink), var(--pink-dark));
  color: white;
  font-family: 'Baloo 2', cursive;
  font-size: 1.1rem;
  font-weight: 800;
  border: none;
  border-radius: 999px;
  padding: 12px 44px;
  cursor: pointer;
  letter-spacing: 1px;
  transition: transform 0.15s;
}

.close-btn:hover {
  transform: scale(1.05);
}

.confetti-piece {
  position: fixed;
  animation: confettiFall linear forwards;
  z-index: 200;
  pointer-events: none;
}

@keyframes confettiFall {
  0% {
    transform: translateY(-20px) rotate(0deg);
    opacity: 1;
  }

  100% {
    transform: translateY(105vh) rotate(720deg);
    opacity: 0;
  }
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.25s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>