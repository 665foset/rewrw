<template>
  <header class="app-header">
    <div class="container header-content">
      <h1 class="site-title">Smart AC Controller</h1>
    </div>
  </header>
</template>

<script setup>
</script>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Nunito:ital,wght@0,300;0,400;0,600;0,700;1,300;1,400&display=swap");

.app-header {
  background: linear-gradient(to right, #1a202c, #2d3748);
  color: #edf2f7;
  padding: 1.5rem 0;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.25);
  position: sticky;
  top: 0;
  z-index: 1000;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.site-title {
  font-family: 'Poppins', sans-serif;
  font-size: 2.5rem;
  font-weight: 700;
  letter-spacing: -0.04em;
  margin: 0;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
  margin-left: 2rem;
}

.nav-list {
  display: flex;
  list-style: none;
  padding: 0;
  margin: 0;
  gap: 2rem;
}

.nav-link {
  font-family: 'Nunito', sans-serif;
  color: #cbd5e0;
  text-decoration: none;
  font-size: 1.15rem;
  font-weight: 500;
  padding: 0.5rem 0.8rem;
  border-radius: 0.375rem;
  transition: all 0.3s ease-in-out;
}

.nav-link:hover {
  color: #a0aec0;
  background-color: rgba(255, 255, 255, 0.1);
  transform: translateY(-2px);
}

@media (max-width: 768px) {
  .app-header {
    padding: 1.25rem 0;
  }
  .header-content {
    flex-direction: column;
    text-align: center;
  }

  .nav-list {
    margin-top: 1.25rem;
    gap: 1.25rem;
    flex-wrap: wrap;
    justify-content: center;
  }

  .site-title {
    font-size: 2rem;
    margin-left: 0;
  }

  .nav-link {
    padding: 0.4rem 0.7rem;
    font-size: 1rem;
  }
}
</style>
