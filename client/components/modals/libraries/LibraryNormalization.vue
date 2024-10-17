<template>
  <div class="w-full h-full px-1 md:px-2 py-1 mb-4">
    <div class="w-full border border-black-200 p-4 my-8">
      <div class="flex flex-wrap items-center">
        <div class="w-1/2 h-64 overflow-auto border-r border-gray-300 relative">
          <template v-for="file in filesList">
            <div class="relative" @mouseenter="showDropdown(file)" @mouseleave="hideDropdown(file)">
              <p v-if="file.leftSide" :key="file.id">
                {{ file.relPath }}
              </p>
              <ul v-if="file.hover" class="absolute left-10 top-1/2 bg-gray-700 mt-2 p-2 border border-gray-300 shadow-lg z-10" @mouseenter="showDropdown(file)" @mouseleave="hideDropdown(file)" :key="file.id + 'list'">
                <template v-for="(series, index) in file.series">
                  <li class="hover:bg-gray-900" @click="normalizePath(file, series)" :key="index">{{ series.series }}{{ series.sequence === null ? '' : ' #' + series.sequence }}</li>
                </template>
              </ul>
            </div>
          </template>
        </div>

        <div class="w-1/2 h-64 overflow-auto pl-4">
          <template v-for="file in filesList">
            <p v-if="!file.leftSide" @click="moveToLeft(file)" :key="file.id">{{ file.nomalizedPath }}</p>
          </template>
        </div>
        <!-- <div class="flex-grow" /> -->
        <div class="ml-auto">
          <ui-btn class="mb-4 block" @click.stop="normalization(true)">Normalize</ui-btn>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    library: {
      type: Object,
      default: () => null
    },
    libraryId: String,
    processing: Boolean
  },
  data() {
    return {
      filesList: []
    }
  },
  computed: {
    librarySettings() {
      return this.library.settings || {}
    },
    mediaType() {
      return this.library.mediaType
    },
    isBookLibrary() {
      return this.mediaType === 'book'
    }
  },
  methods: {
    normalization(normalization) {
      this.$axios
        .$post(`/api/libraries/${this.libraryId}/files-normalization?normalization=${normalization}`, this.filesList)
        .catch((error) => {
          console.error(error)
          this.$toast.error(error)
        })
        .then((data) => {
          if (data.files) {
            this.filesList = data.files
          }
          this.$toast.success(`Successful to normalize the book`)
        })
        .catch((error) => {
          this.$toast.error('Failed to remove metadata files')
        })
    },
    fetchFiles(normalization) {
      this.$axios
        .$post(`/api/libraries/${this.libraryId}/files-normalization?normalization=${normalization}`)
        .then((data) => {
          if (!data.files) {
            this.$toast.info(`No files were found in library`)
          } else {
            this.filesList = data.files
          }
        })
        .catch((error) => {
          console.error('Failed to remove metadata files', error)
          this.$toast.error('Failed to remove metadata files')
        })
        .finally(() => {
          this.$emit('update:processing', false)
        })
    },
    moveToLeft(file) {
      file.leftSide = true
    },
    moveToRight(file) {
      file.leftSide = false
    },
    showDropdown(file) {
      file.hover = true
    },
    hideDropdown(file) {
      file.hover = false
    },
    normalizePath(file, series) {
      if (series.series === 'No Series') {
        file.nomalizedPath = [file.author, file.bookFloder].join('/')
      } else {
        file.nomalizedPath = [file.author, series.series + (series.sequence === null ? '' : '#' + series.sequence), file.bookFloder].join('/')
      }
      if (file.nomalizedPath === file.relPath) {
        file.leftSide = true
      } else {
        file.leftSide = false
      }
    }
  },
  mounted() {
    this.fetchFiles(false)
  }
}
</script>