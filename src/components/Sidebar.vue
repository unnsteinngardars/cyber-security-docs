<script setup>
import { ref, computed, watch } from "vue";
import TreeNode from "./TreeNode.vue";

const props = defineProps({
  tree: { type: Array, required: true },
  currentSlug: { type: String, default: "" },
  error: { type: String, default: null },
});

defineEmits(["toggle"]);

const query = ref("");
const openKey = ref("");

function hasMatch(node, q) {
  if ((node.file || node.component) && node.title.toLowerCase().includes(q))
    return true;
  if (Array.isArray(node.children))
    return node.children.some((c) => hasMatch(c, q));
  return false;
}

function containsSlug(node, slug) {
  if (!slug) return false;
  if (node.slug === slug) return true;
  if (Array.isArray(node.children))
    return node.children.some((c) => containsSlug(c, slug));
  return false;
}

function keyOf(node) {
  return node.slug || node.title;
}

const visibleRoots = computed(() => {
  const q = query.value.trim().toLowerCase();
  return q ? props.tree.filter((n) => hasMatch(n, q)) : props.tree;
});

watch(
  [() => props.currentSlug, () => props.tree],
  ([slug, tree]) => {
    if (!slug || !tree.length) return;
    const node = tree.find((n) => containsSlug(n, slug));
    if (node) openKey.value = keyOf(node);
  },
  { immediate: true }
);

function toggleRoot(node) {
  const k = keyOf(node);
  openKey.value = openKey.value === k ? "" : k;
}
</script>

<template>
  <aside>
    <header>
      <h1>Cyber Security Resources</h1>
      <input v-model="query" type="search" placeholder="Filter pages…" autocomplete="off" />
    </header>
    <nav aria-label="Pages">
      <div v-if="error" class="sidebar-message">
        Failed to load manifest.json<br /><small>{{ error }}</small>
      </div>
      <div v-else-if="query && visibleRoots.length === 0" class="sidebar-message">
        No pages match "{{ query }}"
      </div>
      <ul v-else class="tree-list">
        <TreeNode
          v-for="node in visibleRoots"
          :key="keyOf(node)"
          :node="node"
          :current-slug="currentSlug"
          :query="query"
          :is-open="openKey === keyOf(node)"
          @toggle="toggleRoot(node)"
        />
      </ul>
    </nav>
    <footer>
      <button type="button" class="sidebar-toggle" aria-label="Toggle sidebar" @click="$emit('toggle')">
        &#9776;
      </button>
    </footer>
  </aside>
</template>
