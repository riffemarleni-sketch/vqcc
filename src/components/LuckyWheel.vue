<template>
  <div class="wheel-page">
    <div class="bg-dots"></div>

    <div class="page-decor decor-1">✦</div>
    <div class="page-decor decor-2">✦</div>

    <div class="brand-pill">concung.com</div>

    <div class="title-wrap">
      <h1>VÒNG QUAY</h1>
      <h2>MAY MẮN</h2>
      <p>QUÀ NÀO CŨNG THÍCH</p>
    </div>

    <div class="wheel-wrapper">
      <div class="pointer"></div>

      <div
        class="wheel-outer"
        :style="{
          transform: `rotate(${rotation}deg)`,
          transition: spinning
            ? 'transform 5s cubic-bezier(0.17, 0.67, 0.15, 1)'
            : 'none'
        }"
      >
        <div class="wheel">
          <div
            v-for="(item, index) in prizes"
            :key="index"
            class="slice"
            :style="getSliceStyle(index)"
          >
            <div
              class="slice-inner"
              :style="getSliceContentStyle(index)"
            >
              <img
                v-if="item.image"
                :src="item.image"
                :alt="item.label"
                class="slice-image"
              />
              <div v-else class="slice-icon">{{ item.icon }}</div>

              <div class="slice-label">
                {{ item.label }}
              </div>
            </div>
          </div>

          <div class="center-circle">
            <div class="center-inner">
              concung
              <span>.com</span>
            </div>
          </div>
        </div>
      </div>
    </div>

    <button class="spin-btn" @click="spinWheel" :disabled="spinning">
      {{ spinning ? 'ĐANG QUAY...' : 'QUAY NGAY' }}
    </button>

    <transition name="popup-fade">
      <div v-if="showPopup" class="popup-overlay" @click.self="closePopup">
        <div class="popup-box">
          <div class="popup-top">🎉 CHÚC MỪNG 🎉</div>

          <div class="popup-content">
            <div class="popup-image" v-if="result?.image">
              <img :src="result.image" :alt="result.label" />
            </div>

            <div class="popup-icon" v-else>
              {{ result?.icon || '🎁' }}
            </div>

            <div class="popup-title">Chúc mừng bạn đã nhận được</div>
            <div class="popup-prize">{{ result?.label }}</div>
            <p class="popup-note">
              Vui lòng chụp màn hình và gửi cho quản lý để nhận quà.
            </p>

            <button class="popup-btn" @click="closePopup">Đóng</button>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const prizes = ref([
  { label: 'Xe ô tô điện', icon: '🚗', image: '' },
  { label: 'Nồi chiên không dầu', icon: '🍳', image: '' },
  { label: 'Chúc bạn may mắn lần sau', icon: '🎉', image: '' },
  { label: 'Xe đạp thể thao Giant', icon: '🚲', image: '' },
  { label: 'Máy xay đa năng Nakawa', icon: '🧃', image: '' },
  { label: 'Máy hút bụi cầm tay', icon: '🧹', image: '' }
])

const spinning = ref(false)
const showPopup = ref(false)
const result = ref(null)
const rotation = ref(0)

const total = prizes.value.length
const sliceDeg = 360 / total

function getSliceStyle(index) {
  return {
    transform: `rotate(${index * sliceDeg}deg) skewY(${90 - sliceDeg}deg)`
  }
}

function getSliceContentStyle(index) {
  return {
    transform: `skewY(-${90 - sliceDeg}deg) rotate(${sliceDeg / 2}deg) translate(0, -78%)`
  }
}

function spinWheel() {
  if (spinning.value) return

  spinning.value = true
  showPopup.value = false
  result.value = null

  const randomIndex = Math.floor(Math.random() * prizes.value.length)

  const finalDeg = 360 - (randomIndex * sliceDeg + sliceDeg / 2)
  const extraRounds = 360 * 6

  rotation.value += extraRounds + finalDeg - (rotation.value % 360)

  setTimeout(() => {
    result.value = prizes.value[randomIndex]
    spinning.value = false
    showPopup.value = true
  }, 5200)
}

function closePopup() {
  showPopup.value = false
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.wheel-page {
  position: relative;
  width: 100%;
  min-height: 100vh;
  overflow: hidden;
  background: linear-gradient(180deg, #ff1983 0%, #ff0f79 45%, #ff2a8b 100%);
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  padding: 28px 16px 36px;
}

.bg-dots {
  position: absolute;
  inset: 0;
  background-image: radial-gradient(rgba(255,255,255,0.12) 1px, transparent 1px);
  background-size: 22px 22px;
  pointer-events: none;
}

.page-decor {
  position: absolute;
  z-index: 2;
  color: #ffd84c;
  font-size: 34px;
  animation: twinkle 2.4s infinite ease-in-out;
}

.decor-1 {
  left: calc(50% - 210px);
  top: 280px;
}

.decor-2 {
  right: calc(50% - 220px);
  bottom: 110px;
}

@keyframes twinkle {
  0%, 100% {
    transform: scale(1) rotate(0deg);
    opacity: .8;
  }
  50% {
    transform: scale(1.15) rotate(10deg);
    opacity: 1;
  }
}

.brand-pill {
  position: relative;
  z-index: 2;
  background: #fff;
  color: #df3b86;
  font-weight: 900;
  font-size: 30px;
  line-height: 1;
  padding: 16px 42px;
  border-radius: 999px;
  margin-top: 4px;
  box-shadow: 0 10px 24px rgba(0,0,0,.08);
}

.title-wrap {
  position: relative;
  z-index: 2;
  margin-top: 24px;
  text-align: center;
}

.title-wrap h1,
.title-wrap h2,
.title-wrap p {
  margin: 0;
  line-height: 1;
}

.title-wrap h1 {
  font-size: 84px;
  font-weight: 1000;
  color: #fff;
  text-shadow: 0 6px 0 rgba(0,0,0,.08);
}

.title-wrap h2 {
  margin-top: 8px;
  font-size: 78px;
  font-weight: 1000;
  color: #ffd93d;
  text-shadow: 0 6px 0 rgba(0,0,0,.08);
}

.title-wrap p {
  display: inline-block;
  margin-top: 12px;
  padding: 10px 22px;
  background: rgba(255,255,255,.16);
  border-radius: 999px;
  font-size: 26px;
  font-weight: 900;
  color: #fff6b7;
}

.wheel-wrapper {
  position: relative;
  z-index: 2;
  margin-top: 18px;
  width: min(90vw, 720px);
  height: min(90vw, 720px);
  display: flex;
  justify-content: center;
  align-items: center;
}

.pointer {
  position: absolute;
  top: -6px;
  left: 50%;
  transform: translateX(-50%);
  width: 0;
  height: 0;
  border-left: 34px solid transparent;
  border-right: 34px solid transparent;
  border-top: 76px solid #ffd400;
  z-index: 20;
  filter: drop-shadow(0 4px 8px rgba(0,0,0,.16));
}

.wheel-outer {
  width: 100%;
  height: 100%;
  border-radius: 50%;
  padding: 14px;
  background: linear-gradient(180deg, #ffc5dc, #ff9fc7);
  box-shadow: 0 20px 45px rgba(0,0,0,.16);
}

.wheel {
  position: relative;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  overflow: hidden;
  background: #fff2f8;
  border: 14px solid #ffddeb;
}

.slice {
  position: absolute;
  width: 50%;
  height: 50%;
  top: 0;
  left: 50%;
  transform-origin: 0% 100%;
  overflow: hidden;
}

.slice:nth-child(odd) {
  background: #fdf6fa;
}

.slice:nth-child(even) {
  background: #f7f0f5;
}

.slice-inner {
  position: absolute;
  left: -100%;
  width: 200%;
  height: 200%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  padding-top: 48px;
  text-align: center;
}

.slice-image {
  width: 72px;
  height: 72px;
  object-fit: contain;
  margin-bottom: 10px;
}

.slice-icon {
  font-size: 58px;
  line-height: 1;
  margin-bottom: 10px;
}

.slice-label {
  width: 140px;
  font-size: 18px;
  line-height: 1.15;
  font-weight: 900;
  text-transform: uppercase;
  color: #ef2a7e;
  word-break: break-word;
}

.center-circle {
  position: absolute;
  inset: 50%;
  width: 132px;
  height: 132px;
  transform: translate(-50%, -50%);
  border-radius: 50%;
  background: linear-gradient(180deg, #ff81b6, #ef287f);
  border: 7px solid #ffd4e6;
  z-index: 30;
  display: flex;
  align-items: center;
  justify-content: center;
}

.center-inner {
  width: 98px;
  height: 98px;
  border-radius: 50%;
  background: #ef2a7f;
  border: 4px solid #fff5fa;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  color: #fff;
  font-weight: 900;
  font-size: 18px;
  line-height: 1;
  text-align: center;
}

.center-inner span {
  display: block;
  margin-top: 4px;
}

.spin-btn {
  position: relative;
  z-index: 2;
  margin-top: 24px;
  width: min(86vw, 520px);
  height: 86px;
  border: none;
  border-radius: 24px;
  background: #fff;
  color: #df2f80;
  font-size: 36px;
  font-weight: 1000;
  cursor: pointer;
  box-shadow: 0 16px 30px rgba(0,0,0,.12);
}

.spin-btn:disabled {
  opacity: .75;
  cursor: not-allowed;
}

/* popup */
.popup-overlay {
  position: fixed;
  inset: 0;
  z-index: 999;
  background: rgba(0,0,0,.45);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 16px;
}

.popup-box {
  width: 100%;
  max-width: 430px;
  background: #fff;
  border-radius: 24px;
  overflow: hidden;
  box-shadow: 0 20px 50px rgba(0,0,0,.25);
}

.popup-top {
  background: linear-gradient(180deg, #ff4f99, #f11f7b);
  color: #fff;
  text-align: center;
  padding: 18px;
  font-size: 28px;
  font-weight: 1000;
}

.popup-content {
  padding: 24px;
  text-align: center;
}

.popup-image img {
  width: 120px;
  height: 120px;
  object-fit: contain;
  margin-bottom: 12px;
}

.popup-icon {
  font-size: 70px;
  margin-bottom: 12px;
}

.popup-title {
  font-size: 24px;
  font-weight: 800;
  color: #333;
}

.popup-prize {
  margin-top: 10px;
  font-size: 28px;
  font-weight: 1000;
  color: #e53185;
  line-height: 1.2;
}

.popup-note {
  margin-top: 14px;
  font-size: 14px;
  line-height: 1.5;
  color: #666;
}

.popup-btn {
  margin-top: 20px;
  width: 100%;
  height: 52px;
  border: none;
  border-radius: 14px;
  background: linear-gradient(180deg, #ff4f99, #f11f7b);
  color: #fff;
  font-size: 18px;
  font-weight: 900;
  cursor: pointer;
}

.popup-fade-enter-active,
.popup-fade-leave-active {
  transition: opacity .25s ease;
}

.popup-fade-enter-from,
.popup-fade-leave-to {
  opacity: 0;
}

@media (max-width: 1024px) {
  .title-wrap h1 {
    font-size: 64px;
  }

  .title-wrap h2 {
    font-size: 60px;
  }

  .title-wrap p {
    font-size: 20px;
  }

  .wheel-wrapper {
    width: min(92vw, 620px);
    height: min(92vw, 620px);
  }

  .slice-icon {
    font-size: 48px;
  }

  .slice-image {
    width: 62px;
    height: 62px;
  }

  .slice-label {
    width: 120px;
    font-size: 15px;
  }

  .spin-btn {
    height: 76px;
    font-size: 30px;
  }
}

@media (max-width: 768px) {
  .wheel-page {
    padding-top: 20px;
    padding-bottom: 28px;
  }

  .brand-pill {
    font-size: 20px;
    padding: 13px 28px;
  }

  .title-wrap {
    margin-top: 18px;
  }

  .title-wrap h1 {
    font-size: 42px;
  }

  .title-wrap h2 {
    font-size: 42px;
    margin-top: 6px;
  }

  .title-wrap p {
    font-size: 14px;
    padding: 8px 14px;
    margin-top: 10px;
  }

  .wheel-wrapper {
    margin-top: 16px;
    width: min(94vw, 420px);
    height: min(94vw, 420px);
  }

  .pointer {
    border-left: 22px solid transparent;
    border-right: 22px solid transparent;
    border-top: 52px solid #ffd400;
  }

  .wheel {
    border-width: 10px;
  }

  .slice-inner {
    padding-top: 28px;
  }

  .slice-image {
    width: 42px;
    height: 42px;
    margin-bottom: 6px;
  }

  .slice-icon {
    font-size: 34px;
    margin-bottom: 6px;
  }

  .slice-label {
    width: 82px;
    font-size: 10px;
    line-height: 1.15;
  }

  .center-circle {
    width: 92px;
    height: 92px;
  }

  .center-inner {
    width: 68px;
    height: 68px;
    font-size: 13px;
  }

  .spin-btn {
    width: min(92vw, 340px);
    height: 64px;
    font-size: 24px;
    border-radius: 18px;
    margin-top: 18px;
  }

  .decor-1 {
    left: 24px;
    top: 230px;
  }

  .decor-2 {
    right: 24px;
    bottom: 120px;
  }
}
</style>