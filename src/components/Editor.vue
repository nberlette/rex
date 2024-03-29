<script setup lang='ts'>
import { onMounted, ref, watch, defineProps } from 'vue'
import { useVModel } from '@vueuse/core'
import CodeMirror from 'codemirror'
import 'codemirror/lib/codemirror.css'
import 'codemirror/addon/display/placeholder'
import '../modes/regex'

const props = defineProps<{
  modelValue: string
  mode?: string
  placeholder?: string
  readonly?: boolean
  inline?: boolean
  wrapping?: boolean
  matches?: RegExpMatchArray[]
}>()
const el = ref<HTMLTextAreaElement | null>()
const text = useVModel(props, 'modelValue')

onMounted(() => {
  const cm = CodeMirror.fromTextArea(el.value!, {
    mode: props.mode,
    readOnly: props.readonly,
    scrollbarStyle: 'null',
    lineWrapping: props.wrapping,
    extraKeys: {
      // @ts-expect-error
      Tab: props.inline ? false : undefined,
    },
  })

  cm.on('change', () => {
    text.value = cm.getValue()
  })

  let decorations: CodeMirror.TextMarker<CodeMirror.MarkerRange>[] = []

  watch(() => props.mode, v => cm.setOption('mode', v))
  watch(() => props.readonly, v => cm.setOption('readOnly', v))
  watch(() => props.wrapping, v => cm.setOption('lineWrapping', v))
  watch(
    text,
    (v) => {
      if (v !== cm.getValue())
        cm.replaceRange(v, cm.posFromIndex(0), cm.posFromIndex(Infinity))
    },
    { immediate: true },
  )

  watch(
    () => props.matches,
    (m = []) => {
      decorations.forEach(i => i.clear())
      decorations = Array.from(m)
        .map((i, idx) => {
          const start = i.index
          if (start == null)
            return null!
          const end = start + i[0].length
          return cm.markText(
            cm.posFromIndex(start),
            cm.posFromIndex(end),
            { className: `match full-${idx % 2 === 0 ? 'even' : 'odd'}` },
          )
        })
        .filter(i => i)
    },
    { immediate: true },
  )
})
</script>

<template>
  <div class="editor relative">
    <textarea ref="el" :placeholder="props.placeholder" />
  </div>
</template>

<style lang='postcss'>
.editor {
  @apply outline-none font-mono overflow-auto;
}
.CodeMirror {
  height: 100%;
  background: transparent;
  color: inherit;
}
.CodeMirror pre.CodeMirror-placeholder {
  opacity: 0.3;
  font-style: italic;
}
.CodeMirror-cursor {
  border-color: var(--fg);
}
.CodeMirror-selected {
  background: rgba(125,125,125,0.2);
}
.source .CodeMirror-code span {
  color: var(--fg-semi);
}
.match {
  color: var(--fg) !important;
}
.match.full-even {
  border-bottom: 2px solid #e7d25e;
}
.match.full-odd {
  border-bottom: 2px solid #e79c5e;
}
</style>
