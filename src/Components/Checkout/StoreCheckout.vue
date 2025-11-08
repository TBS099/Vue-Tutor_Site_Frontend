<template>
  <div class="checkout-page">
    <h2>Checkout</h2>

    <form @submit.prevent="submitOrder">
      <div>
        <label>Name:</label>
        <input type="text" v-model="name" required pattern="[A-Za-z ]+" />
      </div>

      <div>
        <label>Phone:</label>
        <input type="tel" v-model="phone" required pattern="[0-9]+" />
      </div>

      <div>
        <h3>Cart Items</h3>
        <ul>
          <li v-for="item in cart" :key="item.id">{{ item.title }} - ${{ item.price }}</li>
        </ul>
      </div>

      <button type="submit">Place Order</button>
    </form>
  </div>
</template>

<script>
export default {
  name: 'StoreCheckout',
  props: {
    cart: { type: Array, required: true },
  },
  data() {
    return {
      name: '',
      phone: '',
    }
  },
  methods: {
    submitOrder() {
      // emit order to parent App
      const order = {
        name: this.name,
        phone: this.phone,
        items: this.cart,
      }
      this.$emit('place-order', order)

      // clear form
      this.name = ''
      this.phone = ''
    },
  },
}
</script>

<style scoped>
.checkout-page {
  padding: 20px;
}

input {
  display: block;
  margin-bottom: 10px;
}
</style>
