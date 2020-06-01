<template>
  <div>
    <router-view/>
  </div>
</template>

<style lang="scss">
// @import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@300;600&display=swap');

@import url('https://fonts.googleapis.com/css2?family=Arvo&family=Montserrat:wght@300;600&display=swap');


body {
  background: #FC5C7D;  /* fallback for old browsers */
  background: -webkit-linear-gradient(to bottom right, #6A82FB, #FC5C7D);  /* Chrome 10-25, Safari 5.1-6 */
  background: linear-gradient(to bottom right, #6A82FB, #FC5C7D); /* W3C, IE 10+/ Edge, Firefox 16+, Chrome 26+, Opera 12+, Safari 7+ */
  color: white;
  min-width: 100vw;
  min-height: 100vh;
  margin: 0;
  padding: 0;
  font-family: 'Montserrat', sans-serif;
  background-repeat: no-repeat;
  overflow-x: hidden;
}
body, div, html, table, button, span, td, th, tbody {
  box-sizing: border-box;
}

.bold {
  font-weight: 600;
}
</style>
