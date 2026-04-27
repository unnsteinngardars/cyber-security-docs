<script setup>
import { computed, ref } from "vue";
import layersData from "../../database/osi-model.json";

const layers = ref(layersData);
const query = ref("");

const colorByLayer = {
  1: "#8a7558",
  2: "#c97c4a",
  3: "#3a8dd6",
  4: "#2ead6c",
  5: "#7c5cbf",
  6: "#d14c7a",
  7: "#e8952a",
};

function colorFor(n) {
  return colorByLayer[n] || "#8a8880";
}

const stack = computed(() =>
  [...layers.value].sort((a, b) => b.layer - a.layer)
);

const filtered = computed(() => {
  const q = query.value.trim().toLowerCase();
  if (!q) return stack.value;
  return stack.value.filter(
    (l) =>
      String(l.layer).includes(q) ||
      l.name.toLowerCase().includes(q) ||
      l.description.toLowerCase().includes(q) ||
      (l.examples || []).some((e) => e.toLowerCase().includes(q))
  );
});
</script>

<template>
  <div class="page-shell" style="--accent: #3a8dd6">
    <div class="page">
      <header class="hdr">
        <div class="icon">⇅</div>
        <div>
          <h1>OSI Model</h1>
          <div class="sub">
            The seven layers of network communication, from application down
            to physical — sourced from
            <code>database/osi-model.json</code>
          </div>
        </div>
      </header>

      <div class="callout">
        <strong>Mnemonic</strong>
        Top → bottom: <em>All People Seem To Need Data Processing.</em>
        Bottom → top: <em>Please Do Not Throw Sausage Pizza Away.</em>
      </div>

      <div class="toolbar">
        <input
          v-model="query"
          type="search"
          placeholder="Filter by layer, name, description, or example…"
          autocomplete="off"
        />
        <span class="count">{{ filtered.length }} / {{ layers.length }}</span>
      </div>

      <div v-if="filtered.length === 0" class="empty">
        No layers match "{{ query }}"
      </div>

      <div v-else class="stack">
        <article
          v-for="l in filtered"
          :key="l.layer"
          class="layer"
          :style="{ '--accent': colorFor(l.layer) }"
        >
          <div class="layer__rail">
            <div class="layer__num">L{{ l.layer }}</div>
            <div class="layer__pos">{{ l.layer }}/7</div>
          </div>
          <div class="layer__body">
            <header class="layer__head">
              <h2 class="layer__name">{{ l.name }}</h2>
              <span class="layer__sub">Layer {{ l.layer }}</span>
            </header>
            <p class="layer__desc">{{ l.description }}</p>
            <section v-if="l.examples && l.examples.length" class="layer__ex">
              <div class="layer__ex-title">Examples</div>
              <ul class="layer__list">
                <li v-for="e in l.examples" :key="e">{{ e }}</li>
              </ul>
            </section>
          </div>
        </article>
      </div>

      <footer class="ft">
        {{ layers.length }} layers · sourced from
        <code>database/osi-model.json</code>
      </footer>
    </div>
  </div>
</template>

<style scoped>
.callout {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  padding: 12px 14px;
  font-size: 11.5px;
  color: var(--dm);
  margin-bottom: 18px;
}
.callout strong {
  color: var(--accent);
  font-family: var(--sans);
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  display: block;
  margin-bottom: 4px;
}
.callout em {
  color: var(--tx);
  font-style: italic;
}

.stack {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.layer {
  display: grid;
  grid-template-columns: 72px 1fr;
  gap: 14px;
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 3px solid var(--accent);
  border-radius: 10px;
  padding: 14px 16px;
  transition: border-color 0.12s ease;
}
.layer:hover {
  border-color: var(--bdhi);
  border-left-color: var(--accent);
}
@media (max-width: 640px) {
  .layer {
    grid-template-columns: 1fr;
    gap: 8px;
  }
}

.layer__rail {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}
.layer__num {
  width: 56px;
  height: 56px;
  display: grid;
  place-items: center;
  font-family: var(--sans);
  font-size: 18px;
  font-weight: 800;
  letter-spacing: 0.04em;
  background: color-mix(in srgb, var(--accent) 14%, transparent);
  color: var(--accent);
  border: 1px solid var(--accent);
  border-radius: 10px;
}
.layer__pos {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 600;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--dm2);
}

.layer__body {
  display: flex;
  flex-direction: column;
  gap: 10px;
  min-width: 0;
}
.layer__head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
}
.layer__name {
  font-family: var(--sans);
  font-size: 17px;
  font-weight: 700;
  color: #fff;
  margin: 0;
  line-height: 1.2;
}
.layer__sub {
  font-family: var(--sans);
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--dm);
}
.layer__desc {
  margin: 0;
  color: var(--tx);
  font-size: 12.5px;
  line-height: 1.65;
}

.layer__ex {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 8px;
  padding: 10px 12px;
}
.layer__ex-title {
  font-family: var(--sans);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
  margin-bottom: 8px;
}
.layer__list {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 5px;
}
.layer__list li {
  font-size: 11.5px;
  color: var(--dm);
  padding-left: 14px;
  position: relative;
  line-height: 1.55;
}
.layer__list li::before {
  content: "›";
  position: absolute;
  left: 0;
  color: var(--accent);
}
</style>
