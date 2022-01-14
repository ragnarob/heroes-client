<template>
  <div>
    <router-view/>
  </div>
</template>

<style lang="scss">
@import url('');

body {
  color: white;
  min-width: 100vw;
  min-height: 100vh;
  margin: 0;
  padding: 0;
  font-family: 'Montserrat', sans-serif;
  background-repeat: no-repeat;
  overflow-x: hidden;
  font-weight: 300;

  ::selection {
    background-color: rgba(255,255,255,0.6);
  } 
}

.bold {
  font-weight: 600;
}
</style>
