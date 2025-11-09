<template>
  <div id="app">
    <SiteNavbar :cartCount="totalCartQuantity" @navigate="handleNavigation" />

    <main class="main-content">
      <LessonList v-if="currentView === 'lessons'" :lessons="lessons" @add-to-cart="addToCart" />

      <StoreCart
        v-else-if="currentView === 'cart'"
        :cart="cart"
        @remove-from-cart="removeFromCart"
        @proceed-to-checkout="handleCheckout"
      />

      <div v-else-if="currentView === 'checkout-success'">
        <div class="checkout-success">
          <h2>Order Confirmed! ✅</h2>
          <p>Thank you for your order. We'll contact you soon.</p>
          <button @click="currentView = 'lessons'">Back to Lessons</button>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import SiteNavbar from '../Components/Navbar/Navbar.vue'
import LessonList from '../Components/LessonList/LessonList.vue'
import StoreCart from '../Components/StoreCart/StoreCart.vue'
import './App.css'

export default {
  name: 'App',
  components: {
    SiteNavbar,
    LessonList,
    StoreCart,
  },
  data() {
    return {
      currentView: 'lessons',
      lessons: [],
      cart: [],
    }
  },
  computed: {
    totalCartQuantity() {
      return this.cart.reduce((sum, item) => sum + item.quantity, 0)
    },
  },
  created() {
    this.loadCartFromStorage()
    this.fetchLessons()
  },
  methods: {
    handleNavigation(view) {
      this.currentView = view
    },

    fetchLessons() {
      // Fetch lessons from your backend API
      fetch('http://localhost:3000/lessons')
        .then((response) => response.json())
        .then((data) => {
          this.lessons = data
          console.log('Fetched lessons:', data)
        })
        .catch((error) => {
          console.error('Error fetching lessons:', error)
        })
    },

    addToCart(lesson) {
      console.log('Adding to cart:', lesson.subject, 'Spaces available:', lesson.spaces)

      if (lesson.spaces > 0) {
        // Check if lesson already in cart
        const existingItem = this.cart.find((item) => item.id === lesson.id)

        if (existingItem) {
          // Increase quantity if already in cart
          existingItem.quantity += 1
          console.log('Increased quantity to:', existingItem.quantity)
        } else {
          // Add new item to cart with quantity
          this.cart.push({
            ...lesson,
            quantity: 1,
          })
          console.log('Added new item to cart')
        }

        // Update available spaces in lessons list
        const lessonInList = this.lessons.find((l) => l.id === lesson.id)
        if (lessonInList) {
          lessonInList.spaces -= 1
          console.log('Updated spaces to:', lessonInList.spaces)
        }

        this.saveCartToStorage()

        // Update spaces in backend
        this.updateLessonSpaces(lesson.id, lessonInList.spaces)
      } else {
        console.log('No spaces available for:', lesson.subject)
      }
    },

    removeFromCart(lessonId) {
      console.log('Removing from cart:', lessonId)
      const cartIndex = this.cart.findIndex((item) => item.id === lessonId)

      if (cartIndex !== -1) {
        const cartItem = this.cart[cartIndex]

        // Restore spaces to lessons list
        const lessonInList = this.lessons.find((l) => l.id === lessonId)
        if (lessonInList) {
          lessonInList.spaces += cartItem.quantity
          console.log('Restored spaces to:', lessonInList.spaces)
        }

        // Remove from cart
        this.cart.splice(cartIndex, 1)
        this.saveCartToStorage()

        // Update backend
        if (lessonInList) {
          this.updateLessonSpaces(lessonId, lessonInList.spaces)
        }
      }
    },

    handleCheckout(orderData) {
      // Send order to backend
      fetch('http://localhost:3000/orders', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          name: orderData.name,
          phone: orderData.phone,
          lessonIds: this.cart.map((item) => item.id),
          total: this.cart.reduce((total, item) => total + item.price * item.quantity, 0)
        }),
      })
        .then((response) => response.json())
        .then((data) => {
          console.log('Order placed:', data)
          this.cart = []
          this.saveCartToStorage()
          this.currentView = 'checkout-success'
        })
        .catch((error) => {
          console.error('Error placing order:', error)
          alert('Error placing order. Please try again.')
        })
    },

    updateLessonSpaces(lessonId, newSpaces) {
      fetch(`http://localhost:3000/lessons/${lessonId}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ spaces: newSpaces }),
      })
      .then((res) => res.json())
      .then((data) => console.log(`Updated lesson ${lessonId}:`, data))
      .catch((error) => console.error("Error updating spaces:", error))
    },

    saveCartToStorage() {
      localStorage.setItem('shoppingCart', JSON.stringify(this.cart))
    },

    loadCartFromStorage() {
      const savedCart = localStorage.getItem('shoppingCart')
      if (savedCart) {
        this.cart = JSON.parse(savedCart)
      }
    },
  },
  watch: {
    cart: {
      handler() {
        this.saveCartToStorage()
      },
      deep: true,
    },
  },
}
</script>
