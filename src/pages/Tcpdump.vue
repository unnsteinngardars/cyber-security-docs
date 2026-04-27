<script setup>
import data from "../../database/tcpdump.json";

const commonFlags = data.commonFlags;
const bpfFilters = data.bpfFilters;
const recipes = data.recipes;
const tips = data.tips;
const limitations = data.limitations;
</script>

<template>
  <div class="page-shell" style="--accent: #d97757">
    <div class="page">
      <header class="hdr">
        <div class="icon">⌬</div>
        <div>
          <h1>tcpdump</h1>
          <div class="sub">
            The Unix CLI packet sniffer — small, fast, ubiquitous, and the
            right tool when you're SSH'd into a server with no GUI.
          </div>
        </div>
        <a
          class="external"
          href="https://www.tcpdump.org/"
          target="_blank"
          rel="noopener noreferrer"
        >
          Open homepage ↗
        </a>
      </header>

      <div class="callout">
        <strong>What it is</strong>
        tcpdump is a command-line packet capture tool built on
        <em>libpcap</em>. It puts an interface into promiscuous mode, runs
        a kernel-side BPF filter, and either prints decoded packets to the
        terminal or writes them to a pcap file for later analysis.
      </div>

      <!-- How it works -->
      <section class="section">
        <h2>How it works</h2>
        <p class="prose">
          tcpdump opens a capture handle on a network interface via
          <strong>libpcap</strong>. A <em>BPF filter</em> compiled from
          your expression runs inside the kernel, so packets that don't
          match are dropped before ever crossing into userspace — that's
          why a tight filter on a busy host costs almost nothing. Matching
          packets are then either decoded to stdout (with steadily more
          detail as you stack <code>-v</code>) or written to a pcap file
          via <code>-w</code> for later analysis in Wireshark.
        </p>
      </section>

      <!-- Anatomy -->
      <section class="section">
        <h2>Anatomy of an invocation</h2>
        <div class="anatomy">
          <code class="anatomy__cmd">
            tcpdump <span class="hl">-nn</span> <span class="hl">-i eth0</span>
            <span class="hl">-w cap.pcap</span>
            <span class="hl">'host 10.0.0.5 and tcp port 443'</span>
          </code>
          <div class="anatomy__legend">
            <div><code class="hl">-nn</code> — no name resolution</div>
            <div><code class="hl">-i eth0</code> — interface to capture on</div>
            <div><code class="hl">-w cap.pcap</code> — write packets to file</div>
            <div><code class="hl">'…'</code> — BPF capture filter (always quote it)</div>
          </div>
        </div>
      </section>

      <!-- Common flags -->
      <section class="section">
        <h2>Common flags</h2>
        <p class="prose">
          The flags you'll reach for daily — output control, file
          rotation, and verbosity. <code>man tcpdump</code> has the full
          list.
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

      <!-- BPF filter syntax -->
      <section class="section">
        <h2>BPF capture filter cookbook</h2>
        <p class="prose">
          tcpdump's filter language is <strong>BPF</strong> (Berkeley
          Packet Filter) — small, kernel-evaluated, and the same syntax
          Wireshark uses for capture filters (different from Wireshark's
          richer <em>display</em> filter language). Combine clauses with
          <code>and</code>, <code>or</code>, <code>not</code>, and parens.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Filter</th>
                <th>Catches</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="f in bpfFilters" :key="f.filter">
                <td><code>{{ f.filter }}</code></td>
                <td>{{ f.description }}</td>
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

      <!-- Wireshark integration -->
      <section class="section">
        <h2>Pairing with Wireshark</h2>
        <p class="prose">
          The standard workflow: <strong>capture with tcpdump on the
          server</strong>, <strong>analyse the pcap in Wireshark on a
          workstation</strong>. tcpdump's pcap file format is
          interchangeable with Wireshark's — no conversion needed. For
          live remote analysis, pipe the capture over SSH:
        </p>
        <pre class="snippet"><code>ssh box "tcpdump -i eth0 -U -w - 'not port 22'" | wireshark -k -i -</code></pre>
        <p class="prose">
          <code>-U</code> flushes packet-by-packet (no buffering),
          <code>-w -</code> writes to stdout, and Wireshark reads stdin
          via <code>-i -</code>. <code>not port 22</code> excludes the SSH
          tunnel itself so the capture doesn't loop.
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
        Reference data sourced from <code>database/tcpdump.json</code>
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

/* Inline snippet */
.snippet {
  margin: 0 0 12px;
  background: var(--bg);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 6px;
  padding: 10px 12px;
  font-family: var(--mono);
  font-size: 11.5px;
  color: var(--accent);
  overflow-x: auto;
  white-space: pre;
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
