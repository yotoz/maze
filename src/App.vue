<template>
  <div id="app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto&family=Sunflower:wght@300;500&display=swap');
html {
  touch-action: pan-y;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}

.title{
  font-family: 'Sunflower', sans-serif;
  font-weight: 500;
  font-size: 2rem;
  margin: 0 0 20px;
  color: #ff7b79;
}

.logo{
  width: 200px;
}
</style>
