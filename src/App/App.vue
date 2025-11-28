<template>
  <div id="app">
    <!-- Navbar Section Start -->
    <SiteNavbar :cartCount="totalCartQuantity" @navigate="handleNavigation" />
    <!-- Navbar Section End -->

    <main class="main-content">
      <!-- Lesson List Section Start -->
      <LessonList
        v-if="currentView === 'lessons'"
        :lessons="lessons"
        @add-to-cart="addToCart"
      />
      <!-- Lesson List Section End -->

      <!-- Cart Section Start -->
      <StoreCart
        v-else-if="currentView === 'cart'"
        :cart="cart"
        @remove-from-cart="removeFromCart"
        @proceed-to-checkout="handleCheckout"
      />
      <!-- Cart Section End -->

      <!-- Checkout Success Section Start -->
      <div
        class="checkout-msg-div"
        v-else-if="currentView === 'checkout-success'"
      >
        <div class="checkout-success">
          <h2>Order Confirmed!</h2>
          <p>Thank you for your order. We'll contact you soon.</p>
          <button @click="currentView = 'lessons'" class="return btn">
            Back to Lessons
          </button>
        </div>
      </div>
      <!-- Checkout Success Section End -->
    </main>
  </div>
</template>

<script>
// Importing necessary components and styles
import SiteNavbar from "../Components/Navbar/Navbar.vue";
import LessonList from "../Components/LessonList/LessonList.vue";
import StoreCart from "../Components/StoreCart/StoreCart.vue";
import "./App.css";

export default {
  name: "App",
  components: {
    SiteNavbar,
    LessonList,
    StoreCart,
  },
  data() {
    return {
      currentView: "lessons", // Tracks current page view
      lessons: [], // Stores lesson data from backend
      cart: [], // Stores cart items
    };
  },
  computed: {
    // Calculates total quantity of items in cart
    totalCartQuantity() {
      return this.cart.reduce((sum, item) => sum + item.quantity, 0);
    },
  },
  created() {
    // Load cart from localStorage and fetch lessons on app start
    this.loadCartFromStorage();
    this.fetchLessons();
  },
  methods: {
    handleNavigation(view) {
      this.currentView = view;
    },

    // Fetch lessons from backend API
    fetchLessons() {
      fetch("https://vue-tutor-site-backend.onrender.com/lessons")
        .then((response) => response.json())
        .then((data) => {
          this.lessons = data;
          console.log("Fetched lessons:", data);
        })
        .catch((error) => {
          console.error("Error fetching lessons:", error);
        });
    },

    // Adds a lesson to the cart and updates spaces
    addToCart(lesson) {
      if (lesson.spaces > 0) {
        const existingItem = this.cart.find((item) => item.id === lesson.id);

        // If item already in cart, increase quantity
        if (existingItem) {
          existingItem.quantity += 1;
        } else {
          this.cart.push({ ...lesson, quantity: 1 });
        }

        // Decrease available spaces
        const lessonInList = this.lessons.find((l) => l.id === lesson.id);
        if (lessonInList) lessonInList.spaces -= 1;

        this.saveCartToStorage();
      }
    },

    // Removes a lesson from the cart and restores spaces
    removeFromCart(lessonId) {
      const cartIndex = this.cart.findIndex((item) => item.id === lessonId);

      if (cartIndex !== -1) {
        const cartItem = this.cart[cartIndex];
        const lessonInList = this.lessons.find((l) => l.id === lessonId);
        if (lessonInList) lessonInList.spaces += cartItem.quantity;

        this.cart.splice(cartIndex, 1);
        this.saveCartToStorage();

        // Update backend with restored spaces
        if (lessonInList)
          this.updateLessonSpaces(lessonId, lessonInList.spaces);
      }
    },

    // Sends order to backend and clears cart
    handleCheckout(orderData) {
      fetch("https://vue-tutor-site-backend.onrender.com/orders", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          name: orderData.name,
          email: orderData.email,
          phone: orderData.phone,
          lessonIds: this.cart.map((item) => item.id),
          total: this.cart.reduce(
            (total, item) => total + item.price * item.quantity,
            0
          ),
        }),
      })
        .then((response) => {
          if (!response.ok) throw new Error("Network response was not ok");
          return response.json();
        })
        .then(() => {
          this.cart = [];
          this.saveCartToStorage();
          this.currentView = "checkout-success";
        })
        .catch((error) => {
          console.error("Error placing order:", error);
          alert("Error placing order. Please try again.");
        });
    },

    // Updates lesson spaces in backend
    updateLessonSpaces(lessonId, newSpaces) {
      fetch(`https://vue-tutor-site-backend.onrender.com/lessons/${lessonId}`, {
        method: "PUT",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ spaces: newSpaces }),
      })
        .then((res) => res.json())
        .then((data) => console.log(`Updated lesson ${lessonId}:`, data))
        .catch((error) => console.error("Error updating spaces:", error));
    },

    // Saves cart to localStorage
    saveCartToStorage() {
      localStorage.setItem("shoppingCart", JSON.stringify(this.cart));
    },

    // Loads cart from localStorage
    loadCartFromStorage() {
      const savedCart = localStorage.getItem("shoppingCart");
      if (savedCart) this.cart = JSON.parse(savedCart);
    },
  },
  watch: {
    cart: {
      handler() {
        this.saveCartToStorage();
      },
      deep: true,
    },
  },
};
</script>
