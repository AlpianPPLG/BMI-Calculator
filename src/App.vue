<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center p-4">
    <div class="bg-white shadow-md rounded-lg p-6 w-full max-w-md">
      <h1 class="text-2xl font-bold mb-4 text-center cursor-pointer" @click="reloadPage">
        BMI Calculator
      </h1>
      <form @submit.prevent="calculateBMI">
        <div class="mb-4">
          <label for="weight" class="block text-sm font-medium text-gray-700">Weight (kg)</label>
          <input
            v-model.number="weight"
            type="number"
            id="weight"
            class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            required
          />
        </div>
        <div class="mb-4">
          <label for="height" class="block text-sm font-medium text-gray-700">Height (cm)</label>
          <input
            v-model.number="height"
            type="number"
            id="height"
            class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            required
          />
        </div>
        <button
          type="submit"
          class="w-full bg-indigo-600 text-white py-2 rounded-md hover:bg-indigo-700 focus:ring-2 focus:ring-indigo-500 focus:ring-opacity-50"
        >
          Calculate BMI
        </button>
      </form>
      <div v-if="bmi" class="mt-6">
        <h2 class="text-xl font-bold text-center">Your BMI is: {{ bmi }}</h2>
        <p class="text-center mt-2">{{ bmiStatus }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      weight: null,
      height: null,
      bmi: null
    }
  },
  computed: {
    bmiStatus() {
      if (!this.bmi) return ''
      if (this.bmi < 18.5) return 'Underweight'
      if (this.bmi < 24.9) return 'Normal weight'
      if (this.bmi < 29.9) return 'Overweight'
      return 'Obesity'
    }
  },
  methods: {
    calculateBMI() {
      const heightInMeters = this.height / 100
      this.bmi = (this.weight / heightInMeters ** 2).toFixed(1)
    },
    reloadPage() {
      window.location.reload()
    }
  }
}
</script>

<style scoped>
body {
  @apply bg-gray-100;
}
</style>
