<script setup>
import { ref, onMounted } from "vue";
const toggleMenu = ref(false);
const colorInput = ref(null);
const customColor = ref([0.109, 0.58, 0.166]);
const selectedColor = ref("white");
const selectedMetal = ref("gold");

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

onMounted(() => {
  if (window.innerWidth > 1024) {
    toggleMenu.value = true;
  }
});
</script>
<template>
  <aside class="aside">
    <button class="aside__menu" @click="toggleMenu = !toggleMenu">
      <img
        class="aside__menu__icon"
        src="@/assets/icons/stripes.svg"
        alt="stripes"
      />
    </button>
    <div class="aside__links" v-if="toggleMenu">
      <div class="aside__links__section">
        <h2 class="aside__links__section__title">Colors</h2>
        <button
          class="aside__links__section__link"
          @click="emitColor([0.8, 0.8, 0.8]), (selectedColor = 'white')"
          :class="{
            'aside__links__section__link--selected': selectedColor === 'white',
          }"
        >
          <span id="white" class="aside__links__section__link__icon"></span
          >White</button
        ><button
          class="aside__links__section__link"
          @click="emitColor([0, 0, 0.25]), (selectedColor = 'navy')"
          :class="{
            'aside__links__section__link--selected': selectedColor === 'navy',
          }"
        >
          <span id="navy" class="aside__links__section__link__icon"></span
          >Navy</button
        ><button
          class="aside__links__section__link"
          @click="emitColor([0.2, 0, 0]), (selectedColor = 'red')"
          :class="{
            'aside__links__section__link--selected': selectedColor === 'red',
          }"
        >
          <span id="red" class="aside__links__section__link__icon"></span>Red
        </button>
        <button
          class="aside__links__section__link"
          @click="emitColor([0, 0, 0]), (selectedColor = 'black')"
          :class="{
            'aside__links__section__link--selected': selectedColor === 'black',
          }"
        >
          <span
            id="black"
            class="aside__links__section__link__icon"
            :class="{
              'aside__links__section__link--selected':
                selectedColor === 'black',
            }"
          ></span
          >Black</button
        ><button
          class="aside__links__section__link"
          @click="openColorPicker(), (selectedColor = 'custom')"
          @tap="openColorPicker(), (selectedColor = 'custom')"
          :class="{
            'aside__links__section__link--selected': selectedColor === 'custom',
          }"
        >
          <input
            type="color"
            ref="colorInput"
            v-model="customColor"
            @input="emitColor(hexToRgb(customColor))"
          /><span
            id="rainbow"
            class="aside__links__section__link__icon"
            :style="`background-color: ${hexToRgb(customColor)}`"
          ></span
          >Custom
        </button>
      </div>
      <div class="aside__links__section">
        <h2 class="aside__links__section__title">Metals</h2>
        <button
          class="aside__links__section__link"
          @click="emitMetalType([0.67, 0.57, 0.189]), (selectedMetal = 'gold')"
          :class="{
            'aside__links__section__link--selected': selectedMetal === 'gold',
          }"
        >
          <span id="gold" class="aside__links__section__link__icon"></span>Gold
        </button>
        <button
          class="aside__links__section__link"
          @click="emitMetalType([0.6, 0.6, 0.6]), (selectedMetal = 'silver')"
          :class="{
            'aside__links__section__link--selected': selectedMetal === 'silver',
          }"
        >
          <span id="silver" class="aside__links__section__link__icon"></span
          >Silver
        </button>
        <button
          class="aside__links__section__link"
          @click="
            emitMetalType([0.105, 0.105, 0.105]), (selectedMetal = 'onyx')
          "
          :class="{
            'aside__links__section__link--selected': selectedMetal === 'onyx',
          }"
        >
          <span id="onyx" class="aside__links__section__link__icon"></span>Onyx
        </button>
      </div>
    </div>
  </aside>
</template>
<style lang="scss" scoped>
.aside {
  position: fixed;
  top: 0;
  bottom: 0;
  right: 60px;
  width: fit-content;
  z-index: 1;
  display: flex;
  flex-direction: column;

  &__menu {
    height: 100px;
    border-radius: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    position: relative;
    background-color: rgb($base-color, 0.6);
    backdrop-filter: blur(6px);
    padding: 1rem;

    @media (min-width: $big-tablet-screen) {
      min-width: 220px;
    }

    &__icon {
      width: 47px;
      cursor: pointer;
    }
  }

  &__links {
    background-color: rgb($base-color, 0.3);
    backdrop-filter: blur(6px);
    display: flex;
    height: calc(100svh - 100px);
    padding: 1rem 0.5rem;
    gap: 1rem;
    overflow-y: scroll;
    flex-direction: column;
    align-items: center;

    @media (min-width: $big-tablet-screen) {
      flex-direction: row;
      padding: 1rem;
      align-items: flex-start;
    }

    &__section {
      display: flex;
      flex-direction: column;
      align-items: center;
      width: fit-content;

      &__title {
        font-size: 1.25rem;
        font-weight: $thick;
        color: $text-color;
        margin-bottom: 1rem;
        text-shadow: $text-shadow;
      }

      &__link {
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        gap: 0.5rem;
        padding: 1rem;
        font-weight: $skinny-thick;
        color: $text-color;
        font-size: 0.8rem;

        @media (min-width: $big-tablet-screen) {
          font-size: 1rem;
        }

        &--selected {
          background-color: rgb($base-color, 0.2);
          border-radius: $radius;
        }

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
          background-image: url("@/assets/icons/palette.svg");
          background-size: 20px;
          background-repeat: no-repeat;
          background-position: center;

          @media (min-width: $big-tablet-screen) {
            background-size: 30px;
          }
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
          width: 30px;
          height: 30px;
          border-radius: 50%;
          border: 2px solid $text-color;
          transition: transform 0.2s;

          @media (min-width: $big-tablet-screen) {
            width: 47px;
            height: 47px;
          }

          &:hover {
            transform: scale(1.1);
            cursor: pointer;
          }
        }
      }
    }
  }
}
</style>
