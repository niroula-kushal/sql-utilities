<script lang="ts">
  import '../app.css';
</script>

<div class="min-h-screen bg-gradient-to-br from-gray-50 to-gray-100">
  <!-- Navigation Header -->
  <header class="bg-white shadow-sm border-b border-gray-200">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
      <div class="flex justify-between items-center h-16">
        <div class="flex items-center space-x-4">
          <div class="flex items-center">
            <svg class="w-8 h-8 text-primary-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 7v10c0 2.21 3.582 4 8 4s8-1.79 8-4V7M4 7c0 2.21 3.582 4 8 4s8-1.79 8-4M4 7c0-2.21 3.582-4 8-4s8 1.79 8 4"></path>
            </svg>
            <h1 class="ml-2 text-xl font-bold text-gray-900">SQL Helper</h1>
          </div>
        </div>
        
        <!-- Navigation Menu -->
        <nav class="hidden md:flex space-x-8">
          <a href="/" class="text-gray-700 hover:text-primary-600 px-3 py-2 rounded-md text-sm font-medium transition-colors">
            IN Helper
          </a>
          <span class="text-gray-400 px-3 py-2 rounded-md text-sm font-medium cursor-not-allowed">
            More Tools Coming Soon
          </span>
        </nav>
        
        <!-- Mobile menu button -->
        <div class="md:hidden">
          <button type="button" class="text-gray-500 hover:text-gray-700 focus:outline-none focus:ring-2 focus:ring-inset focus:ring-primary-500" aria-label="Open mobile menu">
            <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path>
            </svg>
          </button>
        </div>
      </div>
    </div>
  </header>

  <!-- Main Content -->
  <main class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
    <slot />
  </main>

  <!-- Footer -->
  <footer class="bg-white border-t border-gray-200 mt-16">
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-8">
      <div class="text-center text-gray-500 text-sm">
        <p>SQL Helper - Making SQL queries easier, one tool at a time</p>
        <p class="mt-2">Built with SvelteKit & Tailwind CSS</p>
      </div>
    </div>
  </footer>
</div>