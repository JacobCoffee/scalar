<script setup lang="ts">
import {
  ApiReference,
  type ReferenceConfiguration,
} from '@scalar/api-reference'
import { reactive } from 'vue'

import content from '../../../../specifications/scalar-galaxy-3.1.json'

const configuration = reactive<ReferenceConfiguration>({
  theme: 'default',
  proxy: import.meta.env.VITE_REQUEST_PROXY_URL,
  isEditable: false,
  layout: 'classic',
  spec: { content },
})
</script>
<template>
  <ApiReference :configuration="configuration" />
</template>
