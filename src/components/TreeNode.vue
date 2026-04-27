<script setup>
import { ref, computed } from "vue";

const props = defineProps({
  node: { type: Object, required: true },
  currentSlug: { type: String, default: "" },
  query: { type: String, default: "" },
  depth: { type: Number, default: 0 },
});

const hasChildren = computed(
  () => Array.isArray(props.node.children) && props.node.children.length > 0
);
const hasRoute = computed(() => !!(props.node.component || props.node.file));
const isHybrid = computed(() => hasChildren.value && hasRoute.value);
const isFolder = computed(() => hasChildren.value && !hasRoute.value);
const isLeaf = computed(() => !hasChildren.value && hasRoute.value);

const expanded = ref(true);

function matches(node, q) {
  if (!q) return true;
  if (node.file || node.component) {
    if (node.title.toLowerCase().includes(q)) return true;
  }
  if (Array.isArray(node.children))
    return node.children.some((c) => matches(c, q));
  return false;
}

const visibleChildren = computed(() => {
  if (!hasChildren.value) return [];
  const q = props.query.trim().toLowerCase();
  return q
    ? props.node.children.filter((c) => matches(c, q))
    : props.node.children;
});

const effectivelyExpanded = computed(() =>
  props.query ? visibleChildren.value.length > 0 : expanded.value
);

const titleMatchesQuery = computed(() => {
  const q = props.query.trim().toLowerCase();
  return q ? props.node.title.toLowerCase().includes(q) : true;
});

const shouldRender = computed(() => {
  if (!props.query) return true;
  if (titleMatchesQuery.value && hasRoute.value) return true;
  return visibleChildren.value.length > 0;
});
</script>

<template>
  <!-- Pure folder: only children, not directly navigable -->
  <div
    v-if="isFolder && shouldRender"
    class="tree__folder"
    :data-depth="depth"
  >
    <button
      type="button"
      class="tree__toggle"
      :style="{ paddingLeft: `${8 + depth * 12}px` }"
      @click="expanded = !expanded"
    >
      <span class="tree__caret" :class="{ open: effectivelyExpanded }">▶</span>
      <span class="tree__label">{{ node.title }}</span>
    </button>
    <div v-show="effectivelyExpanded" class="tree__children">
      <TreeNode
        v-for="(child, i) in visibleChildren"
        :key="child.slug || child.title + i"
        :node="child"
        :current-slug="currentSlug"
        :query="query"
        :depth="depth + 1"
      />
    </div>
  </div>

  <!-- Hybrid: clickable AND expandable -->
  <div
    v-else-if="isHybrid && shouldRender"
    class="tree__folder"
    :data-depth="depth"
  >
    <div
      class="tree__hybrid"
      :style="{ paddingLeft: `${8 + depth * 12}px` }"
    >
      <button
        type="button"
        class="tree__caret-btn"
        :aria-label="effectivelyExpanded ? 'Collapse' : 'Expand'"
        @click="expanded = !expanded"
      >
        <span class="tree__caret" :class="{ open: effectivelyExpanded }">▶</span>
      </button>
      <router-link
        class="tree__leaf tree__leaf--inline"
        :class="{ active: node.slug === currentSlug }"
        :to="{ name: 'page', params: { slug: node.slug } }"
      >
        {{ node.title }}
      </router-link>
    </div>
    <div v-show="effectivelyExpanded" class="tree__children">
      <TreeNode
        v-for="(child, i) in visibleChildren"
        :key="child.slug || child.title + i"
        :node="child"
        :current-slug="currentSlug"
        :query="query"
        :depth="depth + 1"
      />
    </div>
  </div>

  <!-- Pure leaf -->
  <router-link
    v-else-if="isLeaf"
    class="tree__leaf"
    :class="{ active: node.slug === currentSlug }"
    :to="{ name: 'page', params: { slug: node.slug } }"
    :style="{ paddingLeft: `${22 + depth * 12}px` }"
  >
    {{ node.title }}
  </router-link>
</template>
