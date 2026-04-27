<script setup>
import { computed, ref } from "vue";
import protocolsData from "../../database/routing-protocols.json";

const protocols = ref(protocolsData);

const colorByName = {
  OSPF: "#3a8dd6",
  EIGRP: "#2ead6c",
  BGP: "#7c5cbf",
  RIP: "#c97c4a",
};

function colorFor(name) {
  return colorByName[name] || "#8a8880";
}

const adminDistanceTable = [
  { source: "Connected interface", ad: "0" },
  { source: "Static route",        ad: "1" },
  { source: "eBGP",                ad: "20" },
  { source: "EIGRP (internal)",    ad: "90" },
  { source: "OSPF",                ad: "110" },
  { source: "RIP",                 ad: "120" },
  { source: "EIGRP (external)",    ad: "170" },
  { source: "iBGP",                ad: "200" },
  { source: "Unknown / unreachable", ad: "255" }
];

const compareCols = computed(() => protocols.value);
</script>

<template>
  <div class="page-shell" style="--accent: #3a8dd6">
    <div class="page">
      <header class="hdr">
        <div class="icon">⤳</div>
        <div>
          <h1>Routing</h1>
          <div class="sub">
            How routers decide where to forward packets, and the four
            protocols (OSPF, EIGRP, BGP, RIP) that build the routes —
            sourced from <code>database/routing-protocols.json</code>
          </div>
        </div>
      </header>

      <div class="callout">
        <strong>Mental model</strong>
        Forwarding is the router asking <em>"where do I send this packet
        next?"</em> for every datagram. Routing is the background process
        that fills in the answer ahead of time — exchanging information
        with other routers so the answer is always ready and (mostly)
        correct.
      </div>

      <!-- How routing works -->
      <section class="section">
        <h2>How routing works</h2>
        <p class="prose">
          Every router holds a <strong>routing table</strong> — a list of
          destination prefixes (e.g. <code>10.0.5.0/24</code>) paired with
          a <em>next hop</em> and outgoing interface. When a packet
          arrives, the router does a <strong>longest-prefix match</strong>
          on the destination IP against the table, picks the most specific
          entry, and forwards the packet toward that next hop.
        </p>
        <p class="prose">
          The table itself is filled three ways: <strong>directly
          connected</strong> routes (the router knows about its own
          interfaces), <strong>static</strong> routes (an admin typed
          them in), and <strong>dynamic</strong> routes (a routing
          protocol learned them from a neighbour). Routing protocols are
          the dynamic case — they let routers swap reachability
          information so the network adapts to failures and new links
          without human intervention.
        </p>
      </section>

      <!-- IGP vs EGP -->
      <section class="section">
        <h2>IGP vs EGP — the two-tier hierarchy</h2>
        <p class="prose">
          The internet is a network of networks — each independently
          operated network is an <strong>Autonomous System (AS)</strong>,
          identified by a globally unique AS number. Routing splits along
          that boundary:
        </p>
        <div class="split">
          <div class="split__col">
            <div class="split__title">IGP — Interior Gateway Protocol</div>
            <p>
              Runs <em>inside</em> a single AS. Optimised for fast
              convergence and trust between routers under one
              administration. <strong>OSPF, EIGRP, RIP</strong> are IGPs.
            </p>
          </div>
          <div class="split__col">
            <div class="split__title">EGP — Exterior Gateway Protocol</div>
            <p>
              Runs <em>between</em> ASes. Optimised for stability,
              policy, and scale across mutually distrustful networks.
              <strong>BGP</strong> is the only EGP in modern use; it's
              also used inside an AS as iBGP.
            </p>
          </div>
        </div>
      </section>

      <!-- Algorithm families -->
      <section class="section">
        <h2>Three algorithm families</h2>
        <p class="prose">
          Routing protocols differ in <em>what</em> they advertise and
          <em>how</em> they compute paths. Three families cover all four
          protocols on this page.
        </p>
        <div class="families">
          <article class="family" style="--accent: #c97c4a">
            <div class="family__head">
              <span class="family__badge">DV</span>
              <h3>Distance-Vector</h3>
            </div>
            <p>
              Each router tells its neighbours <em>"here are the
              destinations I can reach, and how far away they are"</em> —
              its entire routing table. Neighbours add their own cost and
              propagate further. Simple, but prone to loops and
              count-to-infinity without safeguards.
            </p>
            <div class="family__ex">RIP · (EIGRP — advanced variant)</div>
          </article>
          <article class="family" style="--accent: #3a8dd6">
            <div class="family__head">
              <span class="family__badge">LS</span>
              <h3>Link-State</h3>
            </div>
            <p>
              Each router floods a description of its directly connected
              links to every other router. Every router builds the
              <em>same</em> map of the topology and independently runs
              Dijkstra to compute shortest paths. Fast convergence and
              loop-free by construction.
            </p>
            <div class="family__ex">OSPF · IS-IS</div>
          </article>
          <article class="family" style="--accent: #7c5cbf">
            <div class="family__head">
              <span class="family__badge">PV</span>
              <h3>Path-Vector</h3>
            </div>
            <p>
              Each route carries the <em>complete path</em> of ASes it
              has traversed. Loops are detected by simply checking whether
              your own AS is already in the path. Path selection is
              policy-driven, not metric-driven — built for inter-AS
              politics, not raw shortest path.
            </p>
            <div class="family__ex">BGP</div>
          </article>
        </div>
      </section>

      <!-- Comparison table -->
      <section class="section">
        <h2>At a glance</h2>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Protocol</th>
                <th>Type</th>
                <th>Scope</th>
                <th>Metric</th>
                <th>AD</th>
                <th>Convergence</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="p in compareCols" :key="p.name">
                <td>
                  <code class="hl" :style="{ '--accent': colorFor(p.name) }">
                    {{ p.name }}
                  </code>
                </td>
                <td>{{ p.type }}</td>
                <td>{{ p.scope }}</td>
                <td>{{ p.metric }}</td>
                <td><code>{{ p.adminDistance }}</code></td>
                <td>{{ p.convergence }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Per-protocol cards -->
      <section class="section">
        <h2>The protocols</h2>
        <article
          v-for="p in protocols"
          :key="p.name"
          class="proto"
          :style="{ '--accent': colorFor(p.name) }"
        >
          <header class="proto__head">
            <div class="proto__title">
              <h3 class="proto__name">{{ p.name }}</h3>
              <div class="proto__full">{{ p.fullName }}</div>
            </div>
            <div class="proto__tags">
              <span class="proto__tag">{{ p.type }}</span>
              <span class="proto__tag proto__tag--scope">{{ p.scope }}</span>
            </div>
          </header>

          <div class="proto__meta">
            <div class="proto__meta-row">
              <span class="proto__meta-lbl">Transport</span>
              <code>{{ p.transport }}</code>
            </div>
            <div class="proto__meta-row">
              <span class="proto__meta-lbl">Metric</span>
              <span>{{ p.metric }}</span>
            </div>
            <div class="proto__meta-row">
              <span class="proto__meta-lbl">Admin distance</span>
              <code>{{ p.adminDistance }}</code>
            </div>
            <div class="proto__meta-row">
              <span class="proto__meta-lbl">Convergence</span>
              <span>{{ p.convergence }}</span>
            </div>
            <div class="proto__meta-row">
              <span class="proto__meta-lbl">IPv6</span>
              <code>{{ p.ipv6 }}</code>
            </div>
          </div>

          <p class="proto__summary">{{ p.summary }}</p>

          <div class="proto__hiw">
            <div class="proto__hiw-lbl">How it works</div>
            <p>{{ p.howItWorks }}</p>
          </div>

          <div class="sw">
            <section class="sw__col sw__col--strong">
              <div class="sw__title">Strengths</div>
              <ul>
                <li v-for="s in p.strengths" :key="s">{{ s }}</li>
              </ul>
            </section>
            <section class="sw__col sw__col--weak">
              <div class="sw__title">Weaknesses</div>
              <ul>
                <li v-for="w in p.weaknesses" :key="w">{{ w }}</li>
              </ul>
            </section>
          </div>
        </article>
      </section>

      <!-- Administrative distance reference -->
      <section class="section">
        <h2>Administrative distance — tie-breaker between sources</h2>
        <p class="prose">
          When the same prefix is learned from multiple sources, the router
          installs the route with the <strong>lowest administrative
          distance</strong>. Lower = more trusted. Cisco-defined values are
          the de-facto standard.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Route source</th>
                <th>Default AD</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="r in adminDistanceTable" :key="r.source">
                <td>{{ r.source }}</td>
                <td><code>{{ r.ad }}</code></td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <footer class="ft">
        Reference data sourced from
        <code>database/routing-protocols.json</code>
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

/* Two-column split */
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
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  padding: 12px 14px;
  font-size: 12px;
  color: var(--tx);
  line-height: 1.6;
}
.split__col p {
  margin: 0;
}
.split__col strong {
  color: #fff;
}
.split__col em {
  color: var(--accent);
  font-style: normal;
}
.split__title {
  font-family: var(--sans);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
  margin-bottom: 8px;
}

/* Algorithm families */
.families {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}
@media (max-width: 900px) {
  .families {
    grid-template-columns: 1fr;
  }
}
.family {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 14px 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.family__head {
  display: flex;
  align-items: center;
  gap: 10px;
}
.family__badge {
  width: 32px;
  height: 32px;
  display: grid;
  place-items: center;
  font-family: var(--sans);
  font-size: 12px;
  font-weight: 800;
  background: color-mix(in srgb, var(--accent) 14%, transparent);
  color: var(--accent);
  border: 1px solid var(--accent);
  border-radius: 8px;
}
.family h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 14px;
  font-weight: 700;
  color: #fff;
}
.family p {
  margin: 0;
  color: var(--tx);
  font-size: 12px;
  line-height: 1.6;
}
.family em {
  color: var(--accent);
  font-style: normal;
}
.family__ex {
  font-family: var(--mono);
  font-size: 11px;
  color: var(--dm);
  padding-top: 8px;
  border-top: 1px solid var(--bd);
}

/* Tables */
.table-wrap {
  overflow-x: auto;
  border: 1px solid var(--bd);
  border-radius: 8px;
  background: var(--sf);
}
.tbl {
  width: 100%;
  border-collapse: collapse;
  font-size: 12px;
}
.tbl th,
.tbl td {
  text-align: left;
  padding: 8px 12px;
  border-bottom: 1px solid var(--bd);
  vertical-align: top;
  color: var(--tx);
}
.tbl thead th {
  background: var(--sf2);
  font-family: var(--sans);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--dm);
}
.tbl tbody tr:last-child td {
  border-bottom: 0;
}
.tbl tbody tr:hover {
  background: var(--sf2);
}
.tbl code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
  white-space: nowrap;
}
.tbl code.hl {
  color: var(--accent);
  border-color: var(--accent);
  font-weight: 700;
}

/* Per-protocol cards */
.proto {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 16px 18px;
  margin-bottom: 14px;
  display: flex;
  flex-direction: column;
  gap: 12px;
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
}
.proto__name {
  margin: 0;
  font-family: var(--sans);
  font-size: 18px;
  font-weight: 800;
  color: #fff;
  letter-spacing: 0.02em;
}
.proto__full {
  font-family: var(--sans);
  font-size: 11.5px;
  color: var(--dm);
}
.proto__tags {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}
.proto__tag {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  padding: 4px 9px;
  border-radius: 999px;
  background: color-mix(in srgb, var(--accent) 14%, transparent);
  color: var(--accent);
  border: 1px solid var(--accent);
  white-space: nowrap;
}
.proto__tag--scope {
  background: var(--sf2);
  color: var(--dm);
  border-color: var(--bd);
}

.proto__meta {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 8px;
  padding: 10px 12px;
  display: flex;
  flex-direction: column;
  gap: 4px;
}
.proto__meta-row {
  display: grid;
  grid-template-columns: 130px 1fr;
  gap: 12px;
  align-items: baseline;
  font-size: 12px;
  color: var(--tx);
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
.proto__meta-row code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 6px;
  font-size: 11.5px;
  color: #9cdcfe;
  justify-self: start;
  white-space: nowrap;
}

.proto__summary {
  margin: 0;
  color: var(--tx);
  font-size: 12.5px;
  line-height: 1.65;
}

.proto__hiw {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  padding: 10px 12px;
}
.proto__hiw-lbl {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
  margin-bottom: 6px;
}
.proto__hiw p {
  margin: 0;
  color: var(--tx);
  font-size: 12px;
  line-height: 1.65;
}

/* Strengths / weaknesses */
.sw {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}
@media (max-width: 720px) {
  .sw {
    grid-template-columns: 1fr;
  }
}
.sw__col {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 8px;
  padding: 12px 14px;
}
.sw__col--strong {
  border-left: 2px solid #2ead6c;
}
.sw__col--weak {
  border-left: 2px solid #e84040;
}
.sw__title {
  font-family: var(--sans);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  margin-bottom: 8px;
}
.sw__col--strong .sw__title {
  color: #2ead6c;
}
.sw__col--weak .sw__title {
  color: #e84040;
}
.sw__col ul {
  list-style: none;
  padding: 0;
  margin: 0;
  display: flex;
  flex-direction: column;
  gap: 5px;
}
.sw__col li {
  font-size: 11.5px;
  color: var(--dm);
  padding-left: 14px;
  position: relative;
  line-height: 1.55;
}
.sw__col li::before {
  content: "›";
  position: absolute;
  left: 0;
}
.sw__col--strong li::before {
  color: #2ead6c;
}
.sw__col--weak li::before {
  color: #e84040;
}
</style>
