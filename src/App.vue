<template>
  <div>
    <router-view/>
  </div>
</template>

<style lang="scss">
// @import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@300;600&display=swap');

@import url('https://fonts.googleapis.com/css2?family=Arvo&family=Montserrat:wght@300;600&display=swap');


body {
  color: white;
  min-width: 100vw;
  min-height: 100vh;
  margin: 0;
  padding: 0;
  font-family: 'Montserrat', sans-serif;
  background-repeat: no-repeat;
  overflow-x: hidden;

  ::selection {
    background-color: rgba(255,255,255,0.6);
  } 
}
body, div, html, table, button, span, td, th, tbody {
  box-sizing: border-box;
}

.bold {
  font-weight: 600;
}
</style>
