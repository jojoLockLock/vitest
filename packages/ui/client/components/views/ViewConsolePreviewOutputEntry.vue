<script setup lang="ts">
import type { UserConsoleLog } from 'vitest'
import { unescapeHtml } from "~/utils/escape"
import { showRaw } from '~/composables/client'
import { ref } from 'vue'
import { Collapse } from 'vue-collapsed'

const isExpandedMap = ref({})


defineProps<{
  taskName: string
  type: UserConsoleLog['type']
  time: UserConsoleLog['time']
  content: UserConsoleLog['content']
}>()

function formatTime(t: number) {
  return (new Date(t)).toLocaleTimeString()
}

function onCopy(content) {
  console.log(content)
}

function extractGroupContent(str) {
    const regex = /\[VITEST-PREVIEW:(.*?)\](.*?)\[VITEST-PREVIEW-END\]/gs;
    let match;
    let group = [];
    let lastIndex = 0;

    while ((match = regex.exec(str))) {
        let content = match[2].trim();
        let type = match[1];
        let rawContent = str.slice(lastIndex, match.index).trim();

        if (rawContent) {
            group.push({
                content: rawContent,
                type: 'RAW'
            });
        }

        group.push({
            content,
            type
        });

        lastIndex = regex.lastIndex;
    }

    let rawContent = str.slice(lastIndex).trim();
    if (rawContent) {
        group.push({
            content: rawContent,
            type: 'RAW'
        });
    }

    return group;
}

function extractContent(str) {
    const groupSplitRegex = /\[VITEST-GROUP(?: name=(.*?))?\](.*?)\[VITEST-GROUP-END(?: name=\1)?\]/gs;
    let groupMatch;
    let lastIndex = 0;
    let results = [];
    let groupId = 1;

    while ((groupMatch = groupSplitRegex.exec(str))) {
        let groupContentBefore = str.slice(lastIndex, groupMatch.index).trim();
        if (groupContentBefore) {
            results.push({
                name: `DEFAULT_GROUP_${groupId++}`,
                group: extractGroupContent(groupContentBefore)
            });
        }

        results.push({
            name: groupMatch[1] || `GROUP_${groupId++}`,
            group: extractGroupContent(groupMatch[2].trim())
        });
        
        lastIndex = groupSplitRegex.lastIndex;
    }

    let groupContentAfter = str.slice(lastIndex).trim();
    if (groupContentAfter) {
        results.push({
            name: `DEFAULT_GROUP_${groupId++}`,
            group: extractGroupContent(groupContentAfter)
        });
    }

    return results;
}


const previewContent = computed(() => {
  return extractContent(content)
})


</script>

<template>
  <div border="b base" p-4 :style="{ position:'relative' }">
    <div
      text-xs mb-1
      :class="type === 'stderr' ? 'text-red-600 dark:text-red-300' : 'op30'"
    >
      {{ formatTime(time) }} | {{ taskName }} | {{ type }} 
    </div>
    
    <div v-for="{ name, group } of extractContent(unescapeHtml(content))" :key="name" font-mono>
        <OutputWrapper :title="name">
            <div v-for="{ type, content } of group" :key={content}>

              <div  v-if="type==='RAW' && showRaw" >
                  <pre data-type="html" v-html="content" />
              </div>

              <div
                v-if="type==='MD'" 
                :style="{ 
                  position:'relative', 
                  height: '600px', 
                  marginBottom: '30px', 
                  overflow: 'scroll',
                  border: '1px solid #e2e8f0',
                }"
              >
                <CodeMirror h-full :model-value="content" read-only v-bind="{ lineNumbers: true }" :mode="markdown" />
              </div>    
            </div>      
        </OutputWrapper>
    </div>
  </div>
</template>
