<script setup lang="ts">
import DeleteIcon from '@/components/icons/DeleteIcon.vue'
import PDFPagePlaceholder from '@/components/PDFPagePlaceholder.vue'
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
const pagesToDelete = ref<number[]>([])

const pagesLoaded = ref<number[]>([])
const handleDeletePage = (page: number) => {
  if (pagesToDelete.value.includes(page)) {
    pagesToDelete.value = pagesToDelete.value.filter((p) => p !== page)
  } else {
    if (pagesToDelete.value.length >= totalPages.value - 1) {
      toast.warning('You are deleting all pages !', { theme: 'dark' })
      return
    }
    pagesToDelete.value.push(page)
  }
}
interface MetaData {
  title: string | null
  author: string | null
  subject: string | null
  keywords: string | null
  creator: string | null
  producer: string | null
  creationDate: Date | string
}

const metaDataValues = ref<MetaData>({
  title: '',
  author: '',
  subject: '',
  keywords: '',
  creator: '',
  producer: '',
  creationDate: '',
})

const handleFile = async (file: File) => {
  if (file && file.type === 'application/pdf') {
    pdfFile.value = file
    pdfSource.value = URL.createObjectURL(file)

    const arrayBuffer = await file.arrayBuffer()
    const pdfDoc = await PDFDocument.load(arrayBuffer)
    totalPages.value = pdfDoc.getPageCount()

    metaDataValues.value = {
      title: pdfDoc.getTitle() || '',
      author: pdfDoc.getAuthor() || '',
      subject: pdfDoc.getSubject() || '',
      keywords: pdfDoc.getKeywords() || '',
      creator: pdfDoc.getCreator() || '',
      producer: pdfDoc.getProducer() || '',
      creationDate: pdfDoc.getCreationDate() || new Date(),
    }

    toast.success('File uploaded successfully!', { theme: 'dark' })
    isFileUploaded.value = true
  } else {
    toast.error('Please select a valid PDF file', { theme: 'dark' })
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

const handleSubmit = async (e: Event) => {
  e.preventDefault()
  if (!pdfFile.value) {
    toast.warning('Please select a PDF file', { theme: 'dark' })
    return
  }
  try {
    const arrayBuffer = await pdfFile.value.arrayBuffer()
    const pdfDoc = await PDFDocument.load(arrayBuffer)

    // Save the metadata
    pdfDoc.setTitle(metaDataValues.value.title || '')
    pdfDoc.setAuthor(metaDataValues.value.author || '')
    pdfDoc.setSubject(metaDataValues.value.subject || '')
    pdfDoc.setKeywords(metaDataValues.value.keywords?.split(',') || [])

    // Delete pages if needed
    if (pagesToDelete.value.length > 0) {
      const sortedPages = pagesToDelete.value.sort((a, b) => b - a)
      sortedPages.forEach((page) => {
        const pageIndex = page - 1
        if (pageIndex >= 0 && pageIndex < pdfDoc.getPageCount()) {
          pdfDoc.removePage(pageIndex)
        }
      })
    }

    // Save and download the PDF
    const pdfBytes = await pdfDoc.save()
    const blob = new Blob([pdfBytes], { type: 'application/pdf' })
    const url = URL.createObjectURL(blob)

    // Create an anchor element and trigger the download
    const link = document.createElement('a')
    link.href = url

    // Replace spaces with underscores
    const fileName = metaDataValues.value.title
      ? metaDataValues.value.title.replace(/ /g, '_') + '.pdf'
      : 'edited_document.pdf'

    link.download = fileName
    document.body.appendChild(link)
    link.click()
    document.body.removeChild(link)

    // Revoke the object URL to free up memory
    URL.revokeObjectURL(url)
  } catch (error) {
    console.error('Error saving PDF:', error)
    toast.error('Error saving PDF', { theme: 'dark' })
  }
}
const handleResetFile = () => {
  isFileUploaded.value = false
  pdfFile.value = null
  pdfSource.value = null
  totalPages.value = 1
  pagesToDelete.value = []
  pagesLoaded.value = []
  metaDataValues.value = {
    title: '',
    author: '',
    subject: '',
    keywords: '',
    creator: '',
    producer: '',
    creationDate: '',
  }
}
</script>

<template>
  <section v-if="!isFileUploaded" class="mx-auto px-4">
    <div class="text-center">
      <h1 class="text-3xl font-bold mb-4">PDF Meta Data Editor</h1>
      <p class="mb-6">Select a PDF file to edit its metadata or delete pages quickly and easily.</p>
      <p>
        This site does not upload your file to any server. All data is managed locally on your
        browser.
      </p>

      <div
        @dragover.prevent="isDragging = true"
        @dragleave.prevent="isDragging = false"
        @drop.prevent="onDrop"
        @click="openFileDialog"
        class="mt-10 flex justify-center items-center border-2 border-dashed border-slate-500/50 rounded-lg p-10 h-64 cursor-pointer transition-colors hover:bg-slate-600/20"
        :class="{ 'bg-slate-100/50': isDragging }"
      >
        <input type="file" accept=".pdf" @change="onFileSelected" ref="fileInput" class="hidden" />
        <p v-if="isDragging" class="text-lg text-slate-600">Drop your PDF file here</p>
        <p v-else class="text-lg text-slate-600">
          Drop your PDF file here or click to select a file
        </p>
      </div>
    </div>
  </section>
  <section v-else class="relative w-full mx-auto px-4 flex justify-center gap-10">
    <main class="flex-2">
      <h2 class="text-2xl font-bold mb-4">PDF Preview</h2>
      <div v-if="pdfSource" class="space-y-4">
        <div
          v-for="page in totalPages"
          :key="page"
          class="relative mb-4 bg-slate-300/30 rounded-lg"
        >
          <div
            class="border border-gray-300 rounded-lg overflow-hidden"
            :class="{ 'opacity-10': pagesToDelete.includes(page) }"
          >
            <PDFPagePlaceholder v-if="!pagesLoaded.includes(page)" />
            <vue-pdf-embed
              :source="pdfSource"
              :page="page"
              @loaded="() => pagesLoaded.push(page)"
            />
          </div>
          <div
            key="delete-page-{{ page }}"
            class="absolute top-0 -left-[50px] cursor-pointer border border-red-500 p-2 rounded-lg text-red-500 hover:bg-red-500/10"
            :class="{ 'bg-red-500/10': pagesToDelete.includes(page) }"
            @click="handleDeletePage(page)"
          >
            <DeleteIcon />
          </div>
        </div>
      </div>
    </main>
    <aside class="sticky top-10 max-w-sm h-fit">
      <h2 class="text-2xl font-bold mb-4">PDF Metadata</h2>
      <p>Total pages: {{ totalPages }}</p>
      <p>Pages to delete: {{ pagesToDelete.join(', ') }}</p>
      <form @submit="handleSubmit" class="mt-5 border border-gray-300 rounded-lg p-4">
        <label for="title" class="block mb-2"
          >Title
          <input type="text" v-model="metaDataValues.title" placeholder="Title" />
        </label>
        <label for="author" class="block mb-2"
          >Author
          <input type="text" v-model="metaDataValues.author" placeholder="Author" />
        </label>
        <label for="subject" class="block mb-2"
          >Subject
          <input type="text" v-model="metaDataValues.subject" placeholder="Subject" />
        </label>
        <label for="keywords" class="block mb-2"
          >Keywords (separate with commas)
          <textarea rows="5" type="text" v-model="metaDataValues.keywords" placeholder="Keywords" />
        </label>
        <button type="submit" class="w-full p-2 mt-2 bg-blue-500 text-white rounded-lg">
          Download PDF
        </button>
        <button
          @click="handleResetFile"
          class="w-full p-2 mt-2 bg-slate-400/20 border border-red-500 text-white rounded-lg"
        >
          Reset file
        </button>
      </form>
    </aside>
  </section>
</template>
