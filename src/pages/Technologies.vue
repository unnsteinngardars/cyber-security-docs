<script setup>
import { ref, computed } from "vue";
import technologiesData from "../../database/technologies.json";

const technologies = ref(technologiesData);
const query = ref("");

const filtered = computed(() => {
  const q = query.value.trim().toLowerCase();
  if (!q) return technologies.value;
  return technologies.value.filter((t) => {
    return (
      t.name.toLowerCase().includes(q) ||
      (t.fullName || "").toLowerCase().includes(q) ||
      (t.description || "").toLowerCase().includes(q)
    );
  });
});
</script>

<template>
  <div class="page-shell">
    <div class="page">
      <header class="hdr">
        <div class="icon">🛡</div>
        <div>
          <h1>Technologies</h1>
          <div class="sub">
            Common security technology categories — sourced from
            <code>database/technologies.json</code>
          </div>
        </div>
      </header>

      <div class="toolbar">
        <input
          v-model="query"
          type="search"
          placeholder="Filter by name or description…"
          autocomplete="off"
        />
        <span class="count">
          {{ filtered.length }} / {{ technologies.length }}
        </span>
      </div>

      <div v-if="filtered.length === 0" class="empty">
        No technologies match the current filter
      </div>

      <div v-else class="techs">
        <article v-for="t in filtered" :key="t.name" class="kcard">
          <div class="kcard__head">
            <div class="kcard__title">
              <span class="kcard__name">{{ t.name }}</span>
              <span v-if="t.fullName" class="kcard__full">{{ t.fullName }}</span>
            </div>
            <div class="kcard__value">{{ t.description }}</div>
          </div>
        </article>
      </div>

      <footer class="ft">
        {{ technologies.length }} technologies · sourced from
        <code>database/technologies.json</code>
      </footer>
    </div>
  </div>
</template>

<style scoped>
.techs {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.kcard {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  overflow: hidden;
  transition: border-color 0.12s ease;
}
.kcard:hover {
  border-color: var(--bdhi);
  border-left-color: var(--accent);
}

.kcard__head {
  display: grid;
  grid-template-columns: minmax(180px, 260px) 1fr;
  gap: 16px;
  align-items: start;
  padding: 12px 14px;
}
@media (max-width: 720px) {
  .kcard__head {
    grid-template-columns: 1fr;
    gap: 6px;
  }
}

.kcard__title {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  gap: 4px;
}

.kcard__name {
  font-family: var(--sans);
  font-size: 14px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
}

.kcard__full {
  font-family: var(--sans);
  font-size: 10.5px;
  font-weight: 600;
  color: var(--accent);
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

.kcard__value {
  color: var(--dm);
  font-size: 12px;
  line-height: 1.55;
}
</style>
