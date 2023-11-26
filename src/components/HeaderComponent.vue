<script setup>
import { ref } from "vue";

let isNavOpen = ref(false);
</script>
<template>
  <header
    class="header"
    @click.self="isNavOpen = false"
    :class="{ opened: isNavOpen === true }"
  >
    <nav class="header__nav" v-if="isNavOpen">
      <NuxtLink to="/thermometer" class="header__nav__button" exact
        >Thermometer</NuxtLink
      >
      <NuxtLink to="/" class="header__nav__button" exact>Microphone</NuxtLink>
      <NuxtLink to="/helmet" class="header__nav__button" exact>Helmet</NuxtLink>
    </nav>

    <img
      class="header__burger"
      src="@/assets/icons/burger.svg"
      alt="icone burger"
      @click="isNavOpen = !isNavOpen"
    />
  </header>
</template>
<style lang="scss">
.header {
  position: absolute;
  padding: 0 2rem 2rem 0;
  z-index: 1;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  justify-content: flex-end;
  gap: 2rem;
  right: 0;
  bottom: 0;

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
      text-shadow: black 0px 0px 2px;
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
.router-link-exact-active {
  background-color: rgba(255, 255, 255, 0.1);
}
.opened {
  // width: 100%;
  // height: 100%;
}
</style>
