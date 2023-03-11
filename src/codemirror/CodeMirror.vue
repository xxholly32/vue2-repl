<template>
  <div class="editor" ref="el"></div>
</template>

<script setup lang="ts">
import { ref, onMounted, watchEffect, inject } from 'vue'
import { debounce } from '../utils'
import CodeMirror from './codemirror'

export interface Props {
  mode?: string
  value?: string
  readonly?: boolean
}

const props = withDefaults(defineProps<Props>(), {
  mode: 'htmlmixed',
  value: '',
  readonly: false
})

const emit = defineEmits<(e: 'change', value: string) => void>()

const el = ref()
const needAutoResize = inject('autoresize')

onMounted(() => {
  const addonOptions = {
    autoCloseBrackets: true,
    autoCloseTags: true,
    matchBrackets: true,
    foldGutter: true,
    highlightSelectionMatches: {
      minChars: 2,
      trim: true,
      style: "matchhighlight",
      showToken: false
    },
    gutters: ['CodeMirror-linenumbers', 'CodeMirror-foldgutter']
  }

  const editor = CodeMirror(el.value!, {
    value: '',
    mode: props.mode,
    readOnly: props.readonly,
    extraKeys: {'Ctrl': 'autocomplete'},//自定义快捷键
    tabSize: 2,
    lineWrapping: true,
    lineNumbers: true,
    hintOptions: {
      completeSingle: false
    },
    ...addonOptions
  })

  editor.on('change', () => {
    emit('change', editor.getValue())
  })

  watchEffect(() => {
    const cur = editor.getValue()
    if (props.value !== cur) {
      editor.setValue(props.value)
    }
  })

  watchEffect(() => {
    editor.setOption('mode', props.mode)
  })

  setTimeout(() => {
    editor.refresh()
  }, 50)

  if (needAutoResize) {
    window.addEventListener(
      'resize',
      debounce(() => {
        editor.refresh()
      })
    )
  }
})
</script>

<style>
.editor {
  position: relative;
  height: 100%;
  width: 100%;
  overflow: hidden;
}

.CodeMirror {
  font-family: var(--font-code);
  line-height: 1.5;
  height: 100%;
}

.cm-matchhighlight {
  background: rgba(150,150,150,0.5) !important
}
</style>
