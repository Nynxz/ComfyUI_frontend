<template>
  <div v-if="hasAnyError && showErrorsTabEnabled" :class="errorWrapperStyles">
    <Button
      variant="textonly"
      :class="
        cn(
          tabStyles,
          'box-border w-full rounded-none bg-destructive-background pt-9 pb-4 text-white hover:bg-destructive-background-hover',
          footerRadiusClass
        )
      "
      @pointerup="snapshotDragOnPointerUp"
      @click.stop="emitIfNotDragged('openErrors')"
    >
      <div class="flex size-full items-center justify-center gap-2">
        <span class="truncate">{{ t('g.error') }}</span>
        <i class="icon-[lucide--info] size-4 shrink-0" />
      </div>
    </Button>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue'
import { useI18n } from 'vue-i18n'
import Button from '@/components/ui/button/Button.vue'
import { RenderShape } from '@/lib/litegraph/src/litegraph'
import { layoutStore } from '@/renderer/core/layout/store/layoutStore'
import { cn } from '@comfyorg/tailwind-utils'

const { t } = useI18n()

interface Props {
  hasAnyError: boolean
  showErrorsTabEnabled: boolean
  shape?: RenderShape
}

const { hasAnyError, shape } = defineProps<Props>()

const emit = defineEmits<{
  openErrors: []
}>()

let suppressNextClick = false

function snapshotDragOnPointerUp() {
  suppressNextClick = layoutStore.isDraggingVueNodes.value
}

function emitIfNotDragged(name: 'openErrors') {
  const wasDrag = suppressNextClick
  suppressNextClick = false
  if (wasDrag) return
  if (name === 'openErrors') emit('openErrors')
}

const RADIUS_CLASS = {
  'rounded-b-17': 'rounded-b-[17px]',
  'rounded-b-20': 'rounded-b-[20px]',
  'rounded-br-17': 'rounded-br-[17px]',
  'rounded-br-20': 'rounded-br-[20px]'
} as const

function getBottomRadius(
  nodeShape: RenderShape | undefined,
  size: '17px' | '20px'
): string {
  if (nodeShape === RenderShape.BOX) return ''
  const prefix = nodeShape === RenderShape.CARD ? 'rounded-br' : 'rounded-b'
  const key =
    `${prefix}-${size === '17px' ? '17' : '20'}` as keyof typeof RADIUS_CLASS
  return RADIUS_CLASS[key]
}

const footerRadiusClass = computed(() =>
  getBottomRadius(shape, hasAnyError ? '20px' : '17px')
)

const tabStyles = 'pointer-events-auto h-9 text-xs'
const footerWrapperBase = 'isolate -z-1 -mt-5 box-border flex'
const errorWrapperStyles = cn(
  footerWrapperBase,
  '-mx-1 -mb-2 w-[calc(100%+8px)] pb-1'
)
</script>
