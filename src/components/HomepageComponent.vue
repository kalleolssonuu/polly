<template>
  <button id="buttonStyle" @click="$router.push('/' + lang)">
    &#x1F3E0;
    <text id="homeText">{{ uiLabels.HomepageButton }}</text>
  </button>
</template>


<script>

export default {
  name: 'HomepageComponent',
  props: {
    HomepageButton: Object,
    uiLabels: Object,
    lang: String
  },
}



</script>


<style>
#buttonStyle {
  background-color: #43918a;
  text-align: center;
  font-size: 1.7vw;
  cursor: pointer;
  border-width: 0ch;
  color: white;
  font-family: "Comic Sans MS", "Comic Sans";
  border-radius: 0.25rem;
  position: fixed;
  bottom: 0;
  left: 0;
  margin: 1vw;
  display: flex;
  align-items: center;
}

#selectGameButtonStyle:hover {
  opacity: 75%;
}

#homeText {
  font-size: 0.9vw;
}
</style>