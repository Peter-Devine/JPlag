<!--
  Table which display all of the comparisons with their participating ids and similarity percentage for the selected metric.
-->
<template>
  <table>
    <tr class="head-row">
      <th>No.</th>
      <th>Submission 1</th>
      <th></th>
      <th>Submission 2</th>
      <th>Match %</th>
    </tr>
    <tr v-for="(comparison, index) in topComparisons"
        :key="comparison.firstSubmissionId
            + comparison.secondSubmissionId
            + comparison.matchPercentage"
        class="selectable"
    >
      <td @click="navigateToComparisonView(comparison.firstSubmissionId, comparison.secondSubmissionId)">{{
          index + 1
        }}.
      </td>
      <td :class="{ 'anonymous-style' : isAnonymous(comparison.firstSubmissionId)}"
          @click="navigateToComparisonView(comparison.firstSubmissionId, comparison.secondSubmissionId)">
        {{ isAnonymous(comparison.firstSubmissionId) ? "Hidden" : comparison.firstSubmissionId }}
      </td>
      <td
          @click="navigateToComparisonView(comparison.firstSubmissionId, comparison.secondSubmissionId)">
        <img alt=">>" src="@/assets/double_arrow_black_18dp.svg"/>
      </td>
      <td :class="{ 'anonymous-style' : isAnonymous(comparison.secondSubmissionId)}"
          @click="navigateToComparisonView(comparison.firstSubmissionId, comparison.secondSubmissionId)">
        {{ isAnonymous(comparison.secondSubmissionId) ? "Hidden" : comparison.secondSubmissionId }}
      </td>
      <td>{{ formattedMatchPercentage(comparison.matchPercentage) }}</td>
      <td>
        <img v-if="isInCluster(comparison.firstSubmissionId, comparison.secondSubmissionId)"
             alt=">>"
             src="@/assets/keyboard_double_arrow_down_black_18dp.svg"
             @click="toggleDialog(index)"
        />
      </td>
      <GDialog v-model="dialog[index]">
        <ClustersList :clusters="getClustersFor(comparison.firstSubmissionId, comparison.secondSubmissionId)"
                      :comparison="comparison"/>
      </GDialog>
    </tr>
  </table>
</template>

<script>
import {defineComponent, ref} from "vue";
import router from "@/router";
import store from "@/store/store";
import {GDialog} from "gitart-vue-dialog";
import ClustersList from "@/components/ClustersList";

export default defineComponent({
  name: "ComparisonsTable",
  components: {ClustersList, GDialog},
  props: {
    topComparisons: {
      type: Array,
      required: true
    },
    clusters: {
      type: Array
    }
  },
  setup(props) {
    let formattedMatchPercentage = (number) => number.toFixed(2)
    const dialog = ref([])
    props.topComparisons.forEach(() => dialog.value.push(false))

    const toggleDialog = (index) => {
      dialog.value[index] = true
    }

    const navigateToComparisonView = (firstId, secondId) => {
      if (!store.state.single) {
        router.push({
          name: "ComparisonView",
          query: {firstId: firstId, secondId: secondId},
        })
      }
    }

    const isInCluster = (id1, id2) => {
      return props.clusters.some(c => c.members.includes(id1) && c.members.includes(id2))
    }

    const isAnonymous = (id) => {
      return store.state.anonymous.has(id)
    }


    const getParticipatingMatchesForId = (id, others) => {
      let matches = []
      props.topComparisons.forEach(comparison => {
        if (comparison.firstSubmissionId.includes(id) && others.includes(comparison.secondSubmissionId)) {
          matches.push({matchedWith: comparison.secondSubmissionId, percentage: comparison.matchPercentage})
        } else if (comparison.secondSubmissionId.includes(id) && others.includes(comparison.firstSubmissionId)) {
          matches.push({matchedWith: comparison.firstSubmissionId, percentage: comparison.matchPercentage})
        }
      })
      return matches;
    }

    const clustersWithParticipatingMatches = props.clusters.map(cluster => {
      let membersArray = {}
      cluster.members.forEach(member => {
        let others = cluster.members.filter(m => !m.includes(member))
        membersArray[member] = getParticipatingMatchesForId(member, others)
      })

      return {
        averageSimilarity: cluster.averageSimilarity,
        strength: cluster.strength,
        members: membersArray
      }
    })

    const getClustersFor = (id1, id2) => {
      return clustersWithParticipatingMatches.filter(c => Object.keys(c.members).includes(id1)
          && Object.keys(c.members).includes(id2))
    }

    return {
      clustersWithParticipatingMatches,
      dialog,

      isAnonymous,
      getClustersFor,
      toggleDialog,
      formattedMatchPercentage,
      navigateToComparisonView,
      isInCluster
    }
  }
})
</script>

<style scoped>
table {
  border-collapse: collapse;
  font-size: larger;
  text-align: center;
}

tr:nth-child(odd) {
  background: var(--primary-color-light);
}

tr:nth-child(even) {
  background: var(--secondary-color);
}

th {
  margin: 0;
  padding-top: 2%;
  padding-bottom: 2%;
  color: var(--on-primary-color);
}

td {
  padding-top: 3%;
  padding-bottom: 3%;
}

.anonymous-style {
  color: #777777;
  filter: blur(1px);
}

.head-row {
  background: var(--primary-color-light);
}

.selectable:hover {
  background: var(--primary-color-dark);
  cursor: pointer;
}
</style>