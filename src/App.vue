<script setup lang="ts">
import { computed, ref, watch } from 'vue'

const splitLength = ref(25)
const splitText = ref('')
const isCopied = ref(false)
const isPermissionDenied = ref(false)

watch([splitLength, splitText], () => {
  isCopied.value = false
  isPermissionDenied.value = false
})

const splittedText = computed(() => {
  const text = splitText.value
  const length = splitLength.value
  const array = []
  for (let i = 0; i < text.length; i += length) {
    array.push(text.slice(i, i + length))
  }

  return array
})

const textAsPlain = computed(() => {
  return new ClipboardItem({ 'text/plain': new Blob([splittedText.value.join('\n')]) })
})

const textAsExcel = computed(() => {
  let result =
    '<table border="0" cellpadding="0" cellspacing="0" width="4" style="border-collapse:collapse;width:48pt">'
  result += '<colgroup><col width="64" style="width:48pt"></colgroup>'
  result += '<tbody>'
  for (let i = 0; i < splittedText.value.length; i++) {
    result += '<tr height="19" style="height:14.4pt">'
    result += `<td height="19" width="64" style="height:14.4pt;width:48pt">${splittedText.value[i]}</td>`
    result += '</tr>'
  }
  result += '</tbody>'
  result += '</table>'

  return new ClipboardItem({ 'text/html': new Blob([result], { type: 'text/html' }) })
})

async function copy() {
  const result = await navigator.permissions.query({ name: 'clipboard-write' as any })
  if (result.state !== 'granted') {
    isPermissionDenied.value = true
    return
  }
  navigator.clipboard.write([textAsExcel.value])
  isCopied.value = true
}
</script>

<template>
  <div class="flex h-screen w-screen flex-col gap-2 p-4 font-sans">
    <label class="flex flex-row gap-2">
      <span class="text-lg">Split length</span>
      <input v-model="splitLength" type="number" class="w-20 border" />
    </label>
    <label class="flex flex-col gap-2">
      <span class="text-lg">Split text</span>
      <textarea v-model="splitText" class="h-40 w-full border" placeholder="Enter text to split"></textarea>
    </label>
    <button @click="copy" class="bg-green-200">Split & Copy</button>
    <div v-if="isCopied" class="bg-green-500 text-white">Copied!</div>
    <div v-if="isPermissionDenied" class="bg-red-500 text-white">Permission denied!</div>
  </div>
</template>
