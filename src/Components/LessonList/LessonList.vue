<template>
  <div class="lesson-list-container">
    <!-- Always show filters sidebar -->
    <div class="filters-sidebar">
      <h3>Filter & Sort</h3>

      <!-- Search Box -->
      <div class="filter-group">
        <h4>Search</h4>
        <input
          type="text"
          v-model="searchQuery"
          placeholder="Search lessons..."
          class="search-input"
        />
      </div>

      <!-- Price Range -->
      <div class="filter-group">
        <h4>Price Range</h4>
        <div class="price-inputs">
          <input type="number" v-model="priceRange.min" placeholder="Min" class="price-input" />
          <span>-</span>
          <input type="number" v-model="priceRange.max" placeholder="Max" class="price-input" />
        </div>
      </div>

      <!-- Lesson Type Checkboxes -->
      <div class="filter-group">
        <h4>Subjects</h4>
        <div class="checkbox-group">
          <label v-for="subject in availableSubjects" :key="subject" class="checkbox-label">
            <input type="checkbox" v-model="selectedSubjects" :value="subject" />
            {{ subject }}
          </label>
        </div>
      </div>

      <!-- Location Checkboxes -->
      <div class="filter-group">
        <h4>Locations</h4>
        <div class="checkbox-group">
          <label v-for="location in availableLocations" :key="location" class="checkbox-label">
            <input type="checkbox" v-model="selectedLocations" :value="location" />
            {{ location }}
          </label>
        </div>
      </div>

      <!-- Sort Options -->
      <div class="filter-group">
        <h4>Sort By</h4>
        <select v-model="sortBy" class="sort-select">
          <option value="subject">Subject A-Z</option>
          <option value="subjectDesc">Subject Z-A</option>
          <option value="location">Location A-Z</option>
          <option value="locationDesc">Location Z-A</option>
          <option value="price">Price Low to High</option>
          <option value="priceDesc">Price High to Low</option>
          <option value="spaces">Most Spaces</option>
          <option value="spacesDesc">Fewest Spaces</option>
        </select>
      </div>

      <!-- Reset Filters -->
      <button @click="resetFilters" class="reset-btn">Reset All Filters</button>
    </div>

    <!-- Main Content -->
    <div class="lessons-main">
      <!-- Results Count -->
      <div class="results-info">
        <p>Showing {{ filteredLessons.length }} of {{ lessons.length }} lessons</p>
      </div>

      <!-- Lessons Grid -->
      <div class="lessons-grid">
        <LessonCard
          v-for="lesson in filteredLessons"
          :key="lesson.id"
          :lesson="lesson"
          :isExpanded="expandedId === lesson.id"
          @toggle="handleToggle"
          @add-to-cart="$emit('add-to-cart', $event)"
        />
      </div>

      <!-- No results message -->
      <div v-if="filteredLessons.length === 0 && lessons.length > 0" class="no-results">
        <p>No lessons found matching your criteria.</p>
        <button @click="resetFilters" class="reset-btn">Reset All Filters</button>
      </div>
    </div>
  </div>
</template>

<script>
import LessonCard from '../LessonCard/LessonCard.vue'
import './LessonList.css'

export default {
  name: 'LessonList',
  components: { LessonCard },
  props: {
    lessons: { type: Array, required: true },
  },
  data() {
    return {
      expandedId: null,
      searchQuery: '',
      priceRange: {
        min: null,
        max: null,
      },
      selectedSubjects: [],
      selectedLocations: [],
      sortBy: 'subject',
    }
  },
  computed: {
    availableSubjects() {
      return [...new Set(this.lessons.map((lesson) => lesson.subject))].sort()
    },
    availableLocations() {
      return [...new Set(this.lessons.map((lesson) => lesson.location))].sort()
    },
    filteredLessons() {
      let filtered = [...this.lessons]

      // Text search
      if (this.searchQuery) {
        const query = this.searchQuery.toLowerCase().trim()
        filtered = filtered.filter(
          (lesson) =>
            lesson.subject.toLowerCase().includes(query) ||
            lesson.location.toLowerCase().includes(query) ||
            (lesson.description && lesson.description.toLowerCase().includes(query)),
        )
      }

      // Price range filter
      if (this.priceRange.min !== null && this.priceRange.min !== '') {
        filtered = filtered.filter((lesson) => lesson.price >= parseFloat(this.priceRange.min))
      }
      if (this.priceRange.max !== null && this.priceRange.max !== '') {
        filtered = filtered.filter((lesson) => lesson.price <= parseFloat(this.priceRange.max))
      }

      // Subject filter
      if (this.selectedSubjects.length > 0) {
        filtered = filtered.filter((lesson) => this.selectedSubjects.includes(lesson.subject))
      }

      // Location filter
      if (this.selectedLocations.length > 0) {
        filtered = filtered.filter((lesson) => this.selectedLocations.includes(lesson.location))
      }

      // Sort
      return filtered.sort((a, b) => {
        switch (this.sortBy) {
          case 'subject':
            return a.subject.localeCompare(b.subject)
          case 'subjectDesc':
            return b.subject.localeCompare(a.subject)
          case 'location':
            return a.location.localeCompare(b.location)
          case 'locationDesc':
            return b.location.localeCompare(a.location)
          case 'price':
            return a.price - b.price
          case 'priceDesc':
            return b.price - a.price
          case 'spaces':
            return b.spaces - a.spaces
          case 'spacesDesc':
            return a.spaces - b.spaces
          default:
            return a.subject.localeCompare(b.subject)
        }
      })
    },
  },
  methods: {
    handleToggle(id) {
      this.expandedId = this.expandedId === id ? null : id
    },
    resetFilters() {
      this.searchQuery = ''
      this.priceRange = { min: null, max: null }
      this.selectedSubjects = []
      this.selectedLocations = []
      this.sortBy = 'subject'
    },
  },
}
</script>
