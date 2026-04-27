<script setup>
import data from "../../database/john.json";

const attackModes = data.attackModes;
const commonFlags = data.commonFlags;
const extractors = data.extractors;
const recipes = data.recipes;
const configFiles = data.configFiles;
const tips = data.tips;
const limitations = data.limitations;
</script>

<template>
  <div class="page-shell" style="--accent: #c0392b">
    <div class="page">
      <header class="hdr">
        <div class="icon">⚒</div>
        <div>
          <h1>John the Ripper</h1>
          <div class="sub">
            The classic offline password cracker — auto-detects dozens of
            hash formats, runs wordlist / rule / mask / incremental
            attacks, and resumes sessions across reboots.
          </div>
        </div>
        <a
          class="external"
          href="https://www.openwall.com/john/"
          target="_blank"
          rel="noopener noreferrer"
        >
          Open homepage ↗
        </a>
      </header>

      <div class="warning">
        <strong>Authorisation required</strong>
        Cracking hashes you do not own is illegal in most jurisdictions.
        Only run John against systems, dumps, or CTF material you have
        explicit written permission to test.
      </div>

      <div class="callout">
        <strong>What it is</strong>
        John the Ripper is a CPU-first offline password cracker. The
        community <em>jumbo</em> fork (<code>john-jumbo</code>) adds GPU
        support, hundreds of extra hash formats, and the rich
        <code>*2john</code> family of extractors. Feed it hashes, pick an
        attack mode, wait for plaintexts to drop into the pot file.
      </div>

      <!-- How it works -->
      <section class="section">
        <h2>How it works</h2>
        <p class="prose">
          John reads a file of hashes (one per line, in its own canonical
          format), <strong>auto-detects</strong> the algorithm, then loops
          a candidate generator into the appropriate hashing routine and
          compares. Recovered passwords are appended to the
          <em>pot file</em> at <code>~/.john/john.pot</code>, so every run
          is incremental — the same hash will not be re-cracked next time.
          Candidate generation is the entire game, and that's what the
          attack modes below configure.
        </p>
      </section>

      <!-- Anatomy of an invocation -->
      <section class="section">
        <h2>Anatomy of an invocation</h2>
        <div class="anatomy">
          <code class="anatomy__cmd">
            john <span class="hl">--format=NT</span>
            <span class="hl">--wordlist=rockyou.txt</span>
            <span class="hl">--rules</span>
            <span class="hl">hashes.txt</span>
          </code>
          <div class="anatomy__legend">
            <div><code class="hl">--format=NT</code> — force the hash type (skip auto-detect)</div>
            <div><code class="hl">--wordlist=…</code> — candidate source</div>
            <div><code class="hl">--rules</code> — apply mangling rules to each word</div>
            <div><code class="hl">hashes.txt</code> — one hash per line</div>
          </div>
        </div>
      </section>

      <!-- Attack modes -->
      <section class="section">
        <h2>Attack modes</h2>
        <p class="prose">
          John has five built-in modes. They are tried roughly in order of
          cost — single first, incremental last. You can chain them across
          runs against the same hash file; the pot ensures earlier wins
          aren't repeated.
        </p>
        <div class="modes">
          <article v-for="m in attackModes" :key="m.name" class="mode">
            <div class="mode__head">
              <h3>{{ m.name }}</h3>
              <code class="hl">{{ m.flag }}</code>
            </div>
            <p class="mode__sum">{{ m.summary }}</p>
            <p class="mode__when"><strong>Use when:</strong> {{ m.useWhen }}</p>
          </article>
        </div>
      </section>

      <!-- Common flags -->
      <section class="section">
        <h2>Common flags</h2>
        <p class="prose">
          The flags you'll reach for daily — format selection, attack
          configuration, and session control. <code>john --help</code>
          prints the full list for your build.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Flag</th>
                <th>What it does</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="f in commonFlags" :key="f.flag">
                <td><code class="hl">{{ f.flag }}</code></td>
                <td>{{ f.description }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Extractors -->
      <section class="section">
        <h2>The <code>*2john</code> family</h2>
        <p class="prose">
          John itself only cracks hashes — to get hashes <em>out of</em> a
          ZIP, PDF, KeePass database, or SSH key, the jumbo build ships a
          family of helpers that read the proprietary format and emit a
          single line in John's canonical hash format. Pipe that into
          <code>john</code> and proceed normally.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Helper</th>
                <th>Purpose</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="e in extractors" :key="e.tool">
                <td><code class="hl">{{ e.tool }}</code></td>
                <td>{{ e.use }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Recipes -->
      <section class="section">
        <h2>Recipes</h2>
        <p class="prose">
          Concrete invocations that solve concrete problems — the patterns
          you'll actually paste into a terminal.
        </p>
        <div class="recipes">
          <article v-for="r in recipes" :key="r.title" class="rcard">
            <h3>{{ r.title }}</h3>
            <pre class="rcard__cmd"><code>{{ r.command }}</code></pre>
            <p>{{ r.details }}</p>
          </article>
        </div>
      </section>

      <!-- Config files -->
      <section class="section">
        <h2>State on disk</h2>
        <p class="prose">
          John keeps a small set of files in <code>~/.john/</code>. Knowing
          where they live makes it easy to back up cracked credentials,
          resume jobs, or wipe state between engagements.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Path</th>
                <th>Purpose</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="c in configFiles" :key="c.path">
                <td><code class="hl">{{ c.path }}</code></td>
                <td>{{ c.purpose }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- John vs hashcat -->
      <section class="section">
        <h2>John vs hashcat</h2>
        <p class="prose">
          They cover the same job from different angles. <strong>John</strong>
          shines on CPU, on weird formats, and on small jobs where its
          auto-detection and rule engine save setup time.
          <strong>hashcat</strong> shines on GPU and at scale — orders of
          magnitude faster on common hashes when you have the silicon. The
          hash files and pot lines are largely interchangeable, so the
          standard workflow is to triage with John and hand the
          stragglers to hashcat.
        </p>
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
        Reference data sourced from <code>database/john.json</code>
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

.warning {
  background: color-mix(in srgb, #d14c4c 10%, transparent);
  border: 1px solid #d14c4c;
  border-radius: 8px;
  padding: 12px 14px;
  font-size: 11.5px;
  color: var(--tx);
  margin-bottom: 18px;
  line-height: 1.6;
}
.warning strong {
  color: #ff8a8a;
  font-family: var(--sans);
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  display: block;
  margin-bottom: 4px;
}

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
.callout code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: var(--accent);
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
.section h2 code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 6px;
  font-size: 13px;
  color: var(--accent);
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

/* Attack modes */
.modes {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 12px;
}
.mode {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 12px 14px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.mode__head {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
  flex-wrap: wrap;
}
.mode__head h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 12.5px;
  font-weight: 700;
  color: #fff;
}
.mode__head code {
  font-family: var(--mono);
  font-size: 11px;
  color: var(--accent);
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 6px;
}
.mode__sum {
  margin: 0;
  font-size: 11.5px;
  line-height: 1.6;
  color: var(--tx);
}
.mode__when {
  margin: 0;
  font-size: 11px;
  line-height: 1.5;
  color: var(--dm);
}
.mode__when strong {
  color: var(--accent);
  font-family: var(--sans);
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  margin-right: 4px;
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
