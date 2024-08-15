<template>
    <div class="flex items-center justify-center min-h-screen bg-gray-100">
      <div class="w-full max-w-sm p-8 bg-white shadow-md rounded-lg">
        <h1 class="text-2xl font-bold mb-6 text-center">Login</h1>
        <form @submit.prevent="login">
          <div class="mb-4">
            <label for="email" class="block text-sm font-medium text-gray-700">Email:</label>
            <input type="email" v-model="email" id="email" required
              class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-gray-500 focus:border-gray-500 sm:text-sm" />
          </div>
          <div class="mb-6">
            <label for="password" class="block text-sm font-medium text-gray-700">Password:</label>
            <input type="password" v-model="password" id="password" required
              class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-gray-500 focus:border-gray-500 sm:text-sm" />
          </div>
           <router-link to="/register" class="font-semibold text-sm underline">Click to Register</router-link>
          <button type="submit" :disabled="isLoading"
            class="w-full bg-gray-600 text-white py-2 px-4 rounded-md shadow-sm mt-3 hover:bg-gray-700 focus:outline-none focus:ring-2 focus:ring-gray-500">
            <span v-if="isLoading" class="loader"></span>
            <span v-else>Login</span>
          </button>
          <p v-if="error" class="mt-4 text-red-500 text-center">{{ error }}</p>
        </form>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        email: '',
        password: '',
        error: null,
        isLoading: false
      };
    },
    methods: {
      async login() {
        this.isLoading = true;
        this.error = null;
        try {
          const response = await fetch('https://fileorganise.onrender.com/api/auth/login', {
            method: 'POST',
            headers: {
              'Content-Type': 'application/json'
            },
            body: JSON.stringify({ email: this.email, password: this.password })
          });
  
          const data = await response.json();
  
          if (!response.ok) {
            throw new Error(data.message || 'Login failed');
          }
  
          localStorage.setItem('token', data.token);
          this.$router.push('/dashboard');  
        } catch (error) {
          this.error = error.message;
        } finally {
          this.isLoading = false;
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
  