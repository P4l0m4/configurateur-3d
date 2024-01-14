<template>
  <header class="header">
    <img
      class="header__logo"
      src="@/assets/images/tacky-sound-logo.svg"
      alt="logo"
    />
    <nav class="header__nav">
      <NuxtLink to="/" class="header__nav__link" exact>Men</NuxtLink>
      <NuxtLink to="/" class="header__nav__link" exact>Women</NuxtLink>
      <NuxtLink to="/" class="header__nav__link" exact>Kids</NuxtLink>
      <NuxtLink to="/" class="header__nav__link" exact>Customize</NuxtLink>
    </nav>
  </header>
</template>
<style lang="scss">
.header {
  position: relative;
  z-index: 1;
  display: flex;
  gap: 2rem;
  align-items: center;
  backdrop-filter: blur(2px);

  &__logo {
    width: 170px;
    padding: 2rem;
  }

  &__nav {
    display: flex;
    gap: 2rem;
    justify-content: center;
    width: 100%;

    &__link {
      font-weight: $thick;
      text-shadow: $text-shadow;

      &:nth-of-type(4) {
        text-decoration: underline;
        text-decoration-thickness: 2px;
        text-underline-offset: 4px;
      }
    }
  }
}
// .router-link-exact-active {
//   text-decoration: underline;
//   text-decoration-thickness: 2px;
//   text-underline-offset: 4px;
// }
</style>
