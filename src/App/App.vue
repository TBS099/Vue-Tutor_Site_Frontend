<template>
  <div id="app">
    <SiteNavbar :cartCount="totalCartQuantity" @navigate="handleNavigation" />

    <main class="main-content">
      <LessonList v-if="currentView === 'lessons'" :lessons="lessons" @add-to-cart="addToCart" />
    </main>
  </div>
</template>

<script>
import SiteNavbar from '../Components/Navbar/Navbar.vue'
import LessonList from '../Components/LessonList/LessonList.vue'
import './App.css'

export default {
  name: 'App',
  components: {
    SiteNavbar,
    LessonList,
  },
  data() {
    return {
      currentView: 'lessons',
      lessons: [],
      cart: [],
    }
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
    }
  },
}
</script>
