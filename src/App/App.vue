<template>
  <div id="app">
    <SiteNavbar :cartCount="totalCartQuantity" @navigate="handleNavigation" />
  </div>
</template>

<script>
import SiteNavbar from '../Components/Navbar/Navbar.vue'

</script>
