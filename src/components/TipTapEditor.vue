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
        <floating-menu :editor="editor" :tippy-options="{ duration: 100 }" v-if="editor">
          <div class="floating-menu">
            <button @click="editor.chain().focus().toggleHeading({ level: 1 }).run()" :class="{ 'is-active': editor.isActive('heading', { level: 1 }) }">
              <H1Icon class="w-4 h-4" />
            </button>
            <button @click="editor.chain().focus().toggleHeading({ level: 2 }).run()" :class="{ 'is-active': editor.isActive('heading', { level: 2 }) }">
              <H2Icon class="w-4 h-4" />
            </button>
            <button @click="editor.chain().focus().toggleBulletList().run()" :class="{ 'is-active': editor.isActive('bulletList') }">
              <ListBulletIcon class="w-4 h-4" />
            </button>
            <Listbox as="div" class="relative inline-flex">
              <ListboxLabel class="sr-only">Insert Layout Element</ListboxLabel>
                  <ListboxButton class="inline-flex items-center">
                    <span class="sr-only">Insert Layout Element</span>
                      <ViewColumnsIcon class="w-4 h-4" />
                  </ListboxButton>

                <transition leave-active-class="transition ease-in duration-100" leave-from-class="opacity-100" leave-to-class="opacity-0">
                  <ListboxOptions class="absolute left-0 z-10 mt-5 origin-top-left divide-y divide-gray-200 overflow-hidden rounded-md bg-white shadow-lg ring-1 ring-black/5 focus:outline-none">
                    <ListboxOption as="template" v-for="(option, oI) in layoutOptions" :key="oI" :value="option">
                      <div @click="editor.chain().focus().insertTable({ rows: option.rows, cols: option.columns, withHeaderRow: false }).run()" class="cursor-pointer p-4 text-sm text-center">
                        <component :is="option.icon" class="w-4 h-4" />
                      </div>
                    </ListboxOption>
                  </ListboxOptions>
                </transition>
            </Listbox>

          </div>
        </floating-menu>
        <editor-content :editor="editor" />
      </div>
    </div>
    <a @click="showOutput = !showOutput" class="block cursor-pointer text-sm text-gray-500 my-10">{{ showOutput ? 'Hide' : 'Show' }} Output</a>
    <div v-if="showOutput">
      <div class="bg-slate-700 rounded-md p-2 mt-10 text-white">{{ editor?.getHTML() }}</div>
      <div class="bg-slate-700 rounded-md p-2 mt-10 text-white">{{ editor?.getJSON() }}</div>
    </div>
  </div>
</template>
<script setup lang="ts">
import Highlight from '@tiptap/extension-highlight'
import Link from '@tiptap/extension-link'
import StarterKit from '@tiptap/starter-kit'
import TextAlign from '@tiptap/extension-text-align'
import Underline from '@tiptap/extension-underline'
import Table from '@tiptap/extension-table'
import TableCell from '@tiptap/extension-table-cell'
import TableHeader from '@tiptap/extension-table-header'
import TableRow from '@tiptap/extension-table-row'
import { BubbleMenu, Editor as TapEditor, EditorContent, FloatingMenu } from '@tiptap/vue-3'
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { BoldIcon, BugAntIcon, H1Icon, H2Icon, ItalicIcon, LinkIcon, ListBulletIcon, StrikethroughIcon, UnderlineIcon, ViewColumnsIcon } from '@heroicons/vue/24/outline'
import { Listbox, ListboxButton, ListboxLabel, ListboxOption, ListboxOptions } from '@headlessui/vue'

const layoutOptions = [
  { rows: 1, columns: 2, icon: BugAntIcon },
  { rows: 1, columns: 3, icon: BugAntIcon },
  { rows: 1, columns: 4, icon: BugAntIcon },
]

const showOutput = ref(false)
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
      Table.configure({
        resizable: true,
      }),
      TableCell,
      TableHeader,
      TableRow,
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
      <h1>Welcome to the TipTap Editor!</h1>

      <h2>Getting Started</h2>

      <p>Hey there! This is a simple WYSIWYG editor where you can add and format text however you like. Feel free to type, style, and make this space your own!</p>

      <p>Whether you're jotting down notes, writing an article, or just experimenting, here are a few things you can try:</p>

      <ul>
        <li>Use headings to organize your thoughts.</li>
        <li>Write as much or as little as you need—no pressure!</li>
        <li>Play around with bold, italics, and other formatting options.</li>
      </ul>

      <p>Have fun, and happy writing! ✨</p>
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

/* Floating menu */
.floating-menu {
  @apply flex bg-gray-300 rounded-md p-1;
}
.floating-menu  button {
  @apply bg-none px-2 rounded-sm
}
.floating-menu  button:hover {
  @apply bg-gray-300;
}
.floating-menu  button.is-active {
  @apply bg-white text-slate-900;
}
.floating-menu  button.is-active:hover {
  @apply text-slate-200;
}

/* Basic editor styles */
.tiptap {
  @apply prose;
}
.tiptap :first-child {
  @apply mt-0;
}

/* List styles */
.tiptap ul,
.tiptap ol {
  @apply p-2;
  @apply ml-4;
}
.tiptap ul {
  @apply list-disc;
}
.tiptap ol {
  @apply list-decimal;
}
.tiptap li p {
  @apply mt-0.5 mb-0.5;
}

/* Table-specific styling */
table {
  @apply w-full table-fixed border-collapse m-0 overflow-hidden;
}

table td,
table th {
  @apply border border-gray-300 box-border min-w-[1em] p-2 relative align-top;
}

table td > *,
table th > * {
  @apply mb-0;
}

table th {
  @apply bg-gray-100 font-bold text-left;
}

table .selectedCell::after {
  @apply bg-gray-200 absolute inset-0 pointer-events-none z-[2];
  content: "";
}

table .column-resize-handle {
  @apply bg-purple-500 absolute -bottom-0.5 -right-0.5 top-0 w-1 pointer-events-none;
}

.tableWrapper {
  @apply my-6 overflow-x-auto;
}

.resize-cursor {
  @apply cursor-col-resize;
}
</style>
