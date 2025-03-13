<script setup lang="ts">
import { PDFDocument } from 'pdf-lib'
import { ref } from 'vue'
import VuePdfEmbed from 'vue-pdf-embed'
import { toast } from 'vue3-toastify'

const isDragging = ref(false)
const fileInput = ref<HTMLInputElement | null>(null)
const isFileUploaded = ref(false)
const pdfFile = ref<File | null>(null)
const pdfSource = ref<string | null>(null)
const totalPages = ref(1)

const handleFile = async (file: File) => {
  if (file && file.type === 'application/pdf') {
    console.log('Archivo PDF recibido:', file)
    isFileUploaded.value = true
    pdfFile.value = file
    pdfSource.value = URL.createObjectURL(file)

    const arrayBuffer = await file.arrayBuffer()
    const pdfDoc = await PDFDocument.load(arrayBuffer)
    totalPages.value = pdfDoc.getPageCount()

    toast.success('File uploaded successfully!')
  } else {
    toast.error('Please select a valid PDF file')
  }
}

const onDrop = (e: DragEvent) => {
  isDragging.value = false
  const file = e.dataTransfer?.files[0]
  if (file) handleFile(file)
}

const onFileSelected = (e: Event) => {
  const target = e.target as HTMLInputElement
  const file = target.files?.[0]
  if (file) handleFile(file)
}

const openFileDialog = () => {
  fileInput.value?.click()
}
</script>

<template>
  <section v-if="!isFileUploaded" class="mx-auto px-4">
    <div class="text-center">
      <h1 class="text-3xl font-bold mb-4">PDF Meta data Editor!</h1>
      <p class="mb-6">Select a PDF file to edit the meta data</p>
      <div
        @dragover.prevent="isDragging = true"
        @dragleave.prevent="isDragging = false"
        @drop.prevent="onDrop"
        @click="openFileDialog"
        class="mt-10 flex justify-center items-center border-2 border-dashed border-slate-500/50 rounded-lg p-10 h-64 cursor-pointer transition-colors"
        :class="{ 'bg-slate-100/50': isDragging }"
      >
        <input type="file" accept=".pdf" @change="onFileSelected" ref="fileInput" class="hidden" />
        <p v-if="isDragging" class="text-lg text-slate-600">Drop your PDF file here</p>
        <p v-else class="text-lg text-slate-600">
          Drag and drop your PDF file here or click to select a file
        </p>
      </div>
    </div>
  </section>
  <section v-else class="mx-auto px-4">
    <h2 class="text-2xl font-bold mb-4">PDF Preview</h2>
    <p class="mb-4">File name: {{ pdfFile?.name }}</p>
    <div v-if="pdfSource" class="space-y-4">
      <div
        v-for="page in totalPages"
        :key="page"
        class="border border-gray-300 rounded-lg overflow-hidden mb-4"
      >
        <vue-pdf-embed :source="pdfSource" :page="page" />
      </div>
    </div>
  </section>
</template>
