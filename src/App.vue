<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center p-4">
    <div class="bg-white shadow-md rounded-lg p-6 w-full max-w-lg">
      <h1 class="text-2xl font-bold mb-4 text-center cursor-pointer" @click="reloadPage">
        BMI Calculator
      </h1>
      <form @submit.prevent="calculateBMI">
        <div class="mb-4">
          <label for="unit" class="block text-sm font-medium text-gray-700">Select Unit</label>
          <select
            v-model="unit"
            class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
          >
            <option value="metric">Metric (kg/cm)</option>
            <option value="imperial">Imperial (lbs/in)</option>
          </select>
        </div>
        <div class="mb-4">
          <label for="weight" class="block text-sm font-medium text-gray-700">
            Weight ({{ unit === 'metric' ? 'kg' : 'lbs' }})
          </label>
          <input
            v-model.number="weight"
            type="number"
            id="weight"
            class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            required
            @input="validateWeight"
          />
          <span v-if="weightError" class="text-red-500 text-xs">{{ weightError }}</span>
        </div>
        <div class="mb-4">
          <label for="height" class="block text-sm font-medium text-gray-700">
            Height ({{ unit === 'metric' ? 'cm' : 'in' }})
          </label>
          <input
            v-model.number="height"
            type="number"
            id="height"
            class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
            required
            @input="validateHeight"
          />
          <span v-if="heightError" class="text-red-500 text-xs">{{ heightError }}</span>
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
        <p class="text-center mt-2 text-sm text-gray-600">{{ healthTips }}</p>
        <div class="mt-4">
          <h3 class="text-lg font-bold">Diet Recommendations:</h3>
          <p class="text-sm text-gray-600">{{ dietRecommendations }}</p>
          <h3 class="text-lg font-bold mt-2">Exercise Recommendations:</h3>
          <p class="text-sm text-gray-600">{{ exerciseRecommendations }}</p>
        </div>
      </div>
      <h3 class="text-lg font-bold mt-6">BMI History</h3>
      <ul class="mt-2">
        <li v-for="(entry, index) in bmiHistory" :key="index" class="text-sm text-gray-700">
          BMI: {{ entry.bmi }} - Date: {{ entry.date }}
        </li>
      </ul>
      <button
        v-if="bmiHistory.length"
        @click="clearHistory"
        class="mt-4 w-full bg-red-600 text-white py-2 rounded-md hover:bg-red-700 focus:ring-2 focus:ring-red-500 focus:ring-opacity-50"
      >
        Clear History
      </button>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      weight: null,
      height: null,
      bmi: null,
      bmiHistory: [],
      unit: 'metric',
      weightError: '',
      heightError: ''
    }
  },
  computed: {
    bmiStatus() {
      if (!this.bmi) return ''
      if (this.bmi < 16) return 'Severely Underweight'
      if (this.bmi < 16.9) return 'Underweight'
      if (this.bmi < 18.4) return 'Mildly Underweight'
      if (this.bmi < 24.9) return 'Normal weight'
      if (this.bmi < 29.9) return 'Overweight'
      if (this.bmi < 34.9) return 'Obesity Class I'
      if (this.bmi < 39.9) return 'Obesity Class II'
      return 'Obesity Class III'
    },
    healthTips() {
      if (!this.bmi) return ''
      if (this.bmi < 18.5)
        return 'Consider consulting a healthcare provider for guidance on healthy weight gain.'
      if (this.bmi < 24.9)
        return 'Keep up the good work! Maintain a balanced diet and regular exercise.'
      if (this.bmi < 29.9)
        return 'Consider a balanced diet and exercise to maintain a healthy weight.'
      return 'Consult a healthcare provider for guidance on weight management.'
    },
    dietRecommendations() {
      if (!this.bmi) return ''
      if (this.bmi < 18.5)
        return 'Include more nutrient-dense foods like nuts, avocados, and whole grains.'
      if (this.bmi < 24.9)
        return 'Continue with a balanced diet rich in fruits, vegetables, whole grains, and lean proteins.'
      if (this.bmi < 29.9)
        return 'Focus on portion control and reducing sugar and saturated fat intake.'
      return 'Incorporate more fruits, vegetables, and whole grains while reducing processed foods.'
    },
    exerciseRecommendations() {
      if (!this.bmi) return ''
      if (this.bmi < 18.5) return 'Engage in moderate-strength training a few times a week.'
      if (this.bmi < 24.9)
        return 'Maintain a routine of at least 150 minutes of moderate aerobic activity per week.'
      if (this.bmi < 29.9) return 'Aim for 150-300 minutes of moderate-intensity exercise per week.'
      return 'Focus on a combination of cardio and strength training for at least 300 minutes a week.'
    }
  },
  methods: {
    calculateBMI() {
      let heightInMeters = this.unit === 'metric' ? this.height / 100 : this.height * 0.0254
      let weightInKg = this.unit === 'metric' ? this.weight : this.weight * 0.453592

      this.bmi = (weightInKg / heightInMeters ** 2).toFixed(1)
      this.saveHistory(this.bmi)
    },
    saveHistory(bmi) {
      const entry = {
        bmi,
        date: new Date().toLocaleString()
      }
      this.bmiHistory.push(entry)
      localStorage.setItem('bmiHistory', JSON.stringify(this.bmiHistory))
    },
    clearHistory() {
      this.bmiHistory = []
      localStorage.removeItem('bmiHistory')
    },
    validateWeight() {
      this.weightError = this.weight <= 0 ? 'Weight must be greater than zero.' : ''
    },
    validateHeight() {
      this.heightError = this.height <= 0 ? 'Height must be greater than zero.' : ''
    },
    reloadPage() {
      window.location.reload()
    }
  },
  mounted() {
    const history = JSON.parse(localStorage.getItem('bmiHistory')) || []
    this.bmiHistory = history
  }
}
</script>

<style scoped>
body {
  @apply bg-gray-100;
}
</style>
