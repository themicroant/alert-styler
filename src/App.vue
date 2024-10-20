<script lang="ts" setup>
import {ref, computed} from 'vue';

const borderColorHue = ref(210);
const borderWidth = ref(4);
const borderType = ref(0);
const backgroundOpacity = ref(0.2);
const shadowSize = ref(5);
const borderRadius = ref(8);
const buttonText = ref('Copy Styles');
const buttonClass = ref('');

const borderTypes = ['solid', 'dashed', 'dotted', 'double', 'groove', 'ridge', 'inset', 'outset'];

const borderColor = computed(() => hslToHex(borderColorHue.value, 1, 0.5));
const backgroundColor = computed(() => hexToRgba(borderColor.value, backgroundOpacity.value));
const backgroundOpacityPercentage = computed(() => {
  return Math.round(backgroundOpacity.value * 100);
});
const shadowStyle = computed(() => {
  return `${shadowSize.value}px ${shadowSize.value}px ${shadowSize.value * 2}px rgba(0, 0, 0, 0.3)`;
});

function hexToRgba(hex: string, alpha: number) {
  const r = parseInt(hex.slice(1, 3), 16);
  const g = parseInt(hex.slice(3, 5), 16);
  const b = parseInt(hex.slice(5, 7), 16);
  return `rgba(${r}, ${g}, ${b}, ${alpha})`;
}

// For greater understanding check out https://en.wikipedia.org/wiki/Munsell_color_system
function hslToHex(hue: number, saturation: number, lightness: number) {
  const chroma = (1 - Math.abs(2 * lightness - 1)) * saturation;
  const x = chroma * (1 - Math.abs((hue / 60) % 2 - 1));
  const m = lightness - chroma / 2;
  let r: number, g: number, b: number;

  if (0 <= hue && hue < 60) {
    [r, g, b] = [chroma, x, 0];
  } else if (60 <= hue && hue < 120) {
    [r, g, b] = [x, chroma, 0];
  } else if (120 <= hue && hue < 180) {
    [r, g, b] = [0, chroma, x];
  } else if (180 <= hue && hue < 240) {
    [r, g, b] = [0, x, chroma];
  } else if (240 <= hue && hue < 300) {
    [r, g, b] = [x, 0, chroma];
  } else {
    [r, g, b] = [chroma, 0, x];
  }

  const red = Math.round((r + m) * 255);
  const green = Math.round((g + m) * 255);
  const blue = Math.round((b + m) * 255);

  return `#${((1 << 24) + (red << 16) + (green << 8) + blue).toString(16).slice(1).toUpperCase()}`;
}

function copyCssStyles() {
  const cssStyles = `
    border-color: ${borderColor.value};
    border-style: ${borderTypes[borderType.value]};
    background-color: ${backgroundColor.value};
    border-radius: ${borderRadius.value}px;
    border-width: ${borderWidth.value}px;
    box-shadow: ${shadowStyle.value};
  `;
  navigator.clipboard.writeText(cssStyles).then(() => {
    buttonText.value = 'Copied!';
    buttonClass.value = 'clicked';

    setTimeout(() => {
      buttonText.value = 'Copy Styles';
      buttonClass.value = '';
    }, 1000);
  });
}
</script>

<template>
  <div>
    <div
        class="alert-box"
        :style="{
        borderColor: borderColor,
        borderWidth: borderWidth + 'px',
        borderStyle: borderTypes[borderType],
        backgroundColor: backgroundColor,
        borderRadius: borderRadius + 'px',
        boxShadow: shadowStyle,
      }"
    >
      <h4>Alert Styler</h4>
      <p>Find your preferred styles, then click Copy Styles to send it to your clipboard.</p>
    </div>

    <div class="controls">
      <div class="control">
        <div class="label-div">
          <label>Border Color:</label>
          <div class="color-box" :style="{ backgroundColor: borderColor }"></div>
          <span>{{ borderColor }}</span>
        </div>
        <input type="range" v-model="borderColorHue" min="0" max="360"/>
      </div>

      <div class="control">
        <label>Border Width: {{ borderWidth }}px</label>
        <input type="range" v-model="borderWidth" min="0" max="20"/>
      </div>

      <div class="control">
        <label>Border Type: {{ borderTypes[borderType] }}</label>
        <input type="range" v-model="borderType" min="0" :max="borderTypes.length - 1" step="1"/>
      </div>

      <div class="control">
        <label>Background Opacity: {{ backgroundOpacityPercentage }}%</label>
        <input type="range" v-model="backgroundOpacity" min="0" max="1" step="0.01"/>
      </div>

      <div class="control">
        <label>Border Radius: {{ borderRadius }}px</label>
        <input type="range" v-model="borderRadius" min="0" max="50"/>
      </div>

      <div class="control">
        <label>Shadow Size: {{ shadowSize }}px</label>
        <input type="range" v-model="shadowSize" min="0" max="25"/>
      </div>

      <div class="copy-div">
        <button @click="copyCssStyles" :class="buttonClass">{{ buttonText }}</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.alert-box {
  box-sizing: border-box;
  text-align: center;
  transition: all 0.3s ease;
  width: 100%;
  max-width: 400px;
  padding: 0 30px;
}

.alert-box h4 {
  margin: 10px 0;
}

.controls {
  border: 1px solid #ccc;
  padding: 16px;
  border-radius: 8px;
  margin-top: 15px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.control {
  margin-bottom: 10px;
  display: flex;
  flex-direction: column;
}

.label-div {
  display: flex;
}

.control label {
  margin-bottom: 12px;
}

input[type='range'] {
  width: 100%;
}

.color-box {
  width: 20px;
  height: 20px;
  margin-left: 8px;
  margin-right: 4px;
  border: 1px solid #000;
}

.copy-div {
  text-align: center;
}
.copy-div button {
  width: 140px;
}

.copy-div button.clicked {
  color: limegreen;
  cursor: default;
}
</style>
