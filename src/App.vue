<template>
  <div class="app-container">
    <header class="header">
      <n-date-picker
          v-model:value="selectedDate"
          type="date"
          class="date-picker"
          :disabled-date="disabledDate"
      />
    </header>
    <div class="image-container" :style="{ backgroundImage: `url(${imageUrl})` }">
      <div class="arrow left" @click="previousImage">&lt;</div>
      <div class="arrow right" @click="nextImage">&gt;</div>
      <div class="info-container">
        <h1>{{ title }}</h1>
        <p>{{ explanation }}</p>
        <p>{{ copyright }}</p>
        <p v-if="errorMessage">{{ errorMessage }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, watch } from 'vue';
import { NDatePicker } from 'naive-ui';
import axios from 'axios';

export default {
  components: {
    NDatePicker
  },
  setup() {
    const selectedDate = ref(Date.now());
    const imageUrl = ref('');
    const title = ref('');
    const explanation = ref('');
    const copyright = ref('');
    const apiKey = 'DEMO_KEY';  // Replace with your own API key
    const images = ref([]);
    const currentIndex = ref(0);
    const errorMessage = ref('');

    const formatDate = (timestamp) => {
      const date = new Date(timestamp);
      const year = date.getFullYear();
      const month = String(date.getMonth() + 1).padStart(2, '0');
      const day = String(date.getDate()).padStart(2, '0');
      return `${year}-${month}-${day}`;
    };

    const disabledDate = (timestamp) => {
      const today = new Date();
      today.setHours(0, 0, 0, 0);
      return timestamp > today.getTime();
    };

    const fetchImage = async (timestamp) => {
      try {
        const formattedDate = formatDate(timestamp);
        console.log(`Fetching image for date: ${formattedDate}`);
        const response = await axios.get(`https://api.nasa.gov/planetary/apod`, {
          params: {
            api_key: apiKey,
            date: formattedDate
          }
        });
        console.log('API Response:', response.data);
        imageUrl.value = response.data.hdurl; // Using hdurl for high-definition image
        title.value = response.data.title;
        explanation.value = response.data.explanation;
        copyright.value = response.data.copyright;
        images.value = [response.data.hdurl]; // Assuming we only fetch one image per request
        currentIndex.value = 0;
        errorMessage.value = ''; // Clear any previous error message
      } catch (error) {
        if (error.response && error.response.status === 429) {
          console.warn('Too many requests. Please try again later.');
          errorMessage.value = 'Too many requests. Please try again later.';
        } else {
          console.error('Error fetching image:', error);
          errorMessage.value = 'Error fetching image. Please try again later.';
        }
      }
    };

    const previousImage = () => {
      if (currentIndex.value > 0) {
        currentIndex.value--;
        imageUrl.value = images.value[currentIndex.value];
      }
    };

    const nextImage = () => {
      if (currentIndex.value < images.value.length - 1) {
        currentIndex.value++;
        imageUrl.value = images.value[currentIndex.value];
      }
    };

    watch(selectedDate, (newDate) => {
      console.log('Selected date changed:', newDate);
      fetchImage(newDate);
    });

    fetchImage(selectedDate.value);

    return {
      selectedDate,
      imageUrl,
      title,
      explanation,
      copyright,
      previousImage,
      nextImage,
      disabledDate,
      errorMessage
    };
  }
};
</script>

<style scoped>
.app-container {
  height: 100vh;
  margin: 0;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.header {
  display: flex;
  justify-content: flex-end;
  padding: 10px;
  background: rgba(0, 0, 0, 0.5);
}

.date-picker {
  width: 200px;
}

.image-container {
  flex: 1;
  background-size: cover;
  background-position: center;
  position: relative;
  display: flex;
  justify-content: center;
  align-items: center;
}

.arrow {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  font-size: 24px;
  color: white;
  cursor: pointer;
  background: rgba(0, 0, 0, 0.5);
  padding: 10px;
  border-radius: 50%;
}

.arrow.left {
  left: 10px;
}

.arrow.right {
  right: 10px;
}

.info-container {
  text-align: center;
  color: white;
  background: rgba(0, 0, 0, 0.5);
  padding: 20px;
  border-radius: 10px;
  max-width: 80%;
}
</style>
