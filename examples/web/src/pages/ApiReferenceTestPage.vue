<script setup lang="ts">
import {
  ApiReference,
  type ReferenceConfiguration,
  type ReferenceLayoutType,
} from '@scalar/api-reference'
import { type ThemeId } from '@scalar/themes'
import { reactive } from 'vue'
import { useRoute } from 'vue-router'

import content from '../../../../specifications/scalar-galaxy-3.1.json'

const route = useRoute()

const configuration = reactive<ReferenceConfiguration>({
  theme: (`${route.params['theme']}` as ThemeId) ?? 'default',
  isEditable: false,
  layout: `${route.params['layout']}` as ReferenceLayoutType,
  darkMode: !!route.query['darkMode'],
  spec: { content },
})
</script>
<template>
  <ApiReference :configuration="configuration" />
</template>
