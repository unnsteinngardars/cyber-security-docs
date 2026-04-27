<script setup>
import { computed, ref } from "vue";
import layersData from "../../database/tcp-ip-model.json";

const layers = ref(layersData);
const query = ref("");

const colorByLayer = {
  1: "#c97c4a",
  2: "#3a8dd6",
  3: "#2ead6c",
  4: "#e8952a",
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
      (l.osiMapping || "").toLowerCase().includes(q) ||
      (l.examples || []).some((e) => e.toLowerCase().includes(q)) ||
      (l.headers || []).some(
        (h) =>
          h.name.toLowerCase().includes(q) ||
          (h.fields || []).some(
            (f) =>
              f.name.toLowerCase().includes(q) ||
              f.description.toLowerCase().includes(q)
          )
      )
  );
});
</script>

<template>
  <div class="page-shell" style="--accent: #2ead6c">
    <div class="page">
      <header class="hdr">
        <div class="icon">≣</div>
        <div>
          <h1>TCP/IP Model</h1>
          <div class="sub">
            The four-layer reference model that actually runs the internet —
            sourced from <code>database/tcp-ip-model.json</code>
          </div>
        </div>
      </header>

      <div class="callout">
        <strong>Mnemonic</strong>
        Top → bottom: <em>All Tigers Ignore Newts.</em>
        Bottom → top: <em>Newts In The Attic.</em>
      </div>

      <div class="callout">
        <strong>OSI vs TCP/IP</strong>
        TCP/IP is the practical model behind real-world networking; OSI is
        the teaching model. TCP/IP folds OSI's seven layers into four —
        merging Physical + Data Link into <em>Network Access</em>, and
        Session + Presentation + Application into <em>Application</em>.
      </div>

      <div class="toolbar">
        <input
          v-model="query"
          type="search"
          placeholder="Filter by layer, name, OSI mapping, or example…"
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
            <div class="layer__pos">{{ l.layer }}/4</div>
          </div>
          <div class="layer__body">
            <header class="layer__head">
              <h2 class="layer__name">{{ l.name }}</h2>
              <span class="layer__sub">Layer {{ l.layer }}</span>
            </header>
            <div v-if="l.osiMapping" class="layer__map">
              <span class="layer__map-label">OSI mapping</span>
              <span class="layer__map-value">{{ l.osiMapping }}</span>
            </div>
            <p class="layer__desc">{{ l.description }}</p>
            <section v-if="l.examples && l.examples.length" class="layer__ex">
              <div class="layer__ex-title">Examples</div>
              <ul class="layer__list">
                <li v-for="e in l.examples" :key="e">{{ e }}</li>
              </ul>
            </section>

            <details v-if="l.headers && l.headers.length" class="layer__hdrs">
              <summary class="layer__hdrs-sum">
                <span class="layer__hdrs-title">Headers</span>
                <span class="layer__hdrs-count">
                  {{ l.headers.length }}
                  {{ l.headers.length === 1 ? "block" : "blocks" }}
                </span>
              </summary>
              <div class="layer__hdrs-body">
                <div
                  v-for="h in l.headers"
                  :key="h.name"
                  class="hdr-block"
                >
                  <div class="hdr-block__head">
                    <span class="hdr-block__name">{{ h.name }}</span>
                    <span class="hdr-block__size">{{ h.size }}</span>
                  </div>
                  <div class="hdr-block__table-wrap">
                    <table class="hdr-block__tbl">
                      <thead>
                        <tr>
                          <th>Field</th>
                          <th>Size</th>
                          <th>Description</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="f in h.fields" :key="f.name">
                          <td><code>{{ f.name }}</code></td>
                          <td class="size">{{ f.size }}</td>
                          <td>{{ f.description }}</td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </div>
              </div>
            </details>
          </div>
        </article>
      </div>

      <footer class="ft">
        {{ layers.length }} layers · sourced from
        <code>database/tcp-ip-model.json</code>
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

.layer__map {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
  padding: 6px 10px;
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 5px;
  font-size: 11.5px;
}
.layer__map-label {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
}
.layer__map-value {
  color: var(--tx);
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

/* Headers section (collapsible) */
.layer__hdrs {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 8px;
  overflow: hidden;
}
.layer__hdrs-sum {
  list-style: none;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 12px;
}
.layer__hdrs-sum::-webkit-details-marker {
  display: none;
}
.layer__hdrs-sum::before {
  content: "▶";
  font-size: 9px;
  color: var(--dm2);
  transition: transform 0.12s ease;
  display: inline-block;
}
.layer__hdrs[open] .layer__hdrs-sum::before {
  transform: rotate(90deg);
}
.layer__hdrs-sum:hover .layer__hdrs-title {
  color: #fff;
}
.layer__hdrs-title {
  flex: 1;
  font-family: var(--sans);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
  transition: color 0.12s ease;
}
.layer__hdrs-count {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 600;
  letter-spacing: 0.06em;
  color: var(--dm2);
}

.layer__hdrs-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 10px;
  border-top: 1px solid var(--bd);
}

.hdr-block {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  overflow: hidden;
}
.hdr-block__head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 8px 12px;
  flex-wrap: wrap;
}
.hdr-block__name {
  flex: 1;
  font-family: var(--sans);
  font-size: 12px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
}
.hdr-block__size {
  font-family: var(--mono);
  font-size: 10.5px;
  color: var(--accent);
  white-space: nowrap;
}

.hdr-block__table-wrap {
  overflow-x: auto;
  border-top: 1px solid var(--bd);
  background: var(--sf);
}
.hdr-block__tbl {
  width: 100%;
  border-collapse: collapse;
  font-size: 11.5px;
}
.hdr-block__tbl th,
.hdr-block__tbl td {
  text-align: left;
  padding: 6px 10px;
  border-bottom: 1px solid var(--bd);
  vertical-align: top;
  color: var(--tx);
}
.hdr-block__tbl thead th {
  background: var(--sf2);
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--dm);
}
.hdr-block__tbl tbody tr:last-child td {
  border-bottom: 0;
}
.hdr-block__tbl td.size {
  font-family: var(--mono);
  font-size: 10.5px;
  color: var(--dm);
  white-space: nowrap;
}
.hdr-block__tbl code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 10.5px;
  color: #9cdcfe;
  white-space: nowrap;
}
</style>
