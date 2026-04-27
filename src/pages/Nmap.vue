<script setup>
import data from "../../database/nmap.json";

const scanTypes = data.scanTypes;
const timing = data.timing;
const outputFormats = data.outputFormats;
const nseScripts = data.nseScripts;
const recipes = data.recipes;
const tips = data.tips;
const limitations = data.limitations;
</script>

<template>
  <div class="page-shell" style="--accent: #6f4ab8">
    <div class="page">
      <header class="hdr">
        <div class="icon">⊕</div>
        <div>
          <h1>Nmap</h1>
          <div class="sub">
            The Network Mapper — host discovery, port scanning, version
            detection, OS fingerprinting, and a 600+ script engine for
            anything else.
          </div>
        </div>
        <a
          class="external"
          href="https://nmap.org/"
          target="_blank"
          rel="noopener noreferrer"
        >
          Open homepage ↗
        </a>
      </header>

      <div class="callout">
        <strong>What it is</strong>
        Nmap crafts and sends packets at a target, watches what comes
        back, and infers state. Open vs closed vs filtered ports, running
        services and versions, OS guesses, even vulnerability checks via
        the <em>Nmap Scripting Engine</em> (NSE) — all from one binary.
      </div>

      <!-- Authorisation warning -->
      <div class="warn">
        <strong>Authorisation</strong>
        Nmap is an active probe. Scanning hosts you don't own or don't
        have written permission to test is illegal in most jurisdictions.
        Use <code>scanme.nmap.org</code> for syntax practice — it's the
        project's authorised target.
      </div>

      <!-- How it works -->
      <section class="section">
        <h2>How it works</h2>
        <p class="prose">
          For each target Nmap runs a pipeline:
          <strong>host discovery</strong> (is it up?) →
          <strong>port scan</strong> (which ports respond?) →
          <strong>service / version detection</strong> (what's listening?)
          → <strong>OS detection</strong> (what stack is it?) →
          <strong>NSE scripts</strong> (deeper checks against discovered
          services). Each stage can be skipped or substituted via flags.
          Results are interpreted as <em>open</em> (got a SYN/ACK or app
          response), <em>closed</em> (got a RST), or
          <em>filtered</em> (no response — usually a firewall).
        </p>
      </section>

      <!-- Anatomy -->
      <section class="section">
        <h2>Anatomy of a scan</h2>
        <div class="anatomy">
          <code class="anatomy__cmd">
            nmap <span class="hl">-sS</span> <span class="hl">-sV</span>
            <span class="hl">-sC</span> <span class="hl">-p 1-65535</span>
            <span class="hl">-T4</span> <span class="hl">-oA scan</span>
            <span>target.example.com</span>
          </code>
          <div class="anatomy__legend">
            <div><code class="hl">-sS</code> — TCP SYN (half-open) scan</div>
            <div><code class="hl">-sV</code> — service / version detection</div>
            <div><code class="hl">-sC</code> — default NSE script set</div>
            <div><code class="hl">-p 1-65535</code> — all TCP ports</div>
            <div><code class="hl">-T4</code> — aggressive timing</div>
            <div><code class="hl">-oA scan</code> — output all formats</div>
          </div>
        </div>
      </section>

      <!-- Scan types -->
      <section class="section">
        <h2>Scan types</h2>
        <p class="prose">
          The <code>-s*</code> family selects <em>how</em> Nmap probes
          ports. SYN scan is the default (and best) when you can run as
          root; everything else is a specialised case.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Flag</th>
                <th>Name</th>
                <th>Behaviour</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="s in scanTypes" :key="s.flag">
                <td><code class="hl">{{ s.flag }}</code></td>
                <td>{{ s.name }}</td>
                <td>{{ s.description }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Timing -->
      <section class="section">
        <h2>Timing templates</h2>
        <p class="prose">
          The <code>-T</code> templates set sensible defaults for how
          aggressively Nmap sends probes. Tune individually with
          <code>--min-rate</code>, <code>--max-retries</code>,
          <code>--host-timeout</code> for fine control.
        </p>
        <div class="timing-grid">
          <article v-for="t in timing" :key="t.flag" class="tcell">
            <div class="tcell__head">
              <code class="hl">{{ t.flag }}</code>
              <span class="tcell__name">{{ t.name }}</span>
            </div>
            <p>{{ t.description }}</p>
          </article>
        </div>
      </section>

      <!-- NSE -->
      <section class="section">
        <h2>NSE — the Nmap Scripting Engine</h2>
        <p class="prose">
          NSE turns Nmap from a port scanner into a network reconnaissance
          framework. Scripts are written in Lua, organised by
          <em>category</em>, and run against discovered services. Invoke
          a category with <code>--script &lt;name&gt;</code>, list with
          <code>nmap --script-help &lt;name&gt;</code>.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Script / category</th>
                <th>Use</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="s in nseScripts" :key="s.name">
                <td><code class="hl">{{ s.name }}</code></td>
                <td>{{ s.description }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Output formats -->
      <section class="section">
        <h2>Output formats</h2>
        <p class="prose">
          On real engagements, always use <code>-oA</code> — multiple
          formats, no recompute, every reporting tool downstream is
          covered.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Flag</th>
                <th>Format</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="o in outputFormats" :key="o.flag">
                <td><code class="hl">{{ o.flag }}</code></td>
                <td>{{ o.description }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Recipes -->
      <section class="section">
        <h2>Recipes</h2>
        <p class="prose">
          Concrete invocations for the questions you'll actually be
          asked.
        </p>
        <div class="recipes">
          <article v-for="r in recipes" :key="r.title" class="rcard">
            <h3>{{ r.title }}</h3>
            <pre class="rcard__cmd"><code>{{ r.command }}</code></pre>
            <p>{{ r.details }}</p>
          </article>
        </div>
      </section>

      <!-- Tips -->
      <section class="section">
        <h2>Tips &amp; gotchas</h2>
        <ul class="bulletlist">
          <li v-for="t in tips" :key="t">{{ t }}</li>
        </ul>
      </section>

      <!-- Limitations -->
      <section class="section">
        <h2>Limitations</h2>
        <ul class="bulletlist bulletlist--warn">
          <li v-for="l in limitations" :key="l">{{ l }}</li>
        </ul>
      </section>

      <footer class="ft">
        Reference data sourced from <code>database/nmap.json</code>
      </footer>
    </div>
  </div>
</template>

<style scoped>
.hdr {
  position: relative;
}
.external {
  position: absolute;
  top: 0;
  right: 0;
  font-family: var(--sans);
  font-size: 10.5px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  color: var(--accent);
  text-decoration: none;
  border: 1px solid var(--accent);
  border-radius: 999px;
  padding: 5px 10px;
  background: color-mix(in srgb, var(--accent) 14%, transparent);
}
.external:hover {
  background: color-mix(in srgb, var(--accent) 24%, transparent);
}
@media (max-width: 640px) {
  .external {
    position: static;
    align-self: flex-start;
    margin-top: 6px;
  }
}

.callout {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  padding: 12px 14px;
  font-size: 11.5px;
  color: var(--dm);
  margin-bottom: 14px;
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

.warn {
  background: color-mix(in srgb, #d14c4c 10%, transparent);
  border: 1px solid #d14c4c;
  border-radius: 8px;
  padding: 10px 14px;
  font-size: 11.5px;
  color: var(--tx);
  margin-bottom: 24px;
  line-height: 1.6;
}
.warn strong {
  color: #d14c4c;
  font-family: var(--sans);
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  display: block;
  margin-bottom: 2px;
}
.warn code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
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

/* Anatomy */
.anatomy {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-radius: 10px;
  padding: 14px 16px;
  display: flex;
  flex-direction: column;
  gap: 12px;
}
.anatomy__cmd {
  font-family: var(--mono);
  font-size: 13px;
  color: var(--tx);
  display: block;
  word-break: break-all;
}
.anatomy__cmd .hl {
  color: var(--accent);
  font-weight: 600;
}
.anatomy__legend {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 6px 16px;
  font-size: 11.5px;
  color: var(--dm);
}
.anatomy__legend code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 0 5px;
  font-size: 11px;
  color: var(--accent);
  margin-right: 6px;
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
}

/* Timing */
.timing-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 8px;
}
.tcell {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 8px;
  padding: 10px 12px;
}
.tcell__head {
  display: flex;
  align-items: baseline;
  gap: 8px;
  margin-bottom: 4px;
}
.tcell__head code {
  background: var(--bg);
  border: 1px solid var(--accent);
  border-radius: 4px;
  padding: 1px 6px;
  font-size: 11.5px;
  color: var(--accent);
}
.tcell__name {
  font-family: var(--sans);
  font-size: 11px;
  font-weight: 700;
  color: #fff;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}
.tcell p {
  margin: 0;
  font-size: 11.5px;
  color: var(--tx);
  line-height: 1.6;
}

/* Recipes */
.recipes {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(320px, 1fr));
  gap: 12px;
}
.rcard {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 12px 14px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.rcard h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 12.5px;
  font-weight: 700;
  color: #fff;
}
.rcard__cmd {
  margin: 0;
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 6px;
  padding: 8px 10px;
  font-family: var(--mono);
  font-size: 11.5px;
  color: var(--accent);
  overflow-x: auto;
  white-space: pre;
}
.rcard p {
  margin: 0;
  font-size: 11.5px;
  line-height: 1.6;
  color: var(--tx);
}

/* Bullet lists */
.bulletlist {
  margin: 0;
  padding: 12px 14px 12px 30px;
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.bulletlist li {
  font-size: 12px;
  color: var(--tx);
  line-height: 1.6;
}
.bulletlist--warn {
  border-left-color: #d14c4c;
}
</style>
