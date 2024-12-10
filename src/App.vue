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
          <h3 class="text-lg font-bold mt-2">Recipe Suggestions:</h3>
          <ul class="list-disc pl-5 text-sm text-gray-600">
            <li v-for="(recipe, index) in recipeRecommendations" :key="index">{{ recipe }}</li>
          </ul>
          <h3 class="text-lg font-bold mt-2">Supplement Recommendations:</h3>
          <p class="text-sm text-gray-600">{{ supplementRecommendations }}</p>
          <h3 class="text-lg font-bold mt-2">Local Food Suggestions:</h3>
          <ul class="list-disc pl-5 text-sm text-gray-600">
            <li v-for="(food, index) in localFoodRecommendations" :key="index">{{ food }}</li>
          </ul>
          <h3 class="text-lg font-bold mt-2">Exercise Recommendations:</h3>
          <p class="text-sm text-gray-600">{{ exerciseRecommendations }}</p>
          <h3 class="text-lg font-bold mt-2">Daily Motivation:</h3>
          <p class="text-sm text-gray-600">{{ dailyMotivation }}</p>
          <h3 class="text-lg font-bold mt-2">Fitness Goals:</h3>
          <input
            v-model.number="fitnessGoal"
            type="number"
            placeholder="Set your fitness goal (in kg)"
            class="mt-1 block w-full p-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"
          />
          <button
            @click="setFitnessGoal"
            class="mt-2 w-full bg-green-600 text-white py-2 rounded-md hover:bg-green-700"
          >
            Set Goal
          </button>
          <p v-if="goalMessage" class="text-green-500 mt-2">{{ goalMessage }}</p>
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
      <button
        @click="showPolicy"
        class="mt-4 w-full bg-gray-600 text-white py-2 rounded-md hover:bg-gray-700 focus:ring-2 focus:ring-gray-500 focus:ring-opacity-50"
      >
        View Policy
      </button>
    </div>

    <!-- Modal for Policy -->
    <div
      v-if="showModal"
      class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50"
    >
      <div class="bg-white rounded-lg p-6 w-3/4 max-w-lg">
        <h2 class="text-xl font-bold mb-4">Application Policy</h2>
        <p class="text-sm text-gray-800">
          Your privacy is important to us. We are committed to protecting your personal information
          and ensuring its use in compliance with applicable laws and regulations. Please read our
          full policy on data collection, usage, and user rights.
        </p>
        <button
          @click="closePolicy"
          class="mt-4 w-full bg-indigo-600 text-white py-2 rounded-md hover:bg-indigo-700"
        >
          Close
        </button>
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
      bmi: null,
      bmiHistory: [],
      unit: 'metric',
      weightError: '',
      heightError: '',
      fitnessGoal: null,
      goalMessage: '',
      recipeRecommendations: [],
      supplementRecommendations: '',
      localFoodRecommendations: [],
      dailyMotivation: this.getDailyMotivation(),
      showModal: false // State for modal visibility
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
      this.getRecipeRecommendations()
      this.getSupplementRecommendations()
      this.getLocalFoodRecommendations()
    },
    getRecipeRecommendations() {
      if (this.bmi < 18.5) {
        this.recipeRecommendations = [
          'Avocado Toast with Whole Grain Bread',
          'Quinoa Salad with Nuts and Fruits',
          'Smoothie with Banana, Spinach, and Almond Milk'
        ]
      } else if (this.bmi < 24.9) {
        this.recipeRecommendations = [
          'Grilled Chicken Salad with Mixed Greens',
          'Baked Salmon with Vegetables',
          'Vegetable Stir-Fry with Tofu'
        ]
      } else if (this.bmi < 29.9) {
        this.recipeRecommendations = [
          'Zucchini Noodles with Pesto',
          'Chickpea Salad with Lemon Dressing',
          'Quinoa Bowl with Black Beans and Avocado'
        ]
      } else {
        this.recipeRecommendations = [
          'Cauliflower Rice Stir-Fry',
          'Grilled Vegetables with Quinoa',
          'Salad with Leafy Greens and Grilled Chicken'
        ]
      }
    },
    getSupplementRecommendations() {
      if (this.bmi < 18.5) {
        this.supplementRecommendations =
          'Consider protein powder or meal replacement shakes to gain weight healthily.'
      } else if (this.bmi < 24.9) {
        this.supplementRecommendations = 'A multivitamin may help maintain overall health.'
      } else if (this.bmi < 29.9) {
        this.supplementRecommendations = 'Consider omega-3 fatty acids and fiber supplements.'
      } else {
        this.supplementRecommendations =
          'Look into meal replacements and appetite suppressants, but consult a healthcare provider first.'
      }
    },
    getLocalFoodRecommendations() {
      if (this.bmi < 18.5) {
        this.localFoodRecommendations = [
          'Nuts and Nut Butters',
          'Whole Grain Bread',
          'Full-Fat Dairy Products'
        ]
      } else if (this.bmi < 24.9) {
        this.localFoodRecommendations = [
          'Fresh Fruits and Vegetables',
          'Lean Meats',
          'Whole Grains'
        ]
      } else if (this.bmi < 29.9) {
        this.localFoodRecommendations = ['Low-Calorie Snacks', 'High-Fiber Foods', 'Lean Proteins']
      } else {
        this.localFoodRecommendations = [
          'Green Leafy Vegetables',
          'Low-Calorie Soups',
          'Lean Protein Sources'
        ]
      }
    },
    getDailyMotivation() {
      const motivations = [
        'Stay positive, work hard, make it happen!',
        'Your only limit is you. Be brave and fearless!',
        'Push yourself because no one else is going to do it for you.',
        'Great things never come from comfort zones.',
        'Dream it. Wish it. Do it.'
      ]
      return motivations[Math.floor(Math.random() * motivations.length)]
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
    },
    setFitnessGoal() {
      this.goalMessage = this.fitnessGoal
        ? `Your fitness goal is set to ${this.fitnessGoal} kg!`
        : ''
    },
    showPolicy() {
      this.showModal = true // Show the modal
    },
    closePolicy() {
      this.showModal = false // Hide the modal
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
