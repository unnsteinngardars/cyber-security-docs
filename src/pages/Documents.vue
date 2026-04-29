<script setup>
import { ref, computed } from "vue";
import documentsData from "../../database/documents.json";

const documents = ref(documentsData);
const query = ref("");
const selectedCategory = ref(null);
const selectedType = ref(null);

const categories = computed(() => {
  const counts = new Map();
  for (const d of documents.value) {
    for (const c of d.categories || []) {
      counts.set(c, (counts.get(c) || 0) + 1);
    }
  }
  return [...counts.entries()]
    .sort((a, b) => a[0].localeCompare(b[0]))
    .map(([name, count]) => ({ name, count }));
});

const types = computed(() => {
  const counts = new Map();
  for (const d of documents.value) {
    if (!d.type) continue;
    counts.set(d.type, (counts.get(d.type) || 0) + 1);
  }
  return [...counts.entries()]
    .sort((a, b) => a[0].localeCompare(b[0]))
    .map(([name, count]) => ({ name, count }));
});

const filtered = computed(() => {
  const q = query.value.trim().toLowerCase();
  const cat = selectedCategory.value;
  const type = selectedType.value;
  return documents.value.filter((d) => {
    const cats = d.categories || [];
    if (cat && !cats.includes(cat)) return false;
    if (type && d.type !== type) return false;
    if (!q) return true;
    return (
      d.name.toLowerCase().includes(q) ||
      cats.some((c) => c.toLowerCase().includes(q)) ||
      (d.publisher || "").toLowerCase().includes(q) ||
      (d.shortDescription || "").toLowerCase().includes(q) ||
      (d.longDescription || "").toLowerCase().includes(q)
    );
  });
});

const typeLabel = {
  pdf: "PDF",
  html: "Web",
  rfc: "RFC",
  book: "Book",
  standard: "Standard",
};
</script>

<template>
  <div class="page-shell">
    <div class="page">
      <header class="hdr">
        <div class="icon">📄</div>
        <div>
          <h1>Documents</h1>
          <div class="sub">
            External references — standards, RFCs, threat reports, training and PDFs — sourced from
            <code>database/documents.json</code>
          </div>
        </div>
      </header>

      <div class="toolbar">
        <input
          v-model="query"
          type="search"
          placeholder="Filter by name, category, publisher, or description…"
          autocomplete="off"
        />
        <span class="count">
          {{ filtered.length }} / {{ documents.length }}
        </span>
      </div>

      <div class="cat-bar" role="group" aria-label="Filter by category">
        <button
          type="button"
          class="cat-pill"
          :class="{ 'cat-pill--active': selectedCategory === null }"
          @click="selectedCategory = null"
        >
          All <span class="cat-pill__count">{{ documents.length }}</span>
        </button>
        <button
          v-for="c in categories"
          :key="c.name"
          type="button"
          class="cat-pill"
          :class="{ 'cat-pill--active': selectedCategory === c.name }"
          @click="selectedCategory = selectedCategory === c.name ? null : c.name"
        >
          {{ c.name }} <span class="cat-pill__count">{{ c.count }}</span>
        </button>
      </div>

      <div class="cat-bar cat-bar--types" role="group" aria-label="Filter by type">
        <button
          type="button"
          class="cat-pill cat-pill--type"
          :class="{ 'cat-pill--active': selectedType === null }"
          @click="selectedType = null"
        >
          Any type
        </button>
        <button
          v-for="t in types"
          :key="t.name"
          type="button"
          class="cat-pill cat-pill--type"
          :class="{ 'cat-pill--active': selectedType === t.name }"
          @click="selectedType = selectedType === t.name ? null : t.name"
        >
          {{ typeLabel[t.name] || t.name }}
          <span class="cat-pill__count">{{ t.count }}</span>
        </button>
      </div>

      <div v-if="filtered.length === 0" class="empty">
        No documents match the current filters
      </div>

      <div v-else class="docs">
        <article v-for="d in filtered" :key="d.name" class="dcard">
          <div class="dcard__head">
            <div class="dcard__title">
              <a
                class="dcard__name"
                :href="d.url"
                target="_blank"
                rel="noopener noreferrer"
              >
                {{ d.name }}
              </a>
              <div class="dcard__meta">
                <span v-if="d.type" class="dcard__type">
                  {{ typeLabel[d.type] || d.type }}
                </span>
                <span v-if="d.publisher" class="dcard__pub">{{ d.publisher }}</span>
                <span v-if="d.year" class="dcard__year">{{ d.year }}</span>
              </div>
              <div v-if="d.categories && d.categories.length" class="dcard__cats">
                <button
                  v-for="c in d.categories"
                  :key="c"
                  type="button"
                  class="dcard__cat"
                  :class="{ 'dcard__cat--active': selectedCategory === c }"
                  @click="selectedCategory = selectedCategory === c ? null : c"
                  :title="`Filter by ${c}`"
                >
                  {{ c }}
                </button>
              </div>
            </div>
            <div class="dcard__value">
              {{ d.shortDescription }}
            </div>
          </div>
          <details v-if="d.longDescription">
            <summary>More details</summary>
            <div class="dcard__body">
              <p>{{ d.longDescription }}</p>
            </div>
          </details>
        </article>
      </div>

      <footer class="ft">
        {{ documents.length }} documents · sourced from
        <code>database/documents.json</code>
      </footer>
    </div>
  </div>
</template>

<style scoped>
.cat-bar {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 12px;
}
.cat-bar--types {
  margin-bottom: 16px;
}
.cat-pill {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 5px 10px;
  background: var(--sf);
  color: var(--dm);
  border: 1px solid var(--bd);
  border-radius: 999px;
  font-family: var(--sans);
  font-size: 10.5px;
  font-weight: 600;
  letter-spacing: 0.04em;
  cursor: pointer;
  transition: border-color 0.12s ease, color 0.12s ease, background 0.12s ease;
}
.cat-pill:hover {
  color: var(--tx);
  border-color: var(--bdhi);
}
.cat-pill--active {
  background: color-mix(in srgb, var(--accent) 14%, transparent);
  color: var(--accent);
  border-color: var(--accent);
}
.cat-pill__count {
  font-family: var(--mono);
  font-size: 10px;
  color: var(--dm2);
  font-weight: 500;
}
.cat-pill--active .cat-pill__count {
  color: var(--accent);
}
.cat-pill--type {
  text-transform: uppercase;
  letter-spacing: 0.08em;
  font-size: 10px;
}

.docs {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.dcard {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  overflow: hidden;
  transition: border-color 0.12s ease;
}
.dcard:hover {
  border-color: var(--bdhi);
  border-left-color: var(--accent);
}

.dcard__head {
  display: grid;
  grid-template-columns: minmax(220px, 320px) 1fr;
  gap: 16px;
  align-items: start;
  padding: 12px 14px;
}
@media (max-width: 720px) {
  .dcard__head {
    grid-template-columns: 1fr;
    gap: 6px;
  }
}

.dcard__title {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 6px;
}

.dcard__name {
  font-family: var(--sans);
  font-size: 13.5px;
  font-weight: 700;
  color: #fff;
  text-decoration: none;
  display: inline-flex;
  align-items: baseline;
  gap: 6px;
  line-height: 1.35;
}
.dcard__name::after {
  content: "↗";
  color: var(--dm2);
  font-size: 11px;
  transition: color 0.12s ease;
}
.dcard__name:hover {
  color: var(--accent);
}
.dcard__name:hover::after {
  color: var(--accent);
}

.dcard__meta {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 6px;
  font-family: var(--sans);
  font-size: 10px;
  color: var(--dm2);
}
.dcard__type {
  background: color-mix(in srgb, var(--accent) 14%, transparent);
  color: var(--accent);
  border: 1px solid var(--accent);
  border-radius: 999px;
  padding: 1px 7px;
  font-size: 9.5px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.08em;
}
.dcard__pub {
  font-weight: 600;
  color: var(--dm);
}
.dcard__year {
  font-family: var(--mono);
}
.dcard__pub + .dcard__year::before {
  content: "·";
  margin-right: 6px;
  color: var(--dm2);
}

.dcard__cats {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.dcard__cat {
  background: transparent;
  color: var(--dm);
  border: 1px solid var(--bd);
  border-radius: 999px;
  padding: 2px 8px;
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  cursor: pointer;
  transition: border-color 0.12s ease, color 0.12s ease, background 0.12s ease;
}
.dcard__cat:hover {
  color: var(--tx);
  border-color: var(--bdhi);
}
.dcard__cat--active {
  background: color-mix(in srgb, var(--accent) 14%, transparent);
  color: var(--accent);
  border-color: var(--accent);
}

.dcard__value {
  color: var(--dm);
  font-size: 12px;
  line-height: 1.5;
}

.dcard details {
  border-top: 1px solid var(--bd);
}
.dcard details[open] {
  background: var(--sf2);
}
.dcard summary {
  list-style: none;
  cursor: pointer;
  padding: 8px 14px;
  font-family: var(--sans);
  font-size: 10.5px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--dm);
  display: flex;
  align-items: center;
  gap: 6px;
}
.dcard summary::-webkit-details-marker {
  display: none;
}
.dcard summary::before {
  content: "▶";
  font-size: 9px;
  color: var(--dm2);
  transition: transform 0.12s ease;
  display: inline-block;
}
.dcard details[open] summary::before {
  transform: rotate(90deg);
}
.dcard summary:hover {
  color: var(--tx);
}

.dcard__body {
  padding: 4px 14px 14px 28px;
  color: var(--tx);
  font-size: 12px;
  line-height: 1.6;
}
.dcard__body p {
  margin: 0;
}
</style>
