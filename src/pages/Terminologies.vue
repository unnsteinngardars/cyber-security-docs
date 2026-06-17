<script setup>
import { ref, computed } from "vue";
import terminologiesData from "../../database/terminologies.json";

const terms = ref(terminologiesData);
const query = ref("");
const selectedCategory = ref(null);

const categories = computed(() => {
  const counts = new Map();
  for (const t of terms.value) {
    for (const c of t.categories || []) {
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
  return terms.value.filter((t) => {
    const cats = t.categories || [];
    if (cat && !cats.includes(cat)) return false;
    if (!q) return true;
    return (
      t.name.toLowerCase().includes(q) ||
      cats.some((c) => c.toLowerCase().includes(q)) ||
      (t.shortDescription || "").toLowerCase().includes(q) ||
      (t.longDescription || "").toLowerCase().includes(q) ||
      (t.example || "").toLowerCase().includes(q)
    );
  });
});
</script>

<template>
  <div class="page-shell">
    <div class="page">
      <header class="hdr">
        <div class="icon">𝐀</div>
        <div>
          <h1>Terminologies</h1>
          <div class="sub">
            Glossary of cybersecurity terms — sourced from
            <code>database/terminologies.json</code>
          </div>
        </div>
      </header>

      <div class="toolbar">
        <input
          v-model="query"
          type="search"
          placeholder="Filter by term, category, or definition…"
          autocomplete="off"
        />
        <span class="count">
          {{ filtered.length }} / {{ terms.length }}
        </span>
      </div>

      <div class="cat-bar" role="group" aria-label="Filter by category">
        <button
          type="button"
          class="cat-pill"
          :class="{ 'cat-pill--active': selectedCategory === null }"
          @click="selectedCategory = null"
        >
          All <span class="cat-pill__count">{{ terms.length }}</span>
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
        No terms match the current filters
      </div>

      <div v-else class="terms">
        <article v-for="t in filtered" :key="t.name" class="tcard">
          <div class="tcard__head">
            <div class="tcard__title">
              <span class="tcard__name">{{ t.name }}</span>
              <div v-if="t.categories && t.categories.length" class="tcard__cats">
                <button
                  v-for="c in t.categories"
                  :key="c"
                  type="button"
                  class="tcard__cat"
                  :class="{ 'tcard__cat--active': selectedCategory === c }"
                  @click="selectedCategory = selectedCategory === c ? null : c"
                  :title="`Filter by ${c}`"
                >
                  {{ c }}
                </button>
              </div>
            </div>
            <div class="tcard__value">
              {{ t.shortDescription }}
            </div>
          </div>
          <details v-if="t.longDescription || t.example">
            <summary>More details</summary>
            <div class="tcard__body">
              <p v-if="t.longDescription">{{ t.longDescription }}</p>
              <div v-if="t.example" class="example">
                <div class="example__lbl">Example</div>
                <p class="example__body">{{ t.example }}</p>
              </div>
            </div>
          </details>
        </article>
      </div>

      <footer class="ft">
        {{ terms.length }} terms · sourced from
        <code>database/terminologies.json</code>
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

.terms {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.tcard {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  overflow: hidden;
  transition: border-color 0.12s ease;
}
.tcard:hover {
  border-color: var(--bdhi);
  border-left-color: var(--accent);
}

.tcard__head {
  display: grid;
  grid-template-columns: minmax(160px, 220px) 1fr;
  gap: 16px;
  align-items: start;
  padding: 12px 14px;
}
@media (max-width: 640px) {
  .tcard__head {
    grid-template-columns: 1fr;
    gap: 6px;
  }
}

.tcard__title {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
}

.tcard__name {
  font-family: var(--sans);
  font-size: 14px;
  font-weight: 700;
  color: var(--hi);
}

.tcard__cats {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.tcard__cat {
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
.tcard__cat:hover {
  color: var(--tx);
  border-color: var(--bdhi);
}
.tcard__cat--active {
  background: color-mix(in srgb, var(--accent) 14%, transparent);
  color: var(--accent);
  border-color: var(--accent);
}

.tcard__value {
  color: var(--dm);
  font-size: 12px;
  line-height: 1.5;
}

.tcard details {
  border-top: 1px solid var(--bd);
}
.tcard details[open] {
  background: var(--sf2);
}
.tcard summary {
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
.tcard summary::-webkit-details-marker {
  display: none;
}
.tcard summary::before {
  content: "▶";
  font-size: 9px;
  color: var(--dm2);
  transition: transform 0.12s ease;
  display: inline-block;
}
.tcard details[open] summary::before {
  transform: rotate(90deg);
}
.tcard summary:hover {
  color: var(--tx);
}

.tcard__body {
  padding: 4px 14px 14px 28px;
  color: var(--tx);
  font-size: 12px;
  line-height: 1.6;
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.tcard__body p {
  margin: 0;
}

.example {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 6px;
  padding: 10px 12px;
}
.example__lbl {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--accent);
  margin-bottom: 6px;
}
.example__body {
  font-size: 12px;
  line-height: 1.6;
  color: var(--tx);
  font-style: italic;
}
</style>
