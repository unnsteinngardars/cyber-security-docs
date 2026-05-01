<script setup>
import { ref, computed, watch } from "vue";

const props = defineProps({
  node: { type: Object, required: true },
  currentSlug: { type: String, default: "" },
  query: { type: String, default: "" },
  depth: { type: Number, default: 0 },
  isOpen: { type: Boolean, default: false },
});

const emit = defineEmits(["toggle"]);

const hasChildren = computed(
  () => Array.isArray(props.node.children) && props.node.children.length > 0
);
const hasRoute = computed(() => !!(props.node.component || props.node.file));
const isHybrid = computed(() => hasChildren.value && hasRoute.value);
const isFolder = computed(() => hasChildren.value && !hasRoute.value);
const isLeaf = computed(() => !hasChildren.value && hasRoute.value);

function matches(node, q) {
  if (!q) return true;
  if ((node.file || node.component) && node.title.toLowerCase().includes(q))
    return true;
  if (Array.isArray(node.children))
    return node.children.some((c) => matches(c, q));
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

const visibleChildren = computed(() => {
  if (!hasChildren.value) return [];
  const q = props.query.trim().toLowerCase();
  return q
    ? props.node.children.filter((c) => matches(c, q))
    : props.node.children;
});

const expanded = computed(() => {
  if (props.query) return visibleChildren.value.length > 0;
  return props.isOpen;
});

const titleMatchesQuery = computed(() => {
  const q = props.query.trim().toLowerCase();
  return q ? props.node.title.toLowerCase().includes(q) : true;
});

const shouldRender = computed(() => {
  if (!props.query) return true;
  if (titleMatchesQuery.value && hasRoute.value) return true;
  return visibleChildren.value.length > 0;
});

const openChildKey = ref("");

watch(
  () => props.currentSlug,
  (slug) => {
    if (!hasChildren.value || !slug) return;
    const child = props.node.children.find((c) => containsSlug(c, slug));
    if (child) openChildKey.value = keyOf(child);
  },
  { immediate: true }
);

function toggle() {
  emit("toggle");
}

function toggleChild(node) {
  const k = keyOf(node);
  openChildKey.value = openChildKey.value === k ? "" : k;
}

const indentPx = computed(() => props.depth * 14 + 8);
</script>

<template>
  <li v-if="(isFolder || isHybrid) && shouldRender" class="tree-item">
    <div class="tree-row" :style="{ paddingLeft: indentPx + 'px' }">
      <button
        type="button"
        class="tree-caret-btn"
        :aria-expanded="expanded"
        :aria-label="expanded ? 'Collapse' : 'Expand'"
        @click="toggle"
      >
        <span class="tree-caret" :class="{ 'is-open': expanded }">▶</span>
      </button>
      <button
        v-if="isFolder"
        type="button"
        class="tree-folder"
        @click="toggle"
      >
        {{ node.title }}
      </button>
      <router-link
        v-else
        class="tree-link"
        :to="{ name: 'page', params: { slug: node.slug } }"
      >
        {{ node.title }}
      </router-link>
    </div>
    <div class="tree-children-wrap" :class="{ 'is-open': expanded }">
      <div class="tree-children-inner">
        <ul class="tree-children">
          <TreeNode
            v-for="child in visibleChildren"
            :key="keyOf(child)"
            :node="child"
            :current-slug="currentSlug"
            :query="query"
            :depth="depth + 1"
            :is-open="openChildKey === keyOf(child)"
            @toggle="toggleChild(child)"
          />
        </ul>
      </div>
    </div>
  </li>

  <li v-else-if="isLeaf" class="tree-item">
    <router-link
      class="tree-link tree-leaf"
      :to="{ name: 'page', params: { slug: node.slug } }"
      :style="{ paddingLeft: indentPx + 22 + 'px' }"
    >
      {{ node.title }}
    </router-link>
  </li>
</template>
