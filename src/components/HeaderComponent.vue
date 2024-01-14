<template>
  <header class="header">
    <img
      class="header__logo"
      src="@/assets/images/tacky-sound-logo.svg"
      alt="logo"
    />
  </header>
</template>
<style lang="scss">
.header {
  position: relative;
  z-index: 1;
  display: flex;
  gap: 2rem;
  align-items: center;

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
