<template>
  <nav class="navbar">
    <h3 class="logo">ClassCart</h3>
    <ul>
      <li><button class="nav-link" @click="$emit('navigate', 'lessons')">Lessons</button></li>
      <li>
        <button class="nav-link" @click="$emit('navigate', 'cart')">Cart ({{ cartCount }})</button>
      </li>
    </ul>
  </nav>
</template>

<script>
import './Navbar.css'

export default {
  name: 'SiteNavbar',
  props: {
    cartCount: { type: Number, default: 0 },
  },
}
</script>
