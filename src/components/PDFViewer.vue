<template>
  <div class="pdf-viewer">

    <div
      class="controller floated top right"
    >

      <PDFPaginator
        v-model="currentPage"
        :pageCount="pageCount"
      />

    </div>

    <div
      class="controller floated bottom right"
    >

      <PDFZoom
        :scale="scale"
        @change="updateScale"
        @fit="updateFit"
      />

    </div>

    <PDFData
      class="pdf-viewer__main"
      :url="url"
      @page-count="updatePageCount"
      @page-focus="updateCurrentPage"
      @document-rendered="onDocumentRendered"
      @document-errored="onDocumentErrored"
      >
      <template v-slot:preview="{pages}">
        <PDFPreview
          v-show="isPreviewEnabled"
          class="pdf-viewer__preview"
          v-bind="{pages, scale, currentPage, pageCount, isPreviewEnabled}"
          />
      </template>

      <template v-slot:document="{pages}">
        <PDFDocument
          class="pdf-viewer__document"
          :class="{ 'preview-enabled': isPreviewEnabled }"
          v-bind="{pages, scale, optimalScale, fit, currentPage, pageCount, isPreviewEnabled}"
          @scale-change="updateScale"
          />
      </template>
    </PDFData>
  </div>
</template>

<script>
import PreviewIcon from '../assets/icon-preview.svg';

import PDFDocument from './PDFDocument';
import PDFData from './PDFData';
import PDFPaginator from './PDFPaginator';
import PDFPreview from './PDFPreview';
import PDFZoom from './PDFZoom';

function floor(value, precision) {
  const multiplier = Math.pow(10, precision || 0);
  return Math.floor(value * multiplier) / multiplier;
}

export default {
  name: 'PDFViewer',

  components: {
    PDFDocument,
    PDFData,
    PDFPaginator,
    PDFPreview,
    PDFZoom,
    PreviewIcon,
  },

  props: {
    url: String,
  },

  data() {
    return {
      scale: undefined,
      optimalScale: 2.0,
      fit: undefined,
      currentPage: 1,
      pageCount: undefined,
      isPreviewEnabled: false,
    };
  },

  methods: {
    onDocumentRendered() {
      this.$emit('document-errored', this.url);
    },

    onDocumentErrored(e) {
      this.$emit('document-errored', e);
    },

    updateScale({scale, isOptimal = false}) {
      const roundedScale = floor(scale, 2);
      if (isOptimal) this.optimalScale = roundedScale;
      this.scale = roundedScale;
    },

    updateFit(fit) {
      this.fit = fit;
    },

    updatePageCount(pageCount) {
      this.pageCount = pageCount;
    },

    updateCurrentPage(pageNumber) {
      this.currentPage = pageNumber;
    },

    togglePreview() {
      this.isPreviewEnabled = !this.isPreviewEnabled;
    },
  },

  watch: {
    url() {
      this.currentPage = undefined;
    },
  },

  mounted() {
    document.body.classList.add('overflow-hidden');
  },
};
</script>

<style scoped>
header {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  padding: 1em;
  position: relative;
  z-index: 99;
}
.header-item {
  margin: 0 2.5em;
}

.pdf-viewer .pdf-viewer__document,
.pdf-viewer .pdf-viewer__preview {
  top: 0;
}

.pdf-viewer__preview {
  display: block;
  width: 15%;
  right: 85%;
}

.pdf-viewer__document {
  top: 0;
  width: 100%;
  left: 0;
}

.pdf-viewer__document.preview-enabled {
  width: 85%;
  left: 15%;
}

.floated {
  position: fixed;
  z-index: 100;
}

.floated.top {
  top: 0;
}

.floated.bottom {
  bottom: 0;
}

.floated.right {
  right: 0;
}

.floated.left {
  left: 0;
}

.controller {
  background: rgba(50, 50, 50, .25);
  padding: 0.25em;
}

.controller.floated.top.right {
  border-bottom-left-radius: .25em;
}

.controller.floated.bottom.right {
  border-top-left-radius: .25em;
}

@media print {
  header {
    display: none;
  }
}
</style>
