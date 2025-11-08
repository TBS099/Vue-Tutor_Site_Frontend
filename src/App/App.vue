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
      fetch('https://your-backend-url/lessons')
        .then((response) => response.json())
        .then((data) => {
          this.lessons = data
        })
        .catch((error) => {
          console.error('Error fetching lessons:', error)
          // Fallback to sample data for development
          this.lessons = this.getSampleLessons()
        })
    },

    getSampleLessons() {
      return [
        {
          id: 1,
          subject: 'Mathematics',
          location: 'London',
          price: 50,
          spaces: 5,
          image: '/images/math.jpg',
          description: 'Advanced mathematics course covering algebra and calculus',
          rating: 4.5,
        },
        {
          id: 2,
          subject: 'Physics',
          location: 'Manchester',
          price: 60,
          spaces: 3,
          image: '/images/physics.jpg',
          description: 'Physics fundamentals and practical applications',
          rating: 4.2,
        },
        {
          id: 3,
          subject: 'Chemistry',
          location: 'London',
          price: 55,
          spaces: 4,
          image: '/images/chemistry.jpg',
          description: 'Organic and inorganic chemistry principles',
          rating: 4.3,
        },
        {
          id: 4,
          subject: 'Biology',
          location: 'Birmingham',
          price: 45,
          spaces: 6,
          image: '/images/biology.jpg',
          description: 'Cell biology, genetics, and human anatomy',
          rating: 4.1,
        },
        {
          id: 5,
          subject: 'Computer Science',
          location: 'London',
          price: 70,
          spaces: 2,
          image: '/images/cs.jpg',
          description: 'Programming fundamentals and algorithms',
          rating: 4.7,
        },
        {
          id: 6,
          subject: 'English Literature',
          location: 'Oxford',
          price: 40,
          spaces: 8,
          image: '/images/english.jpg',
          description: 'Classic and contemporary literature analysis',
          rating: 4.0,
        },
        {
          id: 7,
          subject: 'History',
          location: 'Cambridge',
          price: 35,
          spaces: 7,
          image: '/images/history.jpg',
          description: 'World history and historical analysis',
          rating: 4.4,
        },
        {
          id: 8,
          subject: 'Art',
          location: 'London',
          price: 65,
          spaces: 4,
          image: '/images/art.jpg',
          description: 'Drawing, painting, and art theory',
          rating: 4.6,
        },
        {
          id: 9,
          subject: 'Music',
          location: 'Liverpool',
          price: 55,
          spaces: 5,
          image: '/images/music.jpg',
          description: 'Music theory and instrument training',
          rating: 4.3,
        },
        {
          id: 10,
          subject: 'Geography',
          location: 'Edinburgh',
          price: 30,
          spaces: 9,
          image: '/images/geography.jpg',
          description: 'Physical and human geography studies',
          rating: 3.9,
        },
      ]
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
      fetch('https://your-backend-url/orders', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          name: orderData.name,
          phone: orderData.phone,
          lessonIDs: this.cart.map((item) => item.id),
          spaces: this.cart.reduce((total, item) => total + item.quantity, 0),
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
      fetch(`https://your-backend-url/lesson/${lessonId}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({ spaces: newSpaces }),
      }).catch((error) => console.error('Error updating spaces:', error))
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
