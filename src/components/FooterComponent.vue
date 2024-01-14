<template>
  <footer class="footer">
    <NuxtLink to="/" class="footer__link" exact>Legal mentions</NuxtLink>
    <span class="footer__link" exact>Free shipping and free returns</span>
  </footer>
</template>
<style lang="scss">
.footer {
  display: flex;
  justify-content: flex-end;
  padding: 0.5rem;
  gap: 1rem;
  position: absolute;
  width: 100%;
  bottom: 0;
  right: 0;
  left: 0;
  margin: auto;

  @media (min-width: $big-tablet-screen) {
    right: 220px;
    padding: 1rem;
    margin: inherit;
  }

  &__link {
    font-size: 0.75rem;
    text-decoration: none;
    opacity: 0.6;
    text-shadow: $shadow;

    &:nth-child(2) {
      opacity: 1;
      text-transform: uppercase;
    }
  }
}
</style>
