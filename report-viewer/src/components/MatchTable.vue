<!--
  Table which contains all of the matches for a comparison with navigation links.
-->
<template>
  <div class="match-table-container">
    <table>
      <tr>
        <th>Submission 1</th>
        <th>Submission 2</th>
      </tr>
      <tr>
        <th>File 1</th>
        <th>File 2</th>
        <th>Tokens</th>
      </tr>
      <tr v-for="( match, index ) in matches"
          :key="String(index).concat(match.startInFirst).concat(match.startInSecond)"
          :style="{ background : match.color }"
          @click="$emit('matchSelected', $event, match)">
        <td>
          <div class="td-content">
            <p>{{ match.firstFile }}</p>
            <p>({{ match.startInFirst }} - {{ match.endInFirst }})</p>
          </div>
        </td>
        <td>
          <div class="td-content">
            <p>{{ match.secondFile }}</p>
            <p>({{ match.startInSecond }} - {{ match.endInSecond }})</p>
          </div>
        </td>
        <td>{{ match.tokens }}</td>
      </tr>
    </table>
  </div>
</template>

<script>
import {defineComponent} from "vue";

export default defineComponent({
  name: "MatchTable",
  props: {
    /**
     * Matches of the comparison.
     * type: Array<Match>
     */
    matches: {
      type: Array
    },
    /**
     * ID of first submission
     */
    id1: {
      type: String
    },
    /**
     * ID of second submission
     */
    id2: {
      type: String
    }
  },
  setup() {
    return {}
  }
})
</script>

<style scoped>
.match-table-container {
  width: 100%;
  margin-top: 2%;
  padding: 1%;
  background: var(--primary-color-dark);
  border-radius: 10px;
  overflow: auto;
}

.td-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  flex-wrap: nowrap;
  padding: 5%;
}

.td-content > p {
  margin: 0;
}

table {
  border-collapse: collapse;
  width: 100%;
}

th {
  color: var(--on-primary-color);
  text-align: center;

}

td {
  font-size: small;
  text-align: center;
  padding: 2%;

}
</style>