<script setup>
import { ref, computed } from "vue";
import data from "../../database/cryptography.json";

const types = data.types;
const primitives = data.primitives;

const filter = ref("all");

const typeFilters = [
  { id: "all", label: "All" },
  { id: "Asymmetric", label: "Asymmetric" },
  { id: "Asymmetric construct", label: "Signatures" },
  { id: "Trust binding", label: "Certificates" },
  { id: "Hybrid cryptosystem", label: "Hybrid" },
  { id: "OpenPGP implementation", label: "Tooling" }
];

const visiblePrimitives = computed(() => {
  if (filter.value === "all") return primitives;
  return primitives.filter((p) => p.type === filter.value);
});
</script>

<template>
  <div class="page-shell" style="--accent: #3ab9b0">
    <div class="page">
      <header class="hdr">
        <div class="icon">⚿</div>
        <div>
          <h1>Cryptography</h1>
          <div class="sub">
            The two families of crypto and the primitives that build the
            modern web of trust — RSA, Diffie-Hellman, signatures,
            certificates, PGP, and GPG.
          </div>
        </div>
      </header>

      <div class="callout">
        <strong>Mental model</strong>
        <em>Symmetric</em> crypto is a key in a lock — fast, but you both
        need a copy. <em>Asymmetric</em> crypto is a slot mailbox — anyone
        can drop a letter in (encrypt), only the person with the back-door
        key can take letters out (decrypt). Real systems combine both:
        asymmetric to bootstrap, symmetric for the bulk data.
      </div>

      <!-- Symmetric vs Asymmetric -->
      <section class="section">
        <h2>The two families</h2>
        <p class="prose">
          All practical cryptography sits in one of two camps — or a
          carefully designed hybrid of both.
        </p>
        <div class="type-grid">
          <article
            v-for="t in types"
            :key="t.name"
            class="tcard"
            :style="{ '--accent': t.name === 'Symmetric' ? '#e8952a' : '#3ab9b0' }"
          >
            <header class="tcard__head">
              <h3>{{ t.name }}</h3>
              <span class="tcard__tag">{{ t.tagline }}</span>
            </header>
            <p class="tcard__desc">{{ t.description }}</p>

            <div class="tcard__list">
              <span class="tcard__lbl">Examples</span>
              <div class="chiprow">
                <code v-for="ex in t.examples" :key="ex" class="chip">{{ ex }}</code>
              </div>
            </div>

            <div class="split">
              <div class="split__col split__col--good">
                <h4>Strengths</h4>
                <ul>
                  <li v-for="p in t.pros" :key="p">{{ p }}</li>
                </ul>
              </div>
              <div class="split__col split__col--bad">
                <h4>Weaknesses</h4>
                <ul>
                  <li v-for="c in t.cons" :key="c">{{ c }}</li>
                </ul>
              </div>
            </div>
          </article>
        </div>
      </section>

      <!-- Primitives -->
      <section class="section">
        <h2>Primitives &amp; tooling</h2>
        <p class="prose">
          The building blocks you'll actually meet — algorithms (RSA, DH),
          a construct (signatures), a trust system (certificates), and
          two real tools (PGP and GPG) that wrap them up for end users.
        </p>

        <div class="filterbar" role="group" aria-label="Filter">
          <button
            v-for="f in typeFilters"
            :key="f.id"
            type="button"
            class="pill"
            :class="{ 'pill--active': filter === f.id }"
            @click="filter = f.id"
          >
            {{ f.label }}
          </button>
        </div>

        <div class="prim-stack">
          <article v-for="p in visiblePrimitives" :key="p.name" class="pcard">
            <header class="pcard__head">
              <div>
                <h3>{{ p.name }}</h3>
                <div class="pcard__full">{{ p.fullName }}</div>
              </div>
              <span class="pcard__type">{{ p.type }}</span>
            </header>

            <p class="pcard__summary">{{ p.summary }}</p>

            <div class="pcard__block">
              <span class="pcard__lbl">How it works</span>
              <p>{{ p.howItWorks }}</p>
            </div>

            <div v-if="p.fields" class="pcard__block">
              <span class="pcard__lbl">Key fields in an X.509 cert</span>
              <ul class="bulletlist">
                <li v-for="f in p.fields" :key="f">{{ f }}</li>
              </ul>
            </div>

            <div v-if="p.algorithms" class="pcard__block">
              <span class="pcard__lbl">Common algorithms</span>
              <div class="chiprow">
                <code v-for="a in p.algorithms" :key="a" class="chip">{{ a }}</code>
              </div>
            </div>

            <div class="pcard__block">
              <span class="pcard__lbl">Used for</span>
              <div class="chiprow">
                <span v-for="u in p.uses" :key="u" class="usechip">{{ u }}</span>
              </div>
            </div>

            <div class="split">
              <div class="split__col split__col--good">
                <h4>Strengths</h4>
                <ul>
                  <li v-for="s in p.strengths" :key="s">{{ s }}</li>
                </ul>
              </div>
              <div class="split__col split__col--bad">
                <h4>Weaknesses</h4>
                <ul>
                  <li v-for="w in p.weaknesses" :key="w">{{ w }}</li>
                </ul>
              </div>
            </div>
          </article>
        </div>
      </section>

      <footer class="ft">
        Reference data sourced from <code>database/cryptography.json</code>
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
  margin-bottom: 24px;
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

.section {
  margin-bottom: 32px;
}
.section h2 {
  font-family: var(--sans);
  font-size: 15px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
  margin: 0 0 12px;
}

.prose {
  color: var(--tx);
  font-size: 12.5px;
  line-height: 1.7;
  margin: 0 0 12px;
}
.prose code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11.5px;
  color: #9cdcfe;
}
.prose strong {
  color: #fff;
}
.prose em {
  color: var(--accent);
  font-style: normal;
}

/* Type cards (symmetric vs asymmetric) */
.type-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 14px;
}
.tcard {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 3px solid var(--accent);
  border-radius: 12px;
  padding: 14px 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.tcard__head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 12px;
  flex-wrap: wrap;
}
.tcard__head h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 16px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
}
.tcard__tag {
  font-family: var(--sans);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: var(--accent);
}
.tcard__desc {
  margin: 0;
  font-size: 12px;
  line-height: 1.6;
  color: var(--tx);
}
.tcard__list {
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.tcard__lbl {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--dm);
}

/* Filter pills */
.filterbar {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 14px;
}
.pill {
  background: var(--sf);
  color: var(--dm);
  border: 1px solid var(--bd);
  border-radius: 999px;
  padding: 5px 12px;
  font-family: var(--sans);
  font-size: 10.5px;
  font-weight: 600;
  letter-spacing: 0.04em;
  cursor: pointer;
  transition: border-color 0.12s ease, color 0.12s ease, background 0.12s ease;
}
.pill:hover {
  color: var(--tx);
  border-color: var(--bdhi);
}
.pill--active {
  background: color-mix(in srgb, var(--accent) 14%, transparent);
  color: var(--accent);
  border-color: var(--accent);
}

/* Primitives */
.prim-stack {
  display: flex;
  flex-direction: column;
  gap: 14px;
}
.pcard {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 3px solid var(--accent);
  border-radius: 12px;
  padding: 14px 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.pcard__head {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 12px;
  flex-wrap: wrap;
}
.pcard__head h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 18px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
}
.pcard__full {
  font-size: 11.5px;
  color: var(--dm);
  margin-top: 2px;
  font-style: italic;
}
.pcard__type {
  font-family: var(--sans);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
  border: 1px solid var(--accent);
  border-radius: 999px;
  padding: 3px 10px;
  background: color-mix(in srgb, var(--accent) 10%, transparent);
  white-space: nowrap;
}
.pcard__summary {
  margin: 0;
  font-size: 12.5px;
  line-height: 1.6;
  color: var(--tx);
  font-weight: 500;
}
.pcard__block {
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.pcard__block p {
  margin: 0;
  font-size: 12px;
  color: var(--tx);
  line-height: 1.65;
}
.pcard__lbl {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
}

/* Chip rows */
.chiprow {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}
.chip {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 5px;
  padding: 2px 7px;
  font-family: var(--mono);
  font-size: 11px;
  color: var(--accent);
}
.usechip {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 999px;
  padding: 3px 9px;
  font-family: var(--sans);
  font-size: 10.5px;
  color: var(--tx);
}

/* Pros / Cons split */
.split {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin-top: 4px;
}
@media (max-width: 700px) {
  .split {
    grid-template-columns: 1fr;
  }
}
.split__col {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 8px;
  padding: 10px 12px;
}
.split__col h4 {
  margin: 0 0 6px;
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}
.split__col--good {
  border-left: 2px solid #2ead6c;
}
.split__col--good h4 {
  color: #2ead6c;
}
.split__col--bad {
  border-left: 2px solid #d14c4c;
}
.split__col--bad h4 {
  color: #d14c4c;
}
.split__col ul {
  margin: 0;
  padding-left: 16px;
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.split__col li {
  font-size: 11.5px;
  color: var(--tx);
  line-height: 1.6;
}

/* Plain bullet list */
.bulletlist {
  margin: 0;
  padding-left: 18px;
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.bulletlist li {
  font-size: 11.5px;
  color: var(--tx);
  line-height: 1.6;
}
</style>
