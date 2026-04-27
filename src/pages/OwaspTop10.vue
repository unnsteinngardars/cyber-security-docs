<script setup>
import { computed, ref } from "vue";
import risksData from "../../database/owasp-top-10.json";

const risks = ref(risksData);
const query = ref("");

const colorByNumber = {
  A01: "#e84040",
  A05: "#e8952a",
  A06: "#1d9e8a",
  A02: "#d14c7a",
  A04: "#2ead6c",
  A07: "#7c5cbf",
  A09: "#3a8dd6",
};

function colorFor(num) {
  return colorByNumber[num] || "#8a8880";
}

const iaaa = [
  {
    letter: "I",
    name: "Identity",
    tagline: "The claim",
    body: "Who you say you are. A username, email, employee ID, or client certificate subject — just an assertion, not yet verified.",
    question: '"Who are you?"',
    accent: "#3a8dd6",
  },
  {
    letter: "A",
    name: "Authentication",
    tagline: "The proof",
    body: "Proving the claim — something you know, have, or are. Passwords, TOTP, hardware keys, biometrics. MFA combines factors.",
    question: '"Prove it."',
    accent: "#7c5cbf",
  },
  {
    letter: "A",
    name: "Authorisation",
    tagline: "The permission",
    body: "What the proven identity is allowed to do. Role checks, ACLs, ownership checks — enforced on every sensitive request, not just at login.",
    question: '"What can you do?"',
    accent: "#2ead6c",
  },
  {
    letter: "A",
    name: "Accountability",
    tagline: "The record",
    body: "Every action is attributable to a specific identity through logs and audit trails. Without it, neither investigation nor non-repudiation is possible.",
    question: '"Who did what, and when?"',
    accent: "#e8952a",
  },
];

const filtered = computed(() => {
  const q = query.value.trim().toLowerCase();
  if (!q) return risks.value;
  return risks.value.filter(
    (r) =>
      r.number.toLowerCase().includes(q) ||
      r.name.toLowerCase().includes(q) ||
      r.description.toLowerCase().includes(q) ||
      r.commonPatterns.some((p) => p.toLowerCase().includes(q)) ||
      r.howToPrevent.some((p) => p.toLowerCase().includes(q))
  );
});
</script>

<template>
  <div class="page-shell" style="--accent: #f26522">
    <div class="page">
      <header class="hdr">
        <div class="icon">⚠</div>
        <div>
          <h1>OWASP Top 10</h1>
          <div class="sub">
            The most critical web-application security risks — sourced from
            <code>database/owasp-top-10.json</code>
          </div>
        </div>
      </header>

      <section class="section">
        <h2>IAAA — Identity, Authentication, Authorisation, Accountability</h2>
        <div class="iaaa">
          <article v-for="p in iaaa" :key="p.name" class="pillar" :style="{ '--accent': p.accent }">
            <div class="pillar__head">
              <div class="pillar__badge">{{ p.letter }}</div>
              <div>
                <div class="pillar__title">{{ p.name }}</div>
                <div class="pillar__tagline">{{ p.tagline }}</div>
              </div>
            </div>
            <div class="pillar__body">{{ p.body }}</div>
            <div class="pillar__question">{{ p.question }}</div>
          </article>
        </div>
      </section>

      <section class="section">
        <div class="section-head">
          <h2>The Risks</h2>
          <div class="toolbar">
            <input v-model="query" type="search" placeholder="Filter risks…" autocomplete="off" />
            <span class="count">{{ filtered.length }} / {{ risks.length }}</span>
          </div>
        </div>

        <div v-if="filtered.length === 0" class="empty">
          No risks match "{{ query }}"
        </div>

        <article v-for="r in filtered" :key="r.number" class="risk" :style="{ '--accent': colorFor(r.number) }">
          <header class="risk__head">
            <div class="risk__code">{{ r.number }}</div>
            <div class="risk__head-text">
              <div class="risk__title">{{ r.name }}</div>
              <a class="risk__link" :href="r.link" target="_blank" rel="noopener noreferrer">
                owasp.org ↗
              </a>
            </div>
          </header>

          <p class="risk__body">{{ r.description }}</p>

          <div class="split">
            <section class="split__col split__col--problem">
              <div class="split__title">Common Patterns</div>
              <ul class="risk__list">
                <li v-for="p in r.commonPatterns" :key="p">{{ p }}</li>
              </ul>
            </section>
            <section class="split__col split__col--fix">
              <div class="split__title">How to Prevent</div>
              <ul class="risk__list">
                <li v-for="p in r.howToPrevent" :key="p">{{ p }}</li>
              </ul>
            </section>
          </div>
        </article>
      </section>

      <footer class="ft">
        {{ risks.length }} risks · sourced from
        <code>database/owasp-top-10.json</code>
      </footer>
    </div>
  </div>
</template>

<style scoped>
.page-shell {
  --owasp: #f26522;
  --teal: #1d9e8a;
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

.section-head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 14px;
  margin-bottom: 14px;
  flex-wrap: wrap;
}

.section-head h2 {
  margin: 0;
}

/* IAAA pillars */
.iaaa {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
}

@media (max-width: 900px) {
  .iaaa {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 520px) {
  .iaaa {
    grid-template-columns: 1fr;
  }
}

.pillar {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 16px 16px 14px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.pillar__head {
  display: flex;
  align-items: center;
  gap: 10px;
}

.pillar__badge {
  width: 34px;
  height: 34px;
  flex-shrink: 0;
  border-radius: 8px;
  display: grid;
  place-items: center;
  font-family: var(--sans);
  font-size: 14px;
  font-weight: 800;
  background: color-mix(in srgb, var(--accent) 12%, transparent);
  border: 1px solid var(--accent);
  color: var(--accent);
}

.pillar__title {
  font-family: var(--sans);
  font-size: 14px;
  font-weight: 700;
  color: #fff;
  line-height: 1.15;
}

.pillar__tagline {
  font-size: 10.5px;
  color: var(--dm);
  text-transform: uppercase;
  letter-spacing: 0.08em;
  margin-top: 2px;
}

.pillar__body {
  color: var(--tx);
  font-size: 12.5px;
  line-height: 1.6;
}

.pillar__question {
  font-size: 11px;
  color: var(--dm);
  padding: 6px 8px;
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 5px;
  font-style: italic;
}

/* Risk cards */
.risk {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 12px;
  padding: 18px 20px 18px;
  margin-bottom: 18px;
  display: flex;
  flex-direction: column;
  gap: 14px;
}

.risk__head {
  display: flex;
  align-items: flex-start;
  gap: 14px;
  padding-bottom: 14px;
  border-bottom: 1px solid var(--bd);
}

.risk__code {
  font-family: var(--sans);
  font-size: 14px;
  font-weight: 800;
  padding: 8px 12px;
  border-radius: 8px;
  flex-shrink: 0;
  letter-spacing: 0.04em;
  background: color-mix(in srgb, var(--accent) 12%, transparent);
  color: var(--accent);
  border: 1px solid var(--accent);
}

.risk__head-text {
  display: flex;
  flex-direction: column;
  gap: 4px;
  min-width: 0;
}

.risk__title {
  font-family: var(--sans);
  font-size: 17px;
  font-weight: 700;
  color: #fff;
  line-height: 1.25;
}

.risk__link {
  font-size: 11px;
  color: var(--dm);
  text-decoration: none;
  font-family: var(--sans);
  letter-spacing: 0.04em;
}

.risk__link:hover {
  color: var(--accent);
}

.risk__body {
  color: var(--tx);
  font-size: 12.5px;
  line-height: 1.65;
  margin: 0;
}

/* Two-column patterns / prevention */
.split {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}

@media (max-width: 720px) {
  .split {
    grid-template-columns: 1fr;
  }
}

.split__col {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 8px;
  padding: 12px 14px;
}

.split__col--problem {
  border-left: 2px solid var(--owasp);
}

.split__col--fix {
  border-left: 2px solid var(--teal);
}

.split__title {
  font-family: var(--sans);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  margin-bottom: 10px;
}

.split__col--problem .split__title {
  color: var(--owasp);
}

.split__col--fix .split__title {
  color: var(--teal);
}

.risk__list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 6px;
  padding: 0;
  margin: 0;
}

.risk__list li {
  font-size: 11.5px;
  color: var(--dm);
  padding-left: 14px;
  position: relative;
  line-height: 1.55;
}

.risk__list li::before {
  content: "›";
  position: absolute;
  left: 0;
}

.split__col--problem .risk__list li::before {
  color: var(--owasp);
}

.split__col--fix .risk__list li::before {
  color: var(--teal);
}
</style>
