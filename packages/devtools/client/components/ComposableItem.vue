<script setup lang="ts">
import type { Import, UnimportMeta } from 'unimport'

const props = defineProps<{
  item: Import
  metadata?: UnimportMeta
  filepath?: string
}>()

const copy = useCopy()
const openInEditor = useOpenInEditor()

const name = computed(() => props.item.as || props.item.name)
const usageCount = computed(() => props.metadata?.injectionUsage?.[name.value]?.count || 0)
const modules = computed(() => props.metadata?.injectionUsage?.[name.value]?.moduleIds || [])

const docsUrl = computed(() => {
  if (props.item.meta?.docsUrl)
    return props.item.meta.docsUrl
  if (['nuxt', '#app', 'nuxt3'].includes(props.item.from))
    return (ComposablesDocs.nuxt as any)[props.item.name]
  if (props.item.from === 'vue')
    return (ComposablesDocs.vue as any)[props.item.name]
})
</script>

<template>
  <VDropdown :disabled="!props.metadata">
    <button hover:text-primary>
      <code
        rounded bg-gray:5 px2 py1 font-mono text-sm
        :class="metadata && !usageCount ? 'op30 hover:op100' : ''"
      >
        {{ name }}
        <sup v-if="usageCount" text-primary>x{{ usageCount }}</sup>
      </code>
    </button>
    <template #popper>
      <div max-w-100>
        <div px4 py3 text-sm>
          <NMarkdown
            v-if="item.meta?.description"
            tag="div"
            pb3 text-sm
            :markdown="item.meta.description"
          />
          <div flex="~ gap2" n="primary xs">
            <NButton icon="carbon-copy" @click="copy(name)">
              Copy
            </NButton>
            <NButton v-if="filepath" icon="carbon-code" @click="filepath && openInEditor(filepath)">
              Source
            </NButton>
            <NButton v-if="docsUrl" icon="carbon-catalog" :to="docsUrl" target="_blank">
              Docs
            </NButton>
          </div>
        </div>
        <div border="t base" max-h-60 of-auto px4 py3>
          <template v-if="usageCount">
            <div text-sm>
              <span op50>It has been referenced </span><strong text-primary>{{ usageCount }}</strong><span op50> times by:</span>
            </div>
            <div flex="~ col gap-2" items-start pt3 text-sm>
              <FilepathItem
                v-for="id of modules" :key="id"
                :filepath="id"
              />
            </div>
          </template>
          <template v-else>
            <div text-sm op50>
              Not in use via auto import.
            </div>
          </template>
        </div>
      </div>
    </template>
  </VDropdown>
</template>
