<script setup>
import { ref, watch } from "vue";
import { marked } from "marked";

const props = defineProps({
  file: { type: String, required: true },
});

const srcdoc = ref(null);
const error = ref(null);

function isMarkdown(path) {
  return /\.(md|markdown)$/i.test(path);
}

function wrap(html) {
  return `<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500;700&family=Syne:wght@400;500;700;800&display=swap" rel="stylesheet" />
    <style>
      body {
        font: 15px/1.65 system-ui, -apple-system, "Segoe UI", sans-serif;
        max-width: 820px;
        margin: 32px auto;
        padding: 0 24px 64px;
        color: #1f2330;
        background: #fff;
      }
      h1, h2, h3, h4, h5, h6 {
        font-family: "Syne", system-ui, sans-serif;
        line-height: 1.25;
        margin: 1.6em 0 0.5em;
      }
      h1 { font-size: 28px; border-bottom: 1px solid #eaeaea; padding-bottom: 8px; }
      h2 { font-size: 22px; }
      h3 { font-size: 18px; }
      h4 { font-size: 15px; }
      p { margin: 0.6em 0; }
      a { color: #2b6cb0; }
      code {
        background: #f3f4f7;
        border: 1px solid #e5e7eb;
        border-radius: 4px;
        padding: 1px 5px;
        font-size: 0.92em;
        font-family: "JetBrains Mono", ui-monospace, monospace;
      }
      pre {
        background: #0c0d0f;
        color: #e8e6e0;
        padding: 14px 16px;
        border-radius: 8px;
        overflow-x: auto;
        font-family: "JetBrains Mono", ui-monospace, monospace;
        font-size: 13px;
        line-height: 1.55;
      }
      pre code { background: transparent; border: 0; padding: 0; color: inherit; }
      blockquote {
        border-left: 3px solid #d8d8d8;
        margin: 1em 0;
        padding: 0.2em 1em;
        color: #555;
      }
      ul, ol { padding-left: 1.4em; }
      li { margin: 0.2em 0; }
      hr { border: 0; border-top: 1px solid #eaeaea; margin: 2em 0; }
      table { border-collapse: collapse; margin: 1em 0; }
      th, td { border: 1px solid #e5e7eb; padding: 6px 10px; text-align: left; }
      th { background: #f8f9fb; }
      img { max-width: 100%; }
    </style>
  </head>
  <body>${html}</body>
</html>`;
}

watch(
  () => props.file,
  async (file) => {
    error.value = null;
    if (!isMarkdown(file)) {
      srcdoc.value = null;
      return;
    }
    try {
      const res = await fetch(file, { cache: "no-store" });
      if (!res.ok) throw new Error(`${file}: ${res.status}`);
      const md = await res.text();
      srcdoc.value = wrap(marked.parse(md));
    } catch (err) {
      error.value = err.message;
      srcdoc.value = wrap(
        `<pre style="color:#b00020">Failed to load ${file}: ${err.message}</pre>`
      );
    }
  },
  { immediate: true }
);
</script>

<template>
  <iframe
    v-if="srcdoc !== null"
    :srcdoc="srcdoc"
    title="Page viewer"
    sandbox="allow-same-origin allow-scripts allow-popups allow-forms"
  />
  <iframe
    v-else
    :src="file"
    title="Page viewer"
    sandbox="allow-same-origin allow-scripts allow-popups allow-forms"
  />
</template>

<style scoped>
iframe {
  display: block;
  width: 100%;
  height: 100%;
  min-height: 100vh;
  border: 0;
  background: #fff;
}
</style>
