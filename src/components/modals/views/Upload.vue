<template>
    <div class="modal-content fm-modal-upload">
        <div class="modal-header">
            <h5 class="modal-title">{{ lang.modal.upload.title }}</h5>
            <button type="button" class="close" aria-label="Close" v-on:click="hideModal">
                <span aria-hidden="true">&times;</span>
            </button>
        </div>
        <div class="modal-body">
            <div class="pb-3">
                <div>Upload by:</div>
                <div class="form-check form-check-inline">
                    <input class="form-check-input" type="radio" name="upload-type" id="upload-type-file" v-model="uploadType" v-bind:value="0" v-on:change="onUploadTypeChange" v-bind:disabled="uploading || progressBar > 0">
                    <label class="form-check-label" for="upload-type-file">File</label>
                </div>
                <div class="form-check form-check-inline">
                    <input class="form-check-input" type="radio" name="upload-type" id="upload-type-folder" v-model="uploadType" v-bind:value="1" v-on:change="onUploadTypeChange" v-bind:disabled="uploading || progressBar > 0">
                    <label class="form-check-label" for="upload-type-folder">Folder</label>
                </div>
            </div>
            <div class="fm-btn-wrapper" v-show="!progressBar">
                <div v-if="uploadType == 0">
                  <button type="button" class="btn btn-secondary btn-block">
                      {{ lang.btn.uploadSelect }}
                  </button>
                  <input type="file" multiple name="myfile" v-on:change="selectFiles($event)">
                </div>
                <div v-else>
                  <button type="button" class="btn btn-secondary btn-block">
                      {{ lang.btn.uploadSelectFolder }}
                  </button>
                  <input type="file" multiple directory webkitdirectory mozdirectory name="myfolder" v-on:change="selectFolder">
                </div>
            </div>
            <div class="fm-upload-list" v-if="countFiles">
              <div v-if="uploadType == 0">
                  <div class="d-flex justify-content-between"
                      v-for="(item, index) in newFiles"
                      v-bind:key="index">
                      <div class="w-75 text-truncate">
                          <i class="far" v-bind:class="mimeToIcon(item.type)"/>
                          {{ item.name }}
                      </div>
                      <div class="text-right">
                          {{ bytesToHuman(item.size) }}
                      </div>
                  </div>
                  <hr>
                  <div class="d-flex justify-content-between">
                      <div>
                          <strong>{{ lang.modal.upload.selected }}</strong>
                          {{ newFiles.length }}
                      </div>
                      <div class="text-right">
                          <strong>{{ lang.modal.upload.size }}</strong>
                          {{ allFilesSize }}
                      </div>
                  </div>
                </div>
                <div v-else>
                  <div class="d-flex justify-content-between">
                      <div class="w-75 text-truncate">
                          <i class="far" v-bind:class="mimeToIcon(null)"/>
                          {{ getRootFolder }}
                      </div>
                      <div class="text-right">
                          {{ allFilesSize }}
                      </div>
                  </div>
                  <hr>
                  <div class="d-flex justify-content-between">
                      <div>
                          <strong>{{ lang.modal.upload.selected }}</strong>
                          {{ newFiles.length }}
                      </div>
                      <div class="text-right">
                          <strong>{{ lang.modal.upload.size }}</strong>
                          {{ allFilesSize }}
                      </div>
                  </div>
                </div>
                <hr>
                <div class="d-flex justify-content-between">
                    <div>
                        <strong>{{ lang.modal.upload.ifExist }}</strong>
                    </div>
                    <div class="form-check form-check-inline">
                        <input class="form-check-input"
                               id="uploadRadio1"
                               type="radio"
                               name="uploadOptions"
                               value="0"
                               checked
                               v-model="overwrite">
                        <label class="form-check-label" for="uploadRadio1">
                            {{ lang.modal.upload.skip }}
                        </label>
                    </div>
                    <div class="form-check form-check-inline">
                        <input class="form-check-input"
                               id="uploadRadio2"
                               type="radio"
                               name="uploadOptions"
                               value="1"
                               checked
                               v-model="overwrite">
                        <label class="form-check-label" for="uploadRadio2">
                            {{ lang.modal.upload.overwrite }}
                        </label>
                    </div>
                </div>
                <hr>
            </div>
            <div v-else><p>{{ lang.modal.upload.noSelected }}</p></div>
            <div class="fm-upload-info">
                <!-- Progress Bar -->
                <div class="progress" v-show="countFiles">
                    <div class="progress-bar progress-bar-striped bg-info" role="progressbar"
                         v-bind:aria-valuenow="progressBar"
                         aria-valuemin="0"
                         aria-valuemax="100"
                         v-bind:style="{width: progressBar + '%' }">
                        {{ progressBar }}%
                    </div>
                </div>
            </div>
        </div>
        <div class="modal-footer">
            <button class="btn"
                    v-bind:class="[countFiles ? 'btn-info' : 'btn-light']"
                    v-bind:disabled="!countFiles || uploading || progressBar > 0"
                    v-on:click="uploadFiles">{{ lang.btn.submit }}
            </button>
            <button class="btn btn-light" v-on:click="hideModal()">{{ lang.btn.cancel }}</button>
        </div>
    </div>
</template>

<script>
import modal from '../mixins/modal';
import translate from '../../../mixins/translate';
import helper from '../../../mixins/helper';

export default {
  name: 'Upload',
  mixins: [modal, translate, helper],
  data() {
    return {
      // selected files
      // newFiles: [],

      // overwrite if exists
      overwrite: 0,

      // upload type (0 - file, 1 - folder)
      uploadType: 0,
    };
  },
  computed: {

    /**
     * Progress bar value - %
     * @returns {number}
     */
    progressBar() {
      return this.$store.state.fm.messages.actionProgress;
    },

    /**
     * Count of files selected for upload
     * @returns {number}
     */
    countFiles() {
      return this.newFiles.length;
    },

    /**
     * Calculate the size of all files
     * @returns {*|string}
     */
    allFilesSize() {
      let size = 0;

      for (let i = 0; i < this.newFiles.length; i += 1) {
        size += this.newFiles[i].size;
      }

      return this.bytesToHuman(size);
    },

    /*
     * Get root folder name
     */
    getRootFolder() {
      return this.newFolders.reduce((a, b) => a.length <= b.length ? a : b);
    },

    newFiles: {
        get() {
            return this.$store.getters['fm/getNewFiles'];
        },
        set(files) {
            this.$store.commit('fm/setNewFiles', files)
        }
    },

    newFolders: {
        get() {
            return this.$store.getters['fm/getNewFolders'];
        },
        set(folders) {
            this.$store.commit('fm/setNewFolders', folders)
        }
    },

    uploading: {
        get() {
            return this.$store.getters['fm/getUploading'];
        },
        set(uploading) {
            this.$store.commit('fm/setUploading', uploading)
        }
    },
  },
  methods: {
    /**
     * Select file or files
     * @param event
     */
    selectFiles(event) {
      // files selected?
      if (event.target.files.length === 0) {
        // no file selected
        this.newFiles = [];
      } else {
        // we have file or files
        this.newFiles = event.target.files;
      }
    },

    /**
     * Select folder
     * @param event
     */
    selectFolder(event) {
      // files selected?
      if (event.target.files.length === 0) {
        // no file selected
        this.newFiles = [];
      } else {
        let folders = [];
        for (let i = 0; i < event.target.files.length; i++) {
          let file = event.target.files[i];
          let folder = file.webkitRelativePath.replace(file.name,'');
          if(folder)
            folders.push(folder.substring(0, folder.length - 1));
        }

        // we have file or files
        folders = [...new Set(folders)];
        this.newFolders = folders;
        this.newFiles = event.target.files;
      }
    },

    /**
     * Upload new files
     */
    uploadFiles() {
      // if folders exists
      if (this.newFolders.length) {
        this.uploading = true;
        this.createFolders(JSON.parse(JSON.stringify(this.newFolders)));
      }
      // if files exists
      else if (this.countFiles) {
        // upload files
        this.$store.dispatch('fm/upload', {
          files: this.newFiles,
          folders: this.newFolders,
          overwrite: this.overwrite,
        }).then((response) => {
          // if upload is successful
          if (response.data.result.status === 'success') {
            // close modal window
            this.hideModal();
          }
        });
      }
    },

    createFolders(folders) {
      if(this.uploading) {
        this.$store.dispatch('fm/createDirectory', folders.shift()).then((response) => {
          if(folders.length)
            this.createFolders(folders);
          else
            this.uploadFilesWithDir();
        });
      }
    },

    uploadFilesWithDir() {
      this.uploadFilesByFolder(this.newFiles, JSON.parse(JSON.stringify(this.newFolders)), 1, this.newFolders.length);
    },

    uploadFilesByFolder(files, folders, currFolder, maxFolder) {
      if(this.uploading) {
        let folder = folders.shift();
        let uploadFiles = [];

        for (let i = 0; i < files.length; i++) {
          let file = files[i];
          let fullpath = ((file.filepath) ? file.filepath : file.webkitRelativePath);
          if(fullpath == (folder + '/' + file.name)) {
            uploadFiles.push(file);
          }
        }

        if(uploadFiles.length) {
          if(currFolder >= this.newFolders.length) {
            this.uploading = false;
          }

          this.$store.dispatch('fm/upload', {
            files: uploadFiles,
            directory: folder,
            overwrite: this.overwrite,
            currentIndex: currFolder,
            maxIndex: maxFolder
          }).then((response) => {
            // if upload is successful
            if (response.data.result.status === 'success') {
              // close modal window
              if(currFolder >= this.newFolders.length) {
                this.hideModal();
              }
              else {
                currFolder += 1;
                this.uploadFilesByFolder(files, folders, currFolder, maxFolder);
              }
            }
          });
        }
      }
    },

    /*
     * On upload type changed by user
     */
    onUploadTypeChange() {
      this.newFiles = [];
    }
  },
  watch: {
    newFolders (newVal, oldVal) {
      if(newVal.length)
        this.uploadType = 1;
      else
        this.uploadType = 0;
    }
  },
  created() {
    if(this.newFolders.length)
      this.uploadType = 1;
    else
      this.uploadType = 0;
  },
  beforeDestroy() {
    this.uploading = false;
    this.newFolders = [];
    this.newFiles = [];
  }
};
</script>

<style lang="scss">
    .fm-modal-upload {

        .fm-btn-wrapper {
            position: relative;
            overflow: hidden;
            padding-bottom: 6px;
            margin-bottom: 0.6rem;
        }

        .fm-btn-wrapper input[type=file] {
            font-size: 100px;
            position: absolute;
            left: 0;
            top: 0;
            opacity: 0;
            cursor: pointer;
        }

        .fm-upload-list .far {
            padding-right: 0.5rem;
        }

        .fm-upload-list .form-check-inline {
            margin-right: 0;
        }

        .fm-upload-info > .progress {
            margin-bottom: 1rem;
        }
    }
</style>
