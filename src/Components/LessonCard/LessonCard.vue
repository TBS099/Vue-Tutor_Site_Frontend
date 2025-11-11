<template>
  <div class="lesson-card">
    <!-- Card Image Section Start -->
    <div class="card-img-container">
      <img :src="lesson.imageUrl" alt="Lesson Image" class="lesson-image" />
    </div>
    <!-- Card Image Section End -->

    <!-- Basic Lesson Info Start -->
    <h3 class="lesson-title">{{ lesson.subject }}</h3>
    <p class="lesson-location">Location: {{ lesson.location }}</p>
    <p class="lesson-price">Price: ${{ lesson.price }}</p>
    <p class="lesson-spaces">Spaces: {{ lesson.spaces }}</p>
    <!-- Basic Lesson Info End -->

    <!-- Expanded Lesson Info Start -->
    <div :class="['lesson-details', { show: isExpanded }]">
      <div class="price-rating">
        <p class="lesson-rating">Rating: {{ lesson.rating }} / 5</p>
      </div>
      <p class="lesson-description">{{ lesson.description }}</p>
      <button
        class="add-cart-btn"
        @click="$emit('add-to-cart', lesson)"
        :disabled="lesson.spaces === 0"
      >
        {{ lesson.spaces === 0 ? "No Spaces" : "Add to Cart" }}
      </button>
    </div>
    <!-- Expanded Lesson Info End -->

    <!-- Expand Button Start -->
    <div class="card-expand">
      <button class="expand-btn" @click="$emit('toggle', lesson.id)">
        <span :class="['arrow', { rotated: isExpanded }]">&#9660;</span>
      </button>
    </div>
    <!-- Expand Button End -->
  </div>
</template>

<script>
// Importing CSS for LessonCard component
import "./LessonCard.css";

export default {
  name: "LessonCard",
  props: {
    lesson: {
      type: Object,
      required: true,
      // Ensures lesson object has all required fields
      validator: function (lesson) {
        return (
          lesson.id &&
          lesson.subject &&
          lesson.location &&
          lesson.price !== undefined &&
          lesson.spaces !== undefined
        );
      },
    },
    isExpanded: { type: Boolean, default: false }, // Controls expanded view
  },

  // Events emitted to parent
  emits: ["toggle", "add-to-cart"],
};
</script>
