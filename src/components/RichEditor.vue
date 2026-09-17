<script setup lang="ts">
import { ref } from 'vue'
import { Node, mergeAttributes } from '@tiptap/core'
import { useEditor, EditorContent } from '@tiptap/vue-3'
import StarterKit from '@tiptap/starter-kit'
import Image from '@tiptap/extension-image'
import Link from '@tiptap/extension-link'
import TextAlign from '@tiptap/extension-text-align'
import { TextStyle } from '@tiptap/extension-text-style'
import { Color } from '@tiptap/extension-color'
import Underline from '@tiptap/extension-underline'
import Highlight from '@tiptap/extension-highlight'

const fileImageInput = ref<HTMLInputElement | null>(null)

// === КАСТОМНЫЙ УЗЕЛ: Бейдж файла в стиле Notion ===
const FileLinkBlock = Node.create({
  name: 'fileLinkBlock',
  group: 'inline',
  inline: true,
  atom: true,
  addAttributes() {
    return {
      fileName: { default: 'document' },
      href: { default: '#' },
      fileType: { default: 'FILE' },
      colorClass: { default: 'bg-[#1e2330] text-gray-200 border-gray-700/60' },
      icon: { default: '📎' }
    }
  },
  parseHTML() {
    return [{ tag: 'a[data-file-link]' }]
  },
  renderHTML({ HTMLAttributes }) {
    return [
      'a',
      mergeAttributes({
        href: HTMLAttributes.href,
        'data-file-link': '',
        class: `notion-file-link inline-flex items-center gap-2 px-3 py-1.5 mx-1 my-0.5 rounded-lg text-sm font-medium no-underline cursor-pointer border transition-all shadow-sm ${HTMLAttributes.colorClass}`,
        contenteditable: 'false',
        target: '_blank',
        rel: 'noopener noreferrer'
      }),
      ['span', { class: 'text-base' }, HTMLAttributes.icon],
      ['span', { class: 'underline truncate max-w-[220px]' }, HTMLAttributes.fileName],
      ['span', { class: 'text-xs uppercase tracking-wider ml-1 px-1.5 py-0.5 bg-black/30 rounded opacity-80' }, HTMLAttributes.fileType]
    ]
  },
})

// === ТОЧНЫЙ АНАЛИЗАТОР ССЫЛОК (WORD, EXCEL, PDF И ДР.) ===
function analyzeFileTarget(input: string) {
  const lower = input.toLowerCase()
  let ext = 'LINK'
  let fileName = 'Посилання'
  let colorClass = 'bg-indigo-500/15 text-indigo-300 border-indigo-500/30 hover:bg-indigo-500/25'
  let icon = '🌐'

  // Excel / Таблицы
  if (lower.includes('excel') || lower.includes('spreadsheets') || lower.includes('sheets') || lower.includes('view.officeapps.live.com') && lower.includes('xl') || lower.endsWith('.xlsx') || lower.endsWith('.xls') || lower.endsWith('.csv')) {
    ext = 'EXCEL'
    icon = '📊'
    colorClass = 'bg-emerald-500/15 text-emerald-300 border-emerald-500/30 hover:bg-emerald-500/25'
    fileName = 'Таблиця Excel'
  } 
  // Word / Документы
  else if (lower.includes('word') || lower.includes('document') || lower.includes('docs.google.com/document') || lower.endsWith('.docx') || lower.endsWith('.doc') || lower.endsWith('.txt')) {
    ext = 'WORD'
    icon = '📄'
    colorClass = 'bg-blue-500/15 text-blue-300 border-blue-500/30 hover:bg-blue-500/25'
    fileName = 'Документ Word'
  } 
  // Универсальные ссылки OneDrive / SharePoint (если пользователь вставил 1drv.ms — определяем по контексту или ставим общий документ Office)
  else if (lower.includes('1drv.ms') || lower.includes('sharepoint.com')) {
    // Если в ссылке явно есть признаки таблицы или ворда
    if (lower.includes('x/') || lower.includes('excel') || lower.includes('_layouts/15/excel')) {
      ext = 'EXCEL'
      icon = '📊'
      colorClass = 'bg-emerald-500/15 text-emerald-300 border-emerald-500/30 hover:bg-emerald-500/25'
      fileName = 'Таблиця OneDrive'
    } else {
      ext = 'WORD'
      icon = '📄'
      colorClass = 'bg-blue-500/15 text-blue-300 border-blue-500/30 hover:bg-blue-500/25'
      fileName = 'Документ OneDrive'
    }
  }
  // PowerPoint / Презентации
  else if (lower.includes('presentation') || lower.includes('powerpoint') || lower.includes('slides') || lower.endsWith('.pptx') || lower.endsWith('.ppt')) {
    ext = 'SLIDES'
    icon = '📽️'
    colorClass = 'bg-orange-500/15 text-orange-300 border-orange-500/30 hover:bg-orange-500/25'
    fileName = 'Презентація'
  }
  // PDF
  else if (lower.endsWith('.pdf') || lower.includes('/pdf/')) {
    ext = 'PDF'
    icon = '📕'
    colorClass = 'bg-red-500/15 text-red-300 border-red-500/30 hover:bg-red-500/25'
    fileName = 'PDF Документ'
  } 
  // Архивы
  else if (['.zip', '.rar', '.7z', '.tar'].some(e => lower.endsWith(e))) {
    ext = 'ARCHIVE'
    icon = '📦'
    colorClass = 'bg-yellow-500/15 text-yellow-300 border-yellow-500/30 hover:bg-yellow-500/25'
    fileName = 'Архів файлів'
  }
  // GitHub / Код
  else if (lower.includes('github.com') || lower.includes('gitlab.com')) {
    ext = 'CODE'
    icon = '💻'
    colorClass = 'bg-purple-500/15 text-purple-300 border-purple-500/30 hover:bg-purple-500/25'
    fileName = 'GitHub Репозиторій'
  }
  // Обычный сайт
  else {
    try {
      const urlObj = new URL(input)
      fileName = urlObj.hostname
    } catch {
      fileName = 'Посилання'
    }
  }

  return { fileName, href: input, fileType: ext, colorClass, icon }
}

// === ИНИЦИАЛИЗАЦИЯ РЕДАКТОРА ===
const editor = useEditor({
  content: '<h2>Документація проекту</h2><p>Скопіюйте посилання на Word або Excel (навіть коротке з OneDrive) і просто вставте його сюди через Ctrl + V!</p>',
  extensions: [
    StarterKit,
    TextStyle,
    Color,
    Underline,
    Highlight.configure({ multicolor: true }),
    Image.configure({ inline: true, allowBase64: true }),
    Link.configure({ 
      openOnClick: false,
      HTMLAttributes: {
        target: '_blank',
        rel: 'noopener noreferrer',
        class: 'text-indigo-400 underline cursor-pointer hover:text-indigo-300 transition-colors'
      }
    }),
    TextAlign.configure({ types: ['heading', 'paragraph'] }),
    FileLinkBlock,
  ],
  editorProps: {
    attributes: {
      class: 'prose prose-invert max-w-none focus:outline-none min-h-[500px] p-6',
    },
    // Автоформатирование при вставке любой ссылки
    handlePaste(view, event) {
      const text = event.clipboardData?.getData('text/plain')?.trim()
      if (text && /^https?:\/\//i.test(text)) {
        const fileNodeType = view.state.schema.nodes.fileLinkBlock
        if (!fileNodeType) return false

        event.preventDefault()
        const config = analyzeFileTarget(text)
        view.dispatch(
          view.state.tr.replaceSelectionWith(
            fileNodeType.create(config)
          )
        )
        return true
      }
      return false
    }
  },
})

// Загрузка картинок с ПК
const handleImageUpload = (event: Event) => {
  const target = event.target as HTMLInputElement
  if (!target.files || target.files.length === 0) return
  const file = target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    const base64 = e.target?.result
    if (base64 && editor.value) {
      editor.value.chain().focus().setImage({ src: base64 as string }).run()
    }
    target.value = '' 
  }
  reader.readAsDataURL(file)
}

// Обычные текстовые линки
const setLink = () => {
  const previousUrl = editor.value?.getAttributes('link').href
  let url = window.prompt('URL посилання:', previousUrl)
  if (url === null) return 
  if (url === '') {
    editor.value?.chain().focus().extendMarkRange('link').unsetLink().run()
    return
  }
  if (!/^https?:\/\//i.test(url)) url = 'https://' + url
  editor.value?.chain().focus().extendMarkRange('link').setLink({ href: url }).run()
}
</script>

<template>
  <div class="bg-[#111522] border border-gray-800 rounded-xl overflow-hidden flex flex-col h-full" v-if="editor">
    
    <input type="file" ref="fileImageInput" @change="handleImageUpload" accept="image/*" class="hidden" />

    <div class="flex flex-wrap items-center gap-2 p-2 border-b border-gray-800 bg-[#1a1f35]">
      
      <div class="flex items-center gap-1 border border-gray-700 rounded p-1 bg-[#151a2a]">
        <button @click="editor.chain().focus().toggleBold().run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('bold')}" class="w-8 h-8 flex items-center justify-center rounded text-gray-300 font-bold hover:bg-gray-700">B</button>
        <button @click="editor.chain().focus().toggleItalic().run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('italic')}" class="w-8 h-8 flex items-center justify-center rounded text-gray-300 italic hover:bg-gray-700">I</button>
        <button @click="editor.chain().focus().toggleUnderline().run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('underline')}" class="w-8 h-8 flex items-center justify-center rounded text-gray-300 underline hover:bg-gray-700">U</button>
        <button @click="editor.chain().focus().toggleStrike().run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('strike')}" class="w-8 h-8 flex items-center justify-center rounded text-gray-300 line-through hover:bg-gray-700">S</button>
        <button @click="editor.chain().focus().toggleHighlight().run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('highlight')}" class="w-8 h-8 flex items-center justify-center rounded text-gray-300 hover:bg-gray-700">🖍️</button>
        
        <div class="flex items-center gap-1 ml-1 pl-1 border-l border-gray-700">
          <input type="color" @input="editor.chain().focus().setColor(($event.target as HTMLInputElement).value).run()" class="w-7 h-7 rounded border-0 cursor-pointer bg-transparent" title="Колір тексту" />
          <button @click="editor.chain().focus().unsetColor().run()" class="px-2 py-1 text-xs rounded text-gray-400 hover:text-white hover:bg-gray-700" title="Скинути колір тексту">Скинути</button>
        </div>
      </div>

      <div class="flex items-center gap-1 border border-gray-700 rounded p-1 bg-[#151a2a]">
        <button @click="editor.chain().focus().setTextAlign('left').run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive({ textAlign: 'left' })}" class="px-2 py-1 rounded hover:bg-gray-700 text-gray-300 text-sm">Ліворуч</button>
        <button @click="editor.chain().focus().setTextAlign('center').run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive({ textAlign: 'center' })}" class="px-2 py-1 rounded hover:bg-gray-700 text-gray-300 text-sm">Центр</button>
        <button @click="editor.chain().focus().setTextAlign('right').run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive({ textAlign: 'right' })}" class="px-2 py-1 rounded hover:bg-gray-700 text-gray-300 text-sm">Праворуч</button>
      </div>

      <div class="flex items-center gap-1 border border-gray-700 rounded p-1 bg-[#151a2a]">
        <button @click="editor.chain().focus().toggleHeading({ level: 2 }).run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('heading', { level: 2 })}" class="px-2 py-1 rounded text-gray-300 font-bold text-sm hover:bg-gray-700">H2</button>
        <button @click="editor.chain().focus().toggleHeading({ level: 3 }).run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('heading', { level: 3 })}" class="px-2 py-1 rounded text-gray-300 font-bold text-sm hover:bg-gray-700">H3</button>
        <div class="w-px h-5 bg-gray-600 mx-1"></div>
        <button @click="editor.chain().focus().toggleBulletList().run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('bulletList')}" class="px-2 py-1 rounded hover:bg-gray-700 text-gray-300 text-sm">• Список</button>
        <button @click="editor.chain().focus().toggleOrderedList().run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('orderedList')}" class="px-2 py-1 rounded hover:bg-gray-700 text-gray-300 text-sm">1. Список</button>
      </div>

      <div class="flex items-center gap-1 border border-gray-700 rounded p-1 ml-auto bg-[#151a2a]">
        <button @click="editor.chain().focus().toggleCodeBlock().run()" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('codeBlock')}" class="px-2 py-1 rounded hover:bg-gray-700 text-gray-300 text-sm">Код</button>
        <button @click="setLink" :class="{'bg-indigo-500/20 text-indigo-400': editor.isActive('link')}" class="px-2 py-1 rounded hover:bg-gray-700 text-gray-300 text-sm">🔗 Лінк</button>
        <button @click="fileImageInput?.click()" class="px-2 py-1 rounded hover:bg-gray-700 text-gray-300 text-sm">🖼️ Фото</button>
      </div>
    </div>

    <div class="flex-1 overflow-y-auto bg-[#0b0f19] custom-editor">
      <EditorContent :editor="editor" />
    </div>
  </div>
</template>

<style>
.custom-editor .tiptap { min-height: 100%; }
.custom-editor .tiptap p { margin-bottom: 1rem; color: #d1d5db; }
.custom-editor .tiptap h2 { font-size: 1.5rem; font-weight: 700; margin-top: 2rem; margin-bottom: 1rem; color: #ffffff; }
.custom-editor .tiptap h3 { font-size: 1.25rem; font-weight: 600; margin-top: 1.5rem; margin-bottom: 0.75rem; color: #f3f4f6; }

.custom-editor .tiptap u { text-decoration-color: #818cf8; text-decoration-thickness: 2px; }
.custom-editor .tiptap mark { background-color: rgba(99, 102, 241, 0.3); color: inherit; border-radius: 0.25rem; padding: 0.125rem 0.25rem; }
.custom-editor .tiptap ul { list-style-type: disc; padding-left: 1.5rem; margin-bottom: 1rem; color: #d1d5db; }
.custom-editor .tiptap ol { list-style-type: decimal; padding-left: 1.5rem; margin-bottom: 1rem; color: #d1d5db; }
.custom-editor .tiptap pre { background: #1e2336; color: #a5b4fc; padding: 1rem; border-radius: 0.5rem; overflow-x: auto; font-family: monospace; margin-bottom: 1rem; border: 1px solid #374151; }
.custom-editor .tiptap img { max-width: 100%; height: auto; border-radius: 0.5rem; margin: 1.5rem 0; display: block; }
.custom-editor .tiptap img.ProseMirror-selectednode { outline: 2px solid #6366f1; }
</style>