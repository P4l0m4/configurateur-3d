<script setup>
import { ref } from "vue";

let isNavOpen = ref(false);
let modelChosen = ref("microphone");

defineEmits(["model"]);
</script>
<template>
  <header class="header">
    <nav class="header__nav" v-if="isNavOpen">
      <button
        class="header__nav__button"
        @click="$emit('model', 'microphone'), (modelChosen = 'microphone')"
        :class="{ selected: modelChosen === 'microphone' }"
      >
        Microphone
      </button>
      <button
        class="header__nav__button"
        @click="$emit('model', 'helmet'), (modelChosen = 'helmet')"
        :class="{ selected: modelChosen === 'helmet' }"
      >
        Helmet
      </button>
    </nav>

    <img
      class="header__burger"
      src="@/assets/icons/burger.svg"
      alt=""
      @click="isNavOpen = !isNavOpen"
    />
  </header>
</template>
<style lang="scss">
.header {
  position: absolute;
  bottom: 2rem;
  right: 2rem;
  z-index: 1;
  padding: 1rem;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 2rem;
  &__burger {
    cursor: pointer;
    width: 2rem;
    height: 2rem;
  }

  &__nav {
    display: flex;
    flex-direction: column;
    padding: 1rem;
    background-color: rgba(255, 255, 255, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.1);
    border-radius: 20px;
    backdrop-filter: blur(2px);
    gap: 1rem;

    &__button {
      display: flex;
      background-color: transparent;
      border-radius: 20px;
      color: white;
      opacity: 0.8;
      transition: background-color 0.2s ease;
      padding: 1rem;

      &:hover {
        cursor: pointer;
        background-color: rgba(255, 255, 255, 0.1);
      }
    }
  }
}
.selected {
  background-color: rgba(255, 255, 255, 0.1);
}
</style>
