<script setup>
import { ref, computed } from "vue";
import articlesData from "../../database/articles.json";

const articles = ref(articlesData);
const query = ref("");
const selectedCategory = ref(null);

const categories = computed(() => {
  const counts = new Map();
  for (const a of articles.value) {
    for (const c of a.categories || []) {
      counts.set(c, (counts.get(c) || 0) + 1);
    }
  }
  return [...counts.entries()]
    .sort((a, b) => a[0].localeCompare(b[0]))
    .map(([name, count]) => ({ name, count }));
});

const filtered = computed(() => {
  const q = query.value.trim().toLowerCase();
  const cat = selectedCategory.value;
  return articles.value.filter((a) => {
    const cats = a.categories || [];
    if (cat && !cats.includes(cat)) return false;
    if (!q) return true;
    return (
      a.name.toLowerCase().includes(q) ||
      cats.some((c) => c.toLowerCase().includes(q)) ||
      (a.author || "").toLowerCase().includes(q) ||
      (a.publisher || "").toLowerCase().includes(q) ||
      (a.shortDescription || "").toLowerCase().includes(q) ||
      (a.longDescription || "").toLowerCase().includes(q)
    );
  });
});
</script>

<template>
  <div class="page-shell">
    <div class="page">
      <header class="hdr">
        <div class="icon">🔗</div>
        <div>
          <h1>Articles</h1>
          <div class="sub">
            Online resources — blog posts, write-ups, papers, and curated links — sourced from
            <code>database/articles.json</code>
          </div>
        </div>
      </header>

      <div class="toolbar">
        <input
          v-model="query"
          type="search"
          placeholder="Filter by name, category, author, publisher, or description…"
          autocomplete="off"
        />
        <span class="count">
          {{ filtered.length }} / {{ articles.length }}
        </span>
      </div>

      <div class="cat-bar" role="group" aria-label="Filter by category">
        <button
          type="button"
          class="cat-pill"
          :class="{ 'cat-pill--active': selectedCategory === null }"
          @click="selectedCategory = null"
        >
          All <span class="cat-pill__count">{{ articles.length }}</span>
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

      <div v-if="filtered.length === 0" class="empty">
        No articles match the current filters
      </div>

      <div v-else class="docs">
        <article v-for="a in filtered" :key="a.name" class="dcard">
          <div class="dcard__head">
            <div class="dcard__title">
              <a
                class="dcard__name"
                :href="a.url"
                target="_blank"
                rel="noopener noreferrer"
              >
                {{ a.name }}
              </a>
              <div class="dcard__meta">
                <span v-if="a.author" class="dcard__author">{{ a.author }}</span>
                <span v-if="a.publisher" class="dcard__pub">{{ a.publisher }}</span>
                <span v-if="a.year" class="dcard__year">{{ a.year }}</span>
              </div>
              <div v-if="a.categories && a.categories.length" class="dcard__cats">
                <button
                  v-for="c in a.categories"
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
              {{ a.shortDescription }}
            </div>
          </div>
          <details v-if="a.longDescription">
            <summary>More details</summary>
            <div class="dcard__body">
              <p>{{ a.longDescription }}</p>
            </div>
          </details>
        </article>
      </div>

      <footer class="ft">
        {{ articles.length }} articles · sourced from
        <code>database/articles.json</code>
      </footer>
    </div>
  </div>
</template>

<style scoped>
.cat-bar {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
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
.dcard__author {
  font-weight: 700;
  color: var(--tx);
}
.dcard__pub {
  font-weight: 600;
  color: var(--dm);
}
.dcard__author + .dcard__pub::before {
  content: "·";
  margin-right: 6px;
  color: var(--dm2);
}
.dcard__year {
  font-family: var(--mono);
}
.dcard__pub + .dcard__year::before,
.dcard__author + .dcard__year::before {
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
