<script lang="ts" setup>
import { Collapse } from 'vue-collapsed'

const isExpanded = ref(false)

function handleCollapse() {
   isExpanded.value = !isExpanded.value
}

/**
 * Accessibility attributes
 *
 * https://www.w3.org/WAI/ARIA/apg/example-index/accordion/accordion
 */

const TOGGLE_ID = 'toggle'
const COLLAPSE_ID = 'collapse'

const toggleAttrs = computed(() => ({
   id: TOGGLE_ID,
   'aria-controls': COLLAPSE_ID,
   'aria-expanded': isExpanded.value,
}))

const collapseAttrs = {
   'aria-labelledby': TOGGLE_ID,
   id: COLLAPSE_ID,
   role: 'region',
}

defineProps<{
  title: string
}>()


</script>

<template>
   <article class="Wrapper">
      <ExampleHeader
         title="Single Collapse"
         href="SingleCollapse.vue"
         hint="With Aria attributes"
      />
      <div class="Section">
         <a
            :style="{ cursor: 'pointer', textDecoration: 'underline'  }"
            v-bind="toggleAttrs"
            @click="handleCollapse"
            :class="['Panel', { Active: isExpanded }]"
         >
            {{ title || 'GROUP NAME' }}
         </a>
         <Collapse v-bind="collapseAttrs" :when="isExpanded">
            <slot></slot>
         </Collapse>
      </div>
   </article>
</template>

<!-- Check styles in assets/style.css -->