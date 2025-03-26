<template>
  <div>
    <div class="mt-2 control-group bg-white p-2 border-gray-300 shadow-sm dark:border-gray-700 dark:bg-slate-900 rounded-md shadow-sm">
      <div class="dark:bg-slate-300 rounded-md dark:shadow-sm">
        <bubble-menu v-if="editor" :editor="editor" :tippy-options="{ duration: 100, maxWidth: 'none' }">
          <div class="bubble-menu">
            <button @click="setLink">
              <LinkIcon class="w-4 h-4" />
            </button>
            <button @click="editor?.chain().focus().toggleBold().run()" :class="{ 'is-active': editor?.isActive('bold') }">
              <BoldIcon class="w-4 h-4" />
            </button>
            <button @click="editor?.chain().focus().toggleItalic().run()" :class="{ 'is-active font-italic': editor?.isActive('italic') }">
              <ItalicIcon class="w-4 h-4" />
            </button>
            <button @click="editor?.chain().focus().toggleStrike().run()" :class="{ 'is-active line-through': editor?.isActive('strike') }">
              <StrikethroughIcon class="w-4 h-4" />
            </button>
            <button @click="editor?.chain().focus().toggleUnderline().run()" :class="{ 'is-active underline': editor?.isActive('underline') }">
              <UnderlineIcon class="w-4 h-4" />
            </button>
          </div>
        </bubble-menu>
        <editor-content :editor="editor" />
      </div>
    </div>
    <div class="bg-slate-700 rounded-md p-2 mt-10 text-white">{{ editor?.getHTML() }}</div>
    <div class="bg-slate-700 rounded-md p-2 mt-10 text-white">{{ editor?.getJSON() }}</div>
  </div>
</template>
<script setup lang="ts">
import Highlight from '@tiptap/extension-highlight'
import Link from '@tiptap/extension-link'
import StarterKit from '@tiptap/starter-kit'
import TextAlign from '@tiptap/extension-text-align'
import Underline from '@tiptap/extension-underline'
import { BubbleMenu, Editor as TapEditor, EditorContent } from '@tiptap/vue-3'
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { BoldIcon, ItalicIcon, LinkIcon, StrikethroughIcon, UnderlineIcon } from '@heroicons/vue/24/outline'

const { modelValue } = defineProps({
  modelValue: {
    type: String,
    default: '',
  },
})

const $emit = defineEmits(['update:modelValue'])

const editor = ref<TapEditor | undefined>()

watch(
  () => modelValue,
  (value) => {
    // HTML
    const isSame = editor.value?.getHTML() === value

    // JSON
    // const isSame = JSON.stringify(editor.value.getJSON()) === JSON.stringify(value)

    if (isSame) {
      return
    }

    editor.value?.commands.setContent(value, false)
  }
)

onMounted(() => {
  editor.value = new TapEditor({
    editorProps: {
      attributes: {
        class: 'prose focus:outline-none p-2',
      },
    },
    extensions: [
      StarterKit,
      TextAlign.configure({
        types: ['heading', 'paragraph'],
      }),
      Highlight,
      Link.configure({
        openOnClick: false,
        defaultProtocol: 'https',
      }),
      Underline,
    ],
    content: `
      <p>
        Hey, try to select some text here. There will popup a menu for selecting some inline styles. Remember: you have full control about content and styling of this menu.
      </p>
    `,
    onUpdate: () => {
      // HTML
      $emit('update:modelValue', editor.value?.getHTML())

      // JSON
      // $emit('update:modelValue', editor.value.getJSON())
    },
  })
})

onBeforeUnmount(() => {
  editor.value?.destroy()
})

// popup for setting href for links
function setLink() {
  const previousUrl = editor.value?.getAttributes('link').href
  const url = window.prompt('URL', previousUrl)

  // cancelled
  if (url === null) {
    return
  }

  // empty
  if (url === '') {
    editor.value?.chain().focus().extendMarkRange('link').unsetLink().run()

    return
  }

  // update link
  editor.value?.chain().focus().extendMarkRange('link').setLink({ href: url }).run()
}
</script>
<style>
@reference '../assets/main.css';

/* Bubble menu */
.bubble-menu {
  @apply bg-white shadow-sm rounded-md border border-gray-300 flex p-2;
}
.bubble-menu  button {
  @apply bg-transparent px-2 py-1 rounded-md;
}
.bubble-menu  button:hover {
      @apply bg-slate-300;
}
.bubble-menu  button.is-active {
  @apply bg-slate-100;
}
.bubble-menu  button.is-active:hover {
  @apply bg-slate-50;
}
</style>
