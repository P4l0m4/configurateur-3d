<script setup>
import { ref, onMounted } from "vue";
const colorInput = ref(null);
const customColor = ref([0.109, 0.58, 0.166]);

function hexToRgb(hex) {
  let r = parseInt(hex.slice(1, 3), 16);
  let g = parseInt(hex.slice(3, 5), 16);
  let b = parseInt(hex.slice(5, 7), 16);
  return [r / 100, g / 100, b / 100];
}

function openColorPicker() {
  colorInput.value.click();
}
const emit = defineEmits(["customColor"], ["metalType"]);
//EMIT SELECTED COLOR TO PARENT COMPONENT
function emitColor(color) {
  emit("customColor", color);
}
function emitMetalType(value) {
  emit("metalType", value);
}
</script>
<template>
  <aside class="aside">
    <button class="aside__menu">
      <img
        class="aside__menu__icon"
        src="@/assets/icons/stripes.svg"
        alt="stripes"
      />
    </button>
    <div class="aside__links">
      <button class="aside__links__link" @click="emitColor([0.8, 0.8, 0.8])">
        <span id="white" class="aside__links__link__icon"></span>White</button
      ><button class="aside__links__link" @click="emitColor([0, 0, 0.25])">
        <span id="navy" class="aside__links__link__icon"></span>Navy</button
      ><button class="aside__links__link" @click="emitColor([0.2, 0, 0])">
        <span id="red" class="aside__links__link__icon"></span>Red
      </button>
      <button class="aside__links__link" @click="emitColor([0, 0, 0])">
        <span id="black" class="aside__links__link__icon"></span>Black</button
      ><button class="aside__links__link" @click="openColorPicker()">
        <input
          type="color"
          ref="colorInput"
          v-model="customColor"
          @input="emitColor(hexToRgb(customColor))"
        /><span
          id="rainbow"
          class="aside__links__link__icon"
          :style="`background-color: ${hexToRgb(customColor)}`"
        ></span
        >Custom
      </button>
      <button
        class="aside__links__link"
        @click="emitMetalType([0.67, 0.57, 0.189])"
      >
        <span id="gold" class="aside__links__link__icon"></span>Gold
      </button>
      <button
        class="aside__links__link"
        @click="emitMetalType([0.6, 0.6, 0.6])"
      >
        <span id="silver" class="aside__links__link__icon"></span>Silver
      </button>
      <button
        class="aside__links__link"
        @click="emitMetalType([0.105, 0.105, 0.105])"
      >
        <span id="onyx" class="aside__links__link__icon"></span>Onyx
      </button>
    </div>
  </aside>
</template>
<style lang="scss" scoped>
.aside {
  position: fixed;
  top: 0;
  bottom: 0;
  right: 0;
  z-index: 1;
  display: flex;
  flex-direction: column;
  background-color: linear-gradient(
    180deg,
    rgba(255, 255, 255, 0.5) 0%,
    rgb(255, 255, 255) 100%
  );

  &__menu {
    height: 100px;
    border-radius: 0;
    display: flex;
    background-color: $base-color;
    justify-content: center;
    align-items: center;
    width: 100px;
    position: relative;

    &__icon {
      width: 30px;
      cursor: pointer;
    }
  }

  &__links {
    position: absolute;
    right: 0;
    top: 100px;
    display: flex;
    flex-direction: column;
    width: 100px;
    height: calc(100svh - 100px);
    background: linear-gradient($primary-color, black);
    padding: 1rem;
    height: calc(100svh - 100px);
    overflow-y: scroll;

    &__link {
      display: flex;
      justify-content: center;
      align-items: center;
      flex-direction: column;
      gap: 0.5rem;
      padding: 1rem 0;
      font-weight: $skinny-thick;
      color: $text-color;
      & #white {
        background-color: white;
      }
      & #navy {
        background-color: rgb(0, 0, 92);
      }
      & #red {
        background-color: rgb(92, 0, 18);
      }
      & #black {
        background-color: rgb(0, 0, 0);
      }
      & #rainbow {
        position: relative;
      }
      & #gold {
        background: linear-gradient(
          45deg,
          rgba(255, 219, 101) 0%,
          rgb(255, 255, 255) 50%,
          rgba(255, 219, 101) 54%
        );
      }
      & #silver {
        background: linear-gradient(
          45deg,
          rgb(106, 106, 106) 0%,
          rgb(255, 255, 255) 50%,
          rgb(106, 106, 106) 54%
        );
      }
      & #onyx {
        background: linear-gradient(
          45deg,
          rgb(39, 39, 39) 0%,
          rgb(255, 255, 255) 50%,
          rgb(39, 39, 39) 54%
        );
      }

      & input[type="color"] {
        width: 0;
        height: 0;
        opacity: 0;
        inset: 0;
        margin: auto;
        position: absolute;
      }

      &__icon {
        width: 47px;
        height: 47px;
        border-radius: 50%;
        border: 4px solid $secondary-color;
        transition: transform 0.2s;

        &:hover {
          transform: scale(1.1);
          cursor: pointer;
        }
      }
    }
  }
}
</style>
