<script setup>
import { computed, ref } from "vue";
import protocolsData from "../../database/network-protocols.json";

const protocols = ref(protocolsData);
const query = ref("");
const selectedLayer = ref(null);

const layerOrder = ["Application", "Transport", "Internet", "Network Access"];
const colorByLayer = {
  "Application": "#e8952a",
  "Transport": "#2ead6c",
  "Internet": "#3a8dd6",
  "Network Access": "#c97c4a",
};

function primaryLayer(layer) {
  const l = (layer || "").toLowerCase();
  if (l.startsWith("application")) return "Application";
  if (l.startsWith("transport")) return "Transport";
  if (l.startsWith("internet")) return "Internet";
  if (l.startsWith("network access")) return "Network Access";
  return "Other";
}

function colorFor(layer) {
  return colorByLayer[primaryLayer(layer)] || "#8a8880";
}

const layers = computed(() => {
  const counts = new Map();
  for (const p of protocols.value) {
    const l = primaryLayer(p.layer);
    counts.set(l, (counts.get(l) || 0) + 1);
  }
  return layerOrder
    .filter((l) => counts.has(l))
    .map((name) => ({ name, count: counts.get(name) }));
});

const filtered = computed(() => {
  const q = query.value.trim().toLowerCase();
  const lay = selectedLayer.value;
  return protocols.value.filter((p) => {
    if (lay && primaryLayer(p.layer) !== lay) return false;
    if (!q) return true;
    return (
      p.name.toLowerCase().includes(q) ||
      (p.fullName || "").toLowerCase().includes(q) ||
      (p.layer || "").toLowerCase().includes(q) ||
      (p.transport || "").toLowerCase().includes(q) ||
      (p.ports || "").toLowerCase().includes(q) ||
      (p.description || "").toLowerCase().includes(q) ||
      (p.securityNotes || "").toLowerCase().includes(q) ||
      (p.useCases || []).some((u) => u.toLowerCase().includes(q))
    );
  });
});
</script>

<template>
  <div class="page-shell" style="--accent: #3a8dd6">
    <div class="page">
      <header class="hdr">
        <div class="icon">⇄</div>
        <div>
          <h1>Network Protocols</h1>
          <div class="sub">
            Reference for the protocols that carry traffic across the
            internet — sourced from
            <code>database/network-protocols.json</code>
          </div>
        </div>
      </header>

      <div class="toolbar">
        <input
          v-model="query"
          type="search"
          placeholder="Filter by name, layer, port, description, or use case…"
          autocomplete="off"
        />
        <span class="count">
          {{ filtered.length }} / {{ protocols.length }}
        </span>
      </div>

      <div class="layer-bar" role="group" aria-label="Filter by layer">
        <button
          type="button"
          class="layer-pill"
          :class="{ 'layer-pill--active': selectedLayer === null }"
          @click="selectedLayer = null"
        >
          All <span class="layer-pill__count">{{ protocols.length }}</span>
        </button>
        <button
          v-for="l in layers"
          :key="l.name"
          type="button"
          class="layer-pill"
          :class="{ 'layer-pill--active': selectedLayer === l.name }"
          :style="{ '--accent': colorByLayer[l.name] || '#8a8880' }"
          @click="selectedLayer = selectedLayer === l.name ? null : l.name"
        >
          {{ l.name }}
          <span class="layer-pill__count">{{ l.count }}</span>
        </button>
      </div>

      <div v-if="filtered.length === 0" class="empty">
        No protocols match the current filters
      </div>

      <div v-else class="proto-list">
        <article
          v-for="p in filtered"
          :key="p.name"
          class="proto"
          :style="{ '--accent': colorFor(p.layer) }"
        >
          <header class="proto__head">
            <div class="proto__title">
              <h2 class="proto__name">{{ p.name }}</h2>
              <div class="proto__full">{{ p.fullName }}</div>
            </div>
            <span class="proto__layer">{{ primaryLayer(p.layer) }}</span>
          </header>

          <div class="proto__meta">
            <div class="proto__meta-row">
              <span class="proto__meta-lbl">Layer</span>
              <span class="proto__meta-val">{{ p.layer }}</span>
            </div>
            <div class="proto__meta-row">
              <span class="proto__meta-lbl">Transport</span>
              <span class="proto__meta-val">
                <code>{{ p.transport }}</code>
              </span>
            </div>
            <div class="proto__meta-row">
              <span class="proto__meta-lbl">Ports</span>
              <span class="proto__meta-val">
                <code>{{ p.ports }}</code>
              </span>
            </div>
          </div>

          <p class="proto__desc">{{ p.description }}</p>

          <div v-if="p.securityNotes" class="proto__sec">
            <div class="proto__sec-lbl">Security notes</div>
            <p class="proto__sec-body">{{ p.securityNotes }}</p>
          </div>

          <div
            v-if="p.useCases && p.useCases.length"
            class="proto__uses"
          >
            <span class="proto__uses-lbl">Used for</span>
            <span
              v-for="u in p.useCases"
              :key="u"
              class="proto__use-pill"
            >
              {{ u }}
            </span>
          </div>
        </article>
      </div>

      <footer class="ft">
        {{ protocols.length }} protocols · sourced from
        <code>database/network-protocols.json</code>
      </footer>
    </div>
  </div>
</template>

<style scoped>
.layer-bar {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 18px;
}
.layer-pill {
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
.layer-pill:hover {
  color: var(--tx);
  border-color: var(--bdhi);
}
.layer-pill--active {
  background: color-mix(in srgb, var(--accent) 14%, transparent);
  color: var(--accent);
  border-color: var(--accent);
}
.layer-pill__count {
  font-family: var(--mono);
  font-size: 10px;
  color: var(--dm2);
  font-weight: 500;
}
.layer-pill--active .layer-pill__count {
  color: var(--accent);
}

.proto-list {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.proto {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 16px 18px;
  display: flex;
  flex-direction: column;
  gap: 12px;
  transition: border-color 0.12s ease;
}
.proto:hover {
  border-color: var(--bdhi);
  border-top-color: var(--accent);
}

.proto__head {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 14px;
  padding-bottom: 12px;
  border-bottom: 1px solid var(--bd);
  flex-wrap: wrap;
}
.proto__title {
  display: flex;
  flex-direction: column;
  gap: 2px;
  min-width: 0;
}
.proto__name {
  margin: 0;
  font-family: var(--sans);
  font-size: 17px;
  font-weight: 800;
  color: #fff;
  line-height: 1.2;
}
.proto__full {
  font-family: var(--sans);
  font-size: 11.5px;
  color: var(--dm);
  font-weight: 500;
}
.proto__layer {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  padding: 4px 9px;
  border-radius: 999px;
  background: color-mix(in srgb, var(--accent) 14%, transparent);
  color: var(--accent);
  border: 1px solid var(--accent);
  white-space: nowrap;
}

.proto__meta {
  display: grid;
  grid-template-columns: 1fr;
  gap: 4px;
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 8px;
  padding: 10px 12px;
}
.proto__meta-row {
  display: grid;
  grid-template-columns: 110px 1fr;
  gap: 12px;
  align-items: baseline;
  font-size: 12px;
}
@media (max-width: 540px) {
  .proto__meta-row {
    grid-template-columns: 1fr;
    gap: 2px;
  }
}
.proto__meta-lbl {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--dm);
}
.proto__meta-val {
  color: var(--tx);
}
.proto__meta-val code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 6px;
  font-size: 11.5px;
  color: #9cdcfe;
}

.proto__desc {
  margin: 0;
  color: var(--tx);
  font-size: 12.5px;
  line-height: 1.65;
}

.proto__sec {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-left: 2px solid #e84040;
  border-radius: 8px;
  padding: 10px 12px;
}
.proto__sec-lbl {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: #e84040;
  margin-bottom: 6px;
}
.proto__sec-body {
  margin: 0;
  color: var(--tx);
  font-size: 12px;
  line-height: 1.6;
}

.proto__uses {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 6px;
}
.proto__uses-lbl {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--dm);
  margin-right: 4px;
}
.proto__use-pill {
  font-family: var(--mono);
  font-size: 10.5px;
  color: var(--dm);
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 999px;
  padding: 3px 9px;
}
</style>
