<template>
  <div id="app" :class="{ dark: darkMode }">
    <!-- Background Coin Rain -->
    <div v-if="rainEnabled" class="coin-rain">
      <div
        v-for="(coin, index) in rainCoins"
        :key="index"
        class="rain-coin"
        :style="{
          left: coin.left,
          width: coin.size,
          height: coin.size,
          animationDelay: coin.delay,
          animationDuration: coin.duration,
          backgroundColor: hexToRgba(coinColor, 0.2)
        }"
      ></div>
    </div>

    <!-- Sidebar -->
    <div class="sidebar" :class="{ 'sidebar-hidden': !sidebarVisible, dark: darkMode }">
      <h2>Coin Settings</h2>
      <label for="color">Coin Color Customization:</label>
      <input type="color" id="color" v-model="coinColor" />

      <p class="magnification-tip">Magnify page to ~150% for best experience</p>
      <p class="coin-tip">The coin can be clicked to flip!</p>

      <h3>Extra Settings</h3>
      <div class="setting">
        <label for="flipSpeed">Flip Speed (seconds): {{ flipDuration }}</label>
        <input type="range" id="flipSpeed" v-model.number="flipDuration" min=".75" max="2.5" step="0.05" />
      </div>
      <div class="setting">
        <label for="coinSize">Coin Size (px): {{ coinSize }}</label>
        <input type="range" id="coinSize" v-model.number="coinSize" min="250" max="400" step="5" />
      </div>
      <div class="setting">
        <label>
          <input type="checkbox" v-model="rainEnabled" /> Enable Coin Rain
        </label>
      </div>
      <div class="setting">
        <label>
          <input type="checkbox" v-model="showHistory" /> Show Flip History
        </label>
      </div>
      <div class="setting">
        <label>
          <input type="checkbox" v-model="darkMode" /> Dark Mode
        </label>
      </div>
      <div class="setting">
        <label>
          <input type="checkbox" v-model="showTitle" /> Show Title Text
        </label>
      </div>
      <div class="setting">
        <label>
          <input type="checkbox" v-model="showButton" /> Show Flip Button
        </label>
      </div>
      <div class="setting coin-fact">
        <h4>Coin Fact</h4>
        <p>{{ coinFact }}</p>
        <button @click="generateCoinFact">New Fact</button>
      </div>
      <!-- Reset button at the bottom -->
      <div class="setting reset-settings">
        <button @click="resetSettings">Reset Settings</button>
      </div>
    </div>

    <!-- Flip History Overlay (positioned in the upper left) -->
    <div
      v-if="showHistory"
      class="flip-history-overlay"
      :style="{ left: sidebarVisible ? '350px' : '10px' }"
    >
      <h4>Flip History (Last 15):</h4>
      <ul>
        <li v-for="(flip, index) in flipHistory.slice(-15).reverse()" :key="index">
          {{ flip }}
        </li>
      </ul>
    </div>

    <!-- Main Content -->
    <div class="main-content" :class="{ 'main-shifted': sidebarVisible }">
      <div class="coin-container" :style="{ marginBottom: coinSize / 5 + 'px'}">
        <h2 v-if="showTitle">Coin Flipper!</h2>
        <div
          class="coin"
          :style="{
            backgroundColor: coinColor,
            width: coinSize + 'px',
            height: coinSize + 'px',
            '--flip-duration': flipDuration + 's',
            '--initial-rotation': isFlipping
              ? (previousResult === 'tails' ? '180deg' : '0deg')
              : (coinResult === 'tails' ? '180deg' : '0deg'),
            '--rotation-delta': isFlipping
              ? (720 + ((coinResult === 'tails' ? 180 : 0) - (previousResult === 'tails' ? 180 : 0))) + 'deg'
              : '0deg',
            transform: !isFlipping
              ? (coinResult === 'tails' ? 'rotateX(180deg)' : 'rotateX(0deg)')
              : ''
          }"
          :class="{ flipping: isFlipping }"
          @animationend="onAnimationEnd"
          @click="flipCoin"
        >
          <div class="face front" :style="{ fontSize: (coinSize / 4) + 'px' }">Heads</div>
          <div class="face back" :style="{ fontSize: (coinSize / 4) + 'px' }">Tails</div>
        </div>
      </div>
      <button v-if="showButton" @click="flipCoin" :disabled="isFlipping">Flip Coin</button>
    </div>

    <!-- Arrow toggle button -->
    <button
      class="arrow-toggle"
      @click="toggleSidebar"
      :style="{ left: sidebarVisible ? '300px' : '0px' }"
    >
      {{ sidebarVisible ? '◀' : '▶' }}
    </button>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref } from 'vue';

export default defineComponent({
  name: 'CoinFlipper',
  setup() {
    const coinColor = ref('#FFD700');
    const showTitle = ref(true);
    const showButton = ref(true);
    const isFlipping = ref(false);
    const coinResult = ref<string | null>('heads');
    // This holds the coin’s face before a flip so the animation can start from its current rotation.
    const previousResult = ref<string>('heads');
    const sidebarVisible = ref(true);
    const flipDuration = ref(1);
    const coinSize = ref(250);

    const coinFact = ref("Did you know? The US quarter has featured over 119 different designs since 1932.");
    const coinFacts = [
      "The US quarter has featured over 119 different designs since 1932.",
      "The U.S. penny features Abraham Lincoln, the only U.S. president on a coin who was assassinated.",
      "The U.K. has a 'Two Pound' coin that is bimetallic.",
      "Ancient coins were sometimes made from precious stones instead of metal.",
      "The largest coin ever minted was the Australian 50 Cent coin, weighing over a kilogram.",
      "Coins have been used as a form of currency for over 2,500 years.",
      "The first coins were made from electrum, a natural alloy of gold and silver.",
      "The study of coins is known as numismatics.",
      "The U.S. Mint uses special techniques to mint coins with intricate details.",
      "Some coins feature microprinting as a security measure against counterfeiting.",
      "The oldest known coin dates back to 600 BC from the ancient kingdom of Lydia.",
      "Over 50 countries currently mint their own coins.",
      "Modern coins often contain a blend of metals to produce unique hues.",
      "The U.S. penny was originally 95% copper but is now mostly zinc with a copper coating.",
      "Coin collecting is a hobby enjoyed by millions around the world.",
      "Many coins are designed with hidden symbols representing national heritage.",
      "Commemorative coins celebrate historical events and famous figures.",
      "Some coins have been designed to be easily recognized by the visually impaired.",
      "The precision required in coin minting is a testament to modern engineering.",
      "Coins often feature reeded edges to prevent clipping or shaving of precious metal.",
      "In many cultures, tossing a coin is a way to make decisions or seek luck.",
      "Coin-operated machines paved the way for automated vending technology.",
      "Some coins are produced in limited editions, making them valuable collectibles.",
      "Eco-friendly coin designs are emerging to reduce the use of non-renewable metals.",
      "The weight and size of a coin are standardized to ensure consistency in vending machines."
    ];
    function generateCoinFact() {
      const randomIndex = Math.floor(Math.random() * coinFacts.length);
      coinFact.value = coinFacts[randomIndex];
    }

    const flipHistory = ref<string[]>([]);
    const showHistory = ref(false);
    const darkMode = ref(false);
    const rainEnabled = ref(false);

    // Generate coin rain data (30 coins)
    const rainCoins = Array.from({ length: 30 }, () => ({
      left: Math.random() * 100 + '%',
      size: (20 + Math.random() * 30) + 'px',
      delay: Math.random() * 5 + 's',
      duration: (5 + Math.random() * 5) + 's'
    }));

    // Helper function to convert hex color to RGBA string with desired alpha.
    function hexToRgba(hex: string, alpha: number): string {
      hex = hex.replace('#', '');
      if (hex.length === 3) {
        hex = hex.split('').map(c => c + c).join('');
      }
      const bigint = parseInt(hex, 16);
      const r = (bigint >> 16) & 255;
      const g = (bigint >> 8) & 255;
      const b = bigint & 255;
      return `rgba(${r}, ${g}, ${b}, ${alpha})`;
    }

    function resetSettings() {
      coinColor.value = '#FFD700';
      showTitle.value = true;
      showButton.value = true;
      flipDuration.value = 1;
      coinSize.value = 250;
      rainEnabled.value = false;
      coinFact.value = "Did you know? The US quarter has featured over 119 different designs since 1932.";
      flipHistory.value = [];
      showHistory.value = false;
      darkMode.value = false;
      previousResult.value = 'heads';
      coinResult.value = 'heads';
    }

    function flipCoin() {
      if (isFlipping.value) return;
      // Store the current face so that the flip animation starts from its current rotation.
      previousResult.value = coinResult.value || 'heads';

      let result;
      const history = flipHistory.value;
      let lastThreeSame = false;
      if (history.length >= 3) {
        const lastThree = history.slice(-3);
        if (lastThree.every(x => x === lastThree[0])) {
          lastThreeSame = true;
        }
      }
      if (lastThreeSame) {
        if (Math.random() < 0.8) {
          result = history[history.length - 1] === 'heads' ? 'tails' : 'heads';
        } else {
          result = history[history.length - 1];
        }
      } else {
        result = Math.random() < 0.5 ? 'heads' : 'tails';
      }
      coinResult.value = result;
      isFlipping.value = true;
    }

    function onAnimationEnd() {
      isFlipping.value = false;
      if (coinResult.value) {
        flipHistory.value.push(coinResult.value);
      }
    }

    function toggleSidebar() {
      sidebarVisible.value = !sidebarVisible.value;
    }

    return {
      coinColor,
      showTitle,
      showButton,
      isFlipping,
      coinResult,
      previousResult,
      flipCoin,
      onAnimationEnd,
      sidebarVisible,
      toggleSidebar,
      flipDuration,
      coinSize,
      coinFact,
      generateCoinFact,
      flipHistory,
      showHistory,
      darkMode,
      resetSettings,
      rainEnabled,
      rainCoins,
      hexToRgba
    };
  }
});
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500;700&display=swap');
/* Overall container */
#app {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  font-family: 'Roboto', sans-serif;
  background: #f7f7f7;
  color: #333;
  transition: background 0.3s ease, color 0.3s ease;
}
#app.dark {
  background: #1e1e1e;
  color: #eee;
}

/* Coin Rain */
.coin-rain {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  pointer-events: none;
  z-index: 0;
}
.rain-coin {
  position: absolute;
  top: -50px;
  border: 1px solid rgba(255, 255, 255, 0.5);
  border-radius: 50%;
  transform: rotate(0deg);
  animation: rainFall linear infinite;
}
@keyframes rainFall {
  0% {
    top: -50px;
    opacity: 0.8;
  }
  100% {
    top: 110vh;
    opacity: 0.2;
    transform: rotate(360deg);
  }
}

/* Sidebar styling */
.sidebar {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  width: 300px;
  padding: 1rem;
  background: #fff;
  border-right: 1px solid #ccc;
  box-shadow: 2px 0 5px rgba(0,0,0,0.1);
  transition: transform 0.3s ease;
  transform: translateX(0);
  display: flex;
  flex-direction: column;
  z-index: 10;
}
.sidebar-hidden {
  transform: translateX(-100%);
}
.sidebar.dark {
  background: #2c2c2c;
  border-right: 1px solid #444;
}
.sidebar h2, .sidebar h3 {
  margin: 0 0 0.5rem;
}
.sidebar label {
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
  display: block;
}
.magnification-tip,
.coin-tip {
  font-size: 0.85rem;
  margin: 0.5rem 0 1rem;
  color: #8a8a8a;
}
.setting {
  margin-bottom: 1rem;
}
.setting input[type="range"] {
  width: 100%;
}
.coin-fact p {
  font-size: 0.85rem;
  margin: 0.5rem 0;
}
/* Reset button at bottom */
.reset-settings {
  margin-top: auto;
}

/* Flip History Overlay */
.flip-history-overlay {
  position: absolute;
  top: 10px;
  transition: left 0.3s ease;
  background: rgba(255,255,255,0.9);
  padding: 0.5rem 1rem;
  border: 1px solid #ccc;
  border-radius: 4px;
  max-width: 200px;
  z-index: 1000;
}
#app.dark .flip-history-overlay {
  background: rgba(0,0,0,0.9);
  border-color: #444;
}

/* Main content */
.main-content {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  transition: transform 0.3s ease;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 5;
}
.main-shifted {
  transform: translateX(300px);
}

/* Coin container */
.coin-container {
  text-align: center;
}
.coin-container h2 {
  margin-bottom: 1rem;
  font-size: 2.5rem;
}

/* Coin styling */
.coin {
  border-radius: 50%;
  position: relative;
  transform-style: preserve-3d;
  transition: transform 0.5s ease;
  z-index: 2;
  border: 4px solid rgba(0, 0, 0, 0.3);
  cursor: pointer;
}
.coin .face {
  position: absolute;
  width: 100%;
  height: 100%;
  border-radius: 50%;
  backface-visibility: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 500;
  color: #fff;
}
.coin .face.front {
  background-color: inherit;
}
.coin .face.back {
  background-color: inherit;
  transform: rotateX(180deg);
}
.coin.flipping {
  animation: flip var(--flip-duration, 1s) ease-in-out forwards;
}
@keyframes flip {
  from {
    transform: rotateX(var(--initial-rotation));
  }
  to {
    transform: rotateX(calc(var(--initial-rotation) + var(--rotation-delta)));
  }
}

/* Button styling for light/dark modes */
button {
  margin-top: 1rem;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  cursor: pointer;
  border-radius: 4px;
  border: 1px solid;
  transition: background 0.3s ease, color 0.3s ease;
}
#app:not(.dark) button {
  background: #fff;
  color: #000;
  border-color: #ccc;
}
#app.dark button {
  background: #000;
  color: #fff;
  border-color: #444;
}
button:disabled {
  background: #ccc;
  cursor: not-allowed;
}
/* Button hover effect */
button:hover {
  filter: brightness(1.1);
}

/* Arrow toggle button */
.arrow-toggle {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 30px;
  height: 30px;
  border: none;
  background: #ccc;
  border-radius: 50%;
  cursor: pointer;
  opacity: 0.7;
  transition: opacity 0.3s ease, left 0.3s ease;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 16px;
  z-index: 20;
}
.arrow-toggle:hover {
  opacity: 1;
}
</style>
