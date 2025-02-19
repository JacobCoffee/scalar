<script setup lang="ts">
import rehypeExternalLinks from 'rehype-external-links'
import rehypeFormat from 'rehype-format'
import rehypeHighlight from 'rehype-highlight'
import rehypeRaw from 'rehype-raw'
import rehypeSanitize, { defaultSchema } from 'rehype-sanitize'
import rehypeStringify from 'rehype-stringify'
import remarkGfm from 'remark-gfm'
import remarkParse from 'remark-parse'
import remarkRehype from 'remark-rehype'
import { unified } from 'unified'
import { onServerPrefetch, ref, watch } from 'vue'

import { sleep } from '../../helpers'

const props = withDefaults(
  defineProps<{
    value?: string
    withImages?: boolean
  }>(),
  {
    withImages: false,
  },
)

const html = ref<string>('')

const disallowedTagNames = props.withImages ? [] : ['img', 'picture']

watch(
  () => props.value,
  async () => {
    // Markdown pipeline
    const result = await unified()
      // Parses markdown
      .use(remarkParse)
      // Support autolink literals, footnotes, strikethrough, tables and tasklists
      .use(remarkGfm)
      // Allows any HTML tags
      .use(remarkRehype, { allowDangerousHtml: true })
      // Creates a HTML AST
      .use(rehypeRaw)
      // Removes disallowed tags
      .use(rehypeSanitize, {
        ...defaultSchema,
        // Makes it even more strict
        tagNames: defaultSchema.tagNames?.filter(
          (tag) => !disallowedTagNames.includes(tag),
        ),
      })
      // Syntax highlighting
      .use(rehypeHighlight, {
        detect: true,
      })
      // Adds target="_blank" to external links
      .use(rehypeExternalLinks, { target: '_blank' })
      // Formats the HTML
      .use(rehypeFormat)
      // Converts the HTML AST to a string
      .use(rehypeStringify)
      // Run the pipeline
      .process(props.value)

    // Puts it into the DOM
    html.value = String(result.value)
  },
  { immediate: true },
)

// SSR hack - waits for the watch to complete
onServerPrefetch(async () => await sleep(1))
</script>

<template>
  <div
    class="markdown"
    v-html="html" />
</template>

<style scoped>
.markdown {
  color: var(--scalar-color-1);
  all: unset;
  word-break: break-word;
}
/* all elements inside .markdown, but not <details> and <summary> */
.markdown :deep(*) {
  all: unset;
  margin: 12px 0;
  font-family: var(--scalar-font);
  color: var(--scalar-color-1);
}
.markdown :deep(details) {
  margin: 12px 0;
  color: var(--scalar-color-1);
}
.markdown :deep(summary) {
  margin: 12px 0;
  padding-left: 20px;
  position: relative;
  font-weight: var(--scalar-semibold);
  cursor: pointer;
  user-select: none;
}

.markdown :deep(summary::after) {
  display: block;
  content: '';

  position: absolute;
  top: 1px;
  left: 1px;

  width: 16px;
  height: 16px;

  background-color: var(--scalar-color-3);
  mask-image: url('data:image/svg+xml,<svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M8.25 19.5L15.75 12L8.25 4.5" stroke="black" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/></svg>');
}

.markdown :deep(summary:hover::after) {
  background-color: var(--scalar-color-1);
}

.markdown :deep(details[open] summary::after) {
  transform: rotate(90deg);
}
.markdown :deep(img) {
  overflow: hidden;
  border-radius: var(--scalar-radius);
  max-width: 100%;
}
/* Don't add margin to the first block */
.markdown :deep(> :first-child) {
  margin-top: 0;
}
.markdown :deep(h1),
.markdown :deep(h2),
.markdown :deep(h3),
.markdown :deep(h4),
.markdown :deep(h5),
.markdown :deep(h6) {
  font-size: var(--font-size);
  margin: 18px 0 6px;
  font-weight: var(--scalar-bold);
  display: block;
  line-height: 1.45;
}
.markdown :deep(b),
.markdown :deep(strong) {
  font-weight: var(--scalar-bold);
}
.markdown :deep(p) {
  color: var(--scalar-color-1);
  font-weight: var(--font-weight, var(--scalar-regular));
  line-height: 1.5;
  margin-bottom: 0;
  display: block;
}

.markdown :deep(ul),
.markdown :deep(ol) {
  padding-left: 24px;
  line-height: 1.5;
  margin: 12px 0;
  display: block;
}

.markdown :deep(ul) {
  list-style: disc;
}

.markdown :deep(ol) {
  list-style: decimal;
}

.markdown :deep(ul.contains-task-list) {
  list-style: none;
  padding-left: 0;
}

.markdown :deep(li) {
  margin: 6px 0;
  display: list-item;
}
.markdown :deep(a) {
  color: var(--scalar-color-accent);
  text-decoration: var(--scalar-text-decoration);
  cursor: pointer;
}
.markdown :deep(a:hover) {
  text-decoration: var(--scalar-text-decoration-hover);
}
.markdown :deep(em) {
  font-style: italic;
}
.markdown :deep(del) {
  text-decoration: line-through;
}
.markdown :deep(code) {
  font-family: var(--scalar-font-code);
  background-color: var(--scalar-background-2);
  box-shadow: 0 0 0 1px var(--scalar-border-color);
  font-size: var(--scalar-micro);
  border-radius: 2px;
  padding: 0 3px;
}

.markdown :deep(pre code) {
  display: block;
  white-space: pre;
  padding: 12px;
  line-height: 1.5;
  margin: 12px 0;
  -webkit-overflow-scrolling: touch;
  overflow-x: auto;
  max-width: 100%;
  min-width: 100px;
}

.markdown :deep(blockquote) {
  border-left: 3px solid var(--scalar-border-color);
  padding-left: 12px;
  margin: 0;
  display: block;
}

.markdown :deep(table) {
  display: block;
  overflow-x: auto;
  position: relative;
  border-collapse: collapse;
  width: max-content;
  max-width: 100%;
  margin: 1em 0;
  box-shadow: 0 0 0 1px var(--scalar-border-color);
  border-radius: var(--scalar-radius-lg);
}
.markdown :deep(tbody) {
  display: table-row-group;
  vertical-align: middle;
}
.markdown :deep(thead) {
  display: table-header-group;
  vertical-align: middle;
}

.markdown :deep(tr) {
  display: table-row;
  border-color: inherit;
  vertical-align: inherit;
}
.markdown :deep(td),
.markdown :deep(th) {
  display: table-cell;
  vertical-align: inherit;
  min-width: 1em;
  padding: 6px 9px;
  vertical-align: top;
  line-height: 1.5;
  position: relative;
  word-break: initial;
  font-size: var(--scalar-small);
  color: var(--scalar-color-1);
  font-weight: var(--font-weight, var(--scalar-regular));
  border-right: 1px solid var(--scalar-border-color);
  border-bottom: 1px solid var(--scalar-border-color);
}

.markdown :deep(td > *),
.markdown :deep(th > *) {
  margin-bottom: 0;
}
.markdown :deep(th:empty) {
  display: none;
}
.markdown :deep(td:first-of-type),
.markdown :deep(th:first-of-type) {
  border-left: none;
}

.markdown :deep(td:last-of-type),
.markdown :deep(th:last-of-type) {
  border-right: none;
}

.markdown :deep(tr:last-of-type td) {
  border-bottom: none;
}

.markdown :deep(th) {
  font-weight: var(--scalar-semibold) !important;
  text-align: left;
  border-left-color: transparent;
  background: var(--scalar-background-2);
}

.markdown :deep(tr) > [align='left'] {
  text-align: left;
}
.markdown :deep(tr) > [align='right'] {
  text-align: right;
}
.markdown :deep(tr) > [align='center'] {
  text-align: center;
}
</style>

<style lang="postcss">
.markdown {
  pre code.hljs {
    display: block;
    overflow-x: auto;
    padding: 12px;
  }
  pre * {
    font-size: var(--scalar-small) !important;
    font-family: var(--scalar-font-code) !important;
  }
  code.hljs {
    padding: 3px 5px;
  }
  .hljs {
    background: var(--scalar-background-4);
    color: var(--scalar-color-1);
  }
  .hljs-comment,
  .hljs-quote {
    color: var(--scalar-color-3);
    font-style: italic;
  }
  .hljs-addition,
  .hljs-keyword,
  .hljs-literal,
  .hljs-selector-tag,
  .hljs-type {
    color: var(--scalar-color-green);
  }
  .hljs-number,
  .hljs-selector-attr,
  .hljs-selector-pseudo {
    color: var(--scalar-color-orange);
  }
  .hljs-doctag,
  .hljs-regexp,
  .hljs-string {
    color: var(--scalar-color-blue);
  }
  .hljs-built_in,
  .hljs-name,
  .hljs-section,
  .hljs-title {
    color: var(--scalar-color-1);
  }
  .hljs-class .hljs-title,
  .hljs-selector-id,
  .hljs-template-variable,
  .hljs-title.class_,
  .hljs-variable {
    color: var(--scalar-color-1);
  }
  .hljs-name,
  .hljs-section,
  .hljs-strong {
    font-weight: var(--scalar-semibold);
  }
  .hljs-bullet,
  .hljs-link,
  .hljs-meta,
  .hljs-subst,
  .hljs-symbol {
    color: var(--scalar-color-blue);
  }
  .hljs-deletion {
    color: var(--scalar-color-red);
  }
  .hljs-formula {
    background: var(--scalar-color-1);
  }
  .hljs-attr,
  .hljs-attribute {
    color: var(--scalar-color-1);
  }
  .hljs-emphasis {
    font-style: italic;
  }
}
</style>
