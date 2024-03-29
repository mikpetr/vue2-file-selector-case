<template>
  <div class="FilesExplorer">
    <div class="header">
      <div
        v-if="currentFolder && currentFolder.parentFolder"
        @click="goBack"
        role="button"
        class="button back"
      >
        <a-icon type="arrow-left" />
      </div>
      <div class="title">{{currentFolder && currentFolder.name}}</div>
      <div @click="close" role="button" class="button close">
        <a-icon type="close" />
      </div>
    </div>
    <div v-if="!currentFolder" class="loading">
      <a-icon type="loading" />
    </div>
    <FilesExplorerFilesList
      v-else
      :currentFolder="currentFolder"
      :selectedFiles="selectedFiles"
      @enterFolder="enterFolder"
      @toggleFile="toggleFile"
    />
    <div class="controls">
      <a-button @click="submitFiles" type="primary" :disabled="!selectedFilesCount">
        Select {{selectedFilesCount || ''}} files
      </a-button>
    </div>
  </div>
</template>

<script>
// UNIT TEST: check if there is no back button when there is no parrent folder
// UNIT TEST: check if there is back button when there is parrent folder
// UNIT TEST: check if loading appears while loading folders strcuture
// UNIT TEST: check that we don't show the list while loading folders strcuture
// UNIT TEST: check that currentFolder is getting updated when "enterFolder" event
//            is triggered on FilesExplorerFilesList component
// UNIT TEST: check that file is added in selectedFiles object when "toggleFile" event
//            is triggered on FilesExplorerFilesList component
// UNIT TEST: check that file is removed from selectedFiles object when "toggleFile" event
//            is triggered on FilesExplorerFilesList component
// UNIT TEST: check that "Select files" button is disabled when there is no any file selected
// UNIT TEST: check that text on submit button is "Select files" when there is no files selected
// UNIT TEST: check that there is a right number of selected files on "Select X fiels"
//            button when some files are selected

import folderStructureApi from '@/api/folderStructureApi';
import FilesExplorerFilesList from './FilesExplorerFilesList.vue';

export default {
  name: 'FilesExplorer',
  props: {
    value: Array,
  },
  components: {
    FilesExplorerFilesList,
  },
  data() {
    return {
      selectedFiles: {},
      currentFolder: null,
    };
  },
  computed: {
    selectedFilesCount() {
      return Object.keys(this.selectedFiles).length;
    },
  },
  created() {
    if (this.value) {
      this.value.forEach((file) => {
        this.$set(this.selectedFiles, file.id, file);
      });
    }
  },
  async mounted() {
    const { data } = await folderStructureApi.get();

    this.enterFolder(data);
  },
  methods: {
    enterFolder(folder) {
      this.currentFolder = {
        parentFolder: this.currentFolder,
        ...folder,
      };
    },
    goBack() {
      this.currentFolder = this.currentFolder.parentFolder;
    },
    fileIsSelected(fileId) {
      return !!this.selectedFiles[fileId];
    },
    toggleFile(file) {
      if (this.fileIsSelected(file.id)) {
        this.$delete(this.selectedFiles, file.id);
      } else {
        this.$set(this.selectedFiles, file.id, file);
      }
    },
    submitFiles() {
      const filesArray = Object.keys(this.selectedFiles).map(
        (fileId) => this.selectedFiles[fileId],
      );

      this.$emit('input', filesArray);
      this.$emit('close');
    },
    close() {
      this.$emit('input', []);
      this.$emit('close');
    },
  },
};
</script>

<style lang="scss" scoped>
.FilesExplorer {
  width: 420px;
  background-color: white;
  box-shadow: 0px 4px 8px rgba(0, 0, 0, 0.08), 0px 8px 24px rgba(0, 0, 0, 0.12);
  border-radius: 16px;
  padding: 8px;
  box-sizing: border-box;

  .loading {
    font-size: 30px;
    text-align: center;
    padding: 35px 0;
  }

  .header {
    display: flex;
    justify-content: space-between;
    width: 100%;
    margin-bottom: 8px;

    .title {
      font-size: 20px;
      line-height: 44px;
      font-weight: 500;
      flex-basis: calc(100% - 88px);
      text-align: left;
      margin-left: 14px;
    }

    .button {
      display: block;
      width: 44px;
      height: 44px;
      border-radius: 8px;
      line-height: 44px;
      text-align: center;

      &.back {
        margin-right: -14px;
      }
    }
  }

  .controls {
    text-align: right;
    margin-bottom: 8px;
    margin-right: 8px;
  }
}
</style>
