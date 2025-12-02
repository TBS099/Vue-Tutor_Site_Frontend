<template>
  <!-- Cart Page Component Start -->
  <div class="cart-page">
    <!-- Cart Items Section Start -->
    <div class="cart-items">
      <h2>Your Cart</h2>

      <!-- Empty Cart Message -->
      <div v-if="cart.length === 0">
        <p>Your cart is empty.</p>
      </div>

      <!-- Cart Items List -->
      <div v-else>
        <div v-for="item in cart" :key="item.id" class="cart-item">
          <img :src="item.imageUrl" :alt="item.subject" class="cart-item-image" />
          <div class="cart-item-details">
            <h3>{{ item.subject }}</h3>
            <p>Location: {{ item.location }}</p>
            <p>Price: ${{ item.price }}</p>
            <p>Quantity: {{ item.quantity }}</p>
            <p>Subtotal: ${{ (item.price * item.quantity).toFixed(2) }}</p>
            <button @click="$emit('remove-from-cart', item.id)">Remove</button>
          </div>
        </div>
      </div>
    </div>
    <!-- Cart Items Section End -->

    <!-- Checkout Section Start -->
    <div class="checkout-section" v-if="cart.length > 0">
      <h2>Checkout</h2>

      <!-- Checkout Form -->
      <form
        @submit.prevent="$emit('proceed-to-checkout', { name, email, phone })"
      >
        <!-- Name Input -->
        <div>
          <label>Name:</label>
          <input
            type="text"
            v-model="name"
            required
            pattern="[A-Za-z ]+"
            title="Name should contain only letters and spaces"
          />
          <small v-if="name && !/^[A-Za-z ]+$/.test(name)" class="error">
            Name should contain only letters and spaces
          </small>
        </div>

        <!-- Email Input -->
        <div>
          <label>Email:</label>
          <input
            type="email"
            v-model="email"
            required
            pattern="[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$"
            title="Please enter a valid email address"
          />
          <small
            v-if="
              email &&
              !/^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$/.test(email)
            "
            class="error"
          >
            Please enter a valid email address
          </small>
        </div>

        <!-- Phone Input -->
        <div>
          <label>Phone:</label>
          <input
            type="tel"
            v-model="phone"
            required
            pattern="[0-9]+"
            title="Phone should contain only numbers"
          />
          <small v-if="phone && !/^[0-9]+$/.test(phone)" class="error">
            Phone should contain only numbers
          </small>
        </div>

        <!-- Order Summary -->
        <div class="order-summary">
          <h3>Order Summary</h3>
          <p>Total Items: {{ totalQuantity }}</p>
          <p>Total Amount: ${{ totalAmount }}</p>
        </div>

        <!-- Submit Button -->
        <button type="submit" :disabled="!isFormValid" class="checkout btn">
          Place Order
        </button>
      </form>
    </div>
    <!-- Checkout Section End -->
  </div>
  <!-- Cart Page Component End -->
</template>

<script>
// Importing CSS for StoreCart component
import "./StoreCart.css";

export default {
  name: "StoreCart",
  props: ["cart"],

  data() {
    return {
      name: "",
      email: "",
      phone: "",
    };
  },
  computed: {
    // Total amount of all items in the cart
    totalAmount() {
      return this.cart.reduce(
        (sum, item) => sum + item.price * item.quantity,
        0
      );
    },

    // Total quantity of all items in the cart
    totalQuantity() {
      return this.cart.reduce((sum, item) => sum + item.quantity, 0);
    },

    // Validates the checkout form inputs
    isFormValid() {
      const validName = /^[A-Za-z ]+$/.test(this.name);
      const validEmail =
        /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(this.email);
      const validPhone = /^[0-9]{10,12}$/.test(this.phone);

      return (
        this.name &&
        this.email &&
        this.phone &&
        validName &&
        validEmail &&
        validPhone
      );
    },
  },
};
</script>
