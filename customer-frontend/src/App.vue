<template>
  <div id="app">
    <nav class="navbar navbar-expand-lg navbar-dark bg-dark mb-5">
      <div class="container-fluid">
        <router-link to="/" class="navbar-brand">Customer Manager</router-link>
        <div class="collapse navbar-collapse">
          <div class="navbar-nav">
            <router-link to="/customers" class="nav-link">Customer List</router-link>
          </div>
        </div>
      </div>
    </nav>
    <div class="container">
      <router-view />
    </div>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>
