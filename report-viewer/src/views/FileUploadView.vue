<!--
  Starting view of the application. Presents the options for loading a JPlag report.
-->
<template>
  <div class="container" @dragover.prevent @drop.prevent>
    <img alt="JPlag" src="@/assets/logo-nobg.png"/>
    <h1>JPlag Report Viewer</h1>
    <h2>Select an overview or comparison file or a zip to display.</h2>
    <div class="drop-container" @drop="uploadFile">
      <p> Drop a .json or .zip file here</p>
    </div>
    <div v-if="hasLocalFile" class="local-files-container">
      <p class="local-files-text">Detected local files!</p>
      <button class="local-files-button" @click="continueWithLocal">Continue with local files</button>
    </div>
  </div>
</template>

<script>
import {defineComponent} from "vue";
import jszip from "jszip"
import router from "@/router";
import store from "@/store/store";
import {getFileExtension} from "@/utils/Utils";

export default defineComponent({
  name: "FileUploadView",
  setup() {
    let hasLocalFile;
    //Tries to detect local file. If no files detected, hides local mode from screen.
    try {
      require("../files/overview.json")
      hasLocalFile = true
    } catch (e) {
      console.log(e)
      hasLocalFile = false
    }


    const navigateToOverview = () => {
      router.push({
            name: "OverviewView",
          }
      )
    }

    const navigateToComparisonView = (id1, id2) => {
      router.push({
        name: "ComparisonView",
        query: {
          id1: id1,
          id2: id2
        }
      })
    }
    /**
     * Handles zip file on drop. It extracts the zip and saves each file in the store.
     * @param file
     */
    const handleZipFile = (file) => {
      jszip.loadAsync(file).then(async (zip) => {
        for (const fileName of Object.keys(zip.files)) {
          console.log(fileName)
          await zip.files[fileName].async("string").then(data => {
                store.commit("saveFile", {fileName: fileName, data: data})
              }
          )
        }
        store.commit("setLoadingType", {local: false, zip: true, single: false, fileString: ""})
        navigateToOverview()
      })
    }
    /**
     * Handles a json file on drop. It read the file and passes the file string to next window.
     * @param str
     */
    const handleJsonFile = (str) => {
      let json = JSON.parse(str)
      if (json["submission_folder_path"]) {
        store.commit("setLoadingType", {local: false, zip: false, single: true, fileString: str})
        navigateToOverview()
      } else if (json["first_submission_id"] && json["second_submission_id"]) {
        store.commit("setLoadingType", {local: false, zip: false, single: true, fileString: str})
        navigateToComparisonView(json["first_submission_id"], json["second_submission_id"])
      }
    }
    /**
     * Handles file drop.
     * @param e
     */
    const uploadFile = (e) => {
      let dropped = e.dataTransfer.files
      if (!dropped) return
      let files = [...dropped]
      if (files.length > 1 || files.length === 0) return
      let read = new FileReader()
      read.onload = (e) => {
        let extension = getFileExtension(files[0])
        if (extension === "zip") {
          handleZipFile(files[0])
        } else if (extension === "json") {
          handleJsonFile(e.target.result)
        }
      }
      read.readAsText(files[0])
    }
    /**
     * Handles click on Continue with local files.
     */
    const continueWithLocal = () => {
      store.commit("setLoadingType", {local: true, zip: false, single: false, fileString: ""})
      navigateToOverview()
    }
    return {
      continueWithLocal,
      uploadFile,
      hasLocalFile
    }
  }
})
</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
  background: var(--primary-color-light);
}

.drop-container {
  background: lightgrey;
  border: dashed dodgerblue;
  align-items: center;
  justify-content: space-around;
  border-radius: 10px;
  padding: 2%;
}

.drop-container > p {
  color: dodgerblue;
  text-align: center;
}

.local-files-container {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.local-files-text {
  font-weight: bold;
  color: var(--on-background-color);
}

.local-files-button {
  font-size: large;
  background: var(--primary-color);
  color: var(--on-primary-color);
  font-weight: bold;
  padding: 5%;
  border: none;
  border-radius: 10px;
  box-shadow: var(--shadow-color) 2px 3px 3px;
}

.local-files-button:hover {
  cursor: pointer;
  background: var(--primary-color-dark);
}

input {
  display: none;
}

label {
  font-weight: bold;
  font-size: larger;
  background: #ECECEC;
  border-radius: 10px;
  box-shadow: #777777 2px 3px 3px;
  padding: 2%;
  margin-top: 1%;
}

</style>