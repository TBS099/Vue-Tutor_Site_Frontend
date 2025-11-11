<template>
  <!-- Navbar Component Start -->
  <nav class="navbar">
    <!-- Logo Section Start -->
    <h3 class="logo">ClassCart</h3>
    <!-- Logo Section End -->

    <!-- Navigation Links Start -->
    <ul>
      <li>
        <button class="nav-link" @click="$emit('navigate', 'lessons')">
          Lessons
        </button>
      </li>
      <li>
        <button class="nav-link" @click="$emit('navigate', 'cart')">
          Cart ({{ cartCount }})
        </button>
      </li>
    </ul>
    <!-- Navigation Links End -->
  </nav>
  <!-- Navbar Component End -->
</template>

<script>
// Importing CSS for navbar styling
import "./Navbar.css";

export default {
  name: "SiteNavbar",
  props: {
    cartCount: { type: Number, default: 0 }, // Number of items in cart
  },
};
</script>
