<template>

  <table style="border-collapse: collapse; border-spacing: 0;" cellspacing="0" cellpadding="0">
    <tr v-for="(list, ykey) in crossword" v-bind:key="'y' + ykey">
      <td v-for="(element, xkey) in list" v-bind:key="'x' + xkey">
        <WordBox v-on:PositionFromBox="this.sendPositionToPlayView($event)" v-bind:xkey="xkey" v-bind:ykey="ykey"
          v-bind:letter="element.letter" v-bind:direction="element.direction" v-bind:inHorizontal="element.inHorizontal"
          v-bind:inVertical="element.inVertical" v-bind:isFirstLetter="element.isFirstLetter"
          v-bind:wordInOrder="element.wordInOrder" v-bind:isOccupied="element.isOccupied"
          v-bind:sourceName="this.sourceName" v-bind:cellsAmount="this.cellsAmount">
        </WordBox>
      </td>
    </tr>
  </table>

</template>



<script>

import WordBox from '../components/WordBox.vue'

export default {
  data: function () {
    return {
      dims: String(40 / this.cellsAmount) + "vw"
    }
  },
  name: 'CrossWord',
  components: {
    WordBox,
  },
  props: {
    crossword: Array,
    cellsAmount: Number,
    solutionsList: Object,
    sourceName: String,
    wordInOrder: Number
  },
  watcher: {
    crossword: {

    }
  },

  mounted: function () {
    document.querySelector(':root').style.setProperty('--dims', this.dims)
  },
  methods: {

    sendPositionToPlayView: function (event) {
      this.$emit("sendPosition", event)
    }
  },
  watch: {
    cellsAmount: {
      handler: function (newValue, oldValue) {
        this.dims = String(40 / newValue - (40 / this.cellsAmount) / 35) + "vw"
        document.querySelector(':root').style.setProperty('--dims', this.dims)

        console.log(oldValue)
      },
      deep: true
    }
  }
}
</script>


<style scoped>
:root {
  --dims: 1em;
  --dimsX: 1em;
  --dimsY: 1em;
}

#crosswordWrapper {

  height: 50vw;
  width: 50vw;
  margin: 0 auto;
  background-color: #A7CAB1;

}

table {
  table-layout: auto;
  height: 40vw;
  width: 40vw;
  border-spacing: 0;
}

tr {
  height: var(--dims);
}

td {
  width: var(--dims);
}
</style>

