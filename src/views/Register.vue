<template>
  <div class="flex items-center justify-center min-h-screen bg-gray-100">
    <div class="w-full max-w-lg p-8 bg-white shadow-md rounded-lg">
      <h1 class="text-2xl font-bold mb-6 text-center">Register</h1>

      <form @submit.prevent="register">
        <!-- <div v-if="loading" class="flex items-center justify-center mb-4">
          <svg class="animate-spin h-5 w-5 text-gray-600" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8v2a6 6 0 00-6 6h-2zm4.293-3.293a1 1 0 00-1.414 1.414L9.586 12l-2.707 2.707a1 1 0 001.414 1.414L12 13.414l2.707 2.707a1 1 0 001.414-1.414L14.414 12l2.707-2.707a1 1 0 00-1.414-1.414L12 10.586 9.293 7.293z"></path>
          </svg>
        </div> -->

        <div>
          <div class="mb-4">
            <label for="name" class="block text-sm font-medium text-gray-700">Name:</label>
            <input type="text" v-model="name" id="name" required
              class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-gray-500 focus:border-gray-500 sm:text-sm" />
          </div>
          <div class="mb-4">
            <label for="email" class="block text-sm font-medium text-gray-700">Email:</label>
            <input type="email" v-model="email" id="email" required
              class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-gray-500 focus:border-gray-500 sm:text-sm" />
          </div>
          <div class="mb-4">
            <label for="password" class="block text-sm font-medium text-gray-700">Password:</label>
            <input type="password" v-model="password" id="password" required
              class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-gray-500 focus:border-gray-500 sm:text-sm" />
          </div>   
          <div class="mb-4">        
              <label for="confirm-password" class="block text-sm font-medium text-gray-700">Confirm Password</label>
              <input
                id="confirm-password"
                v-model="confirmPassword"
                type="password"
                required
                class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-blue-500 focus:border-blue-500 sm:text-sm"
              />
          </div>      
          <router-link to="/login" class="font-semibold text-sm underline">Click to Login</router-link>
          <button type="submit" :disabled="loading"
            class="w-full bg-gray-600 text-white py-2 px-4 rounded-md shadow-sm mt-3 hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-500">
            <span v-if="loading" class="loader"></span>
            <span v-else>Register</span>
          </button>
          <p v-if="error" class="mt-4 text-red-500 text-center">{{ error }}</p>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      name: '',
      email: '',
      password: '',
      confirmPassword: '',
      error: null,
      loading: false
    };
  },
  methods: {   
    async register() {
      this.loading = true;

      if (this.password !== this.confirmPassword) {
        this.error = 'Passwords do not match';
        this.loading = false;
        return;
      }

      try {
        const response = await fetch('https://fileorganise.onrender.com/api/auth/register', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({ 
            name: this.name,    
            email: this.email, 
            password: this.password 
          })
        });
        if (response.ok) {
          const data = await response.json();
          localStorage.setItem('token', data.token);
          this.$router.push('/dashboard');  
        } else {
          this.error = 'User with this email already exists';
          this.loading = false;
          return;
        }
      } catch (error) {
        this.error = error.message;
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>

<style scoped>
.loader {
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top: 2px solid #fff;
  width: 20px;
  height: 20px;
  animation: spin 1s linear infinite;
  display: inline-block;
  margin-right: 10px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
