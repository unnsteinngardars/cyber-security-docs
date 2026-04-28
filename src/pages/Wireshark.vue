<script setup>
import data from "../../database/wireshark.json";

const captureFilters = data.captureFilters;
const displayFilters = data.displayFilters;
const cliTools = data.cliTools;
const useCases = data.useCases;
const stats = data.stats;
const tips = data.tips;
const limitations = data.limitations;
</script>

<template>
  <div class="page-shell" style="--accent: #1f78d1">
    <div class="page">
      <header class="hdr">
        <div class="icon">▣</div>
        <div>
          <h1>Wireshark</h1>
          <div class="sub">
            Reference-grade packet capture and protocol dissector — the
            analyst's microscope for network traffic.
          </div>
        </div>
        <a
          class="external"
          href="https://www.wireshark.org/"
          target="_blank"
          rel="noopener noreferrer"
        >
          Open homepage ↗
        </a>
      </header>

      <div class="callout">
        <strong>What it is</strong>
        Wireshark is a free, open-source <em>protocol analyzer</em>. It
        captures raw frames from a network interface, dissects them
        layer-by-layer (Ethernet → IP → TCP → HTTP → …), and presents
        the result in a tree you can search, filter, and reassemble.
      </div>

      <!-- How it works -->
      <section class="section">
        <h2>How it works</h2>
        <p class="prose">
          Wireshark sits on top of <strong>libpcap</strong> (Linux/macOS)
          or <strong>Npcap</strong> (Windows) — the same capture libraries
          tcpdump uses. The interface is put into
          <em>promiscuous mode</em> so it accepts every frame on the wire,
          not just frames addressed to it. Each packet is run through the
          dissector chain — first Ethernet, then whichever EtherType
          handler matches, recursively — and the parsed tree is shown in
          the bottom pane. <em>Display filters</em> are then applied
          post-capture to narrow what you see; <em>capture filters</em>
          (BPF syntax) decide what's recorded in the first place.
        </p>
      </section>

      <!-- Capture vs Display filters -->
      <section class="section">
        <h2>Capture filters vs display filters</h2>
        <p class="prose">
          The most common confusion for new users. <strong>Capture
          filters</strong> are evaluated by libpcap at capture time using
          the <em>BPF</em> language — minimal syntax, fast, but you can't
          recover what you didn't capture. <strong>Display filters</strong>
          run inside Wireshark over already-captured data using a much
          richer expression language — they can reference any dissected
          field. Default rule of thumb: capture broadly, filter narrowly.
        </p>

        <div class="split">
          <div class="split__col">
            <h4>Capture filter (BPF)</h4>
            <p class="split__sub">Pre-capture · libpcap syntax</p>
            <div class="table-wrap">
              <table class="tbl">
                <tbody>
                  <tr v-for="f in captureFilters" :key="f.filter">
                    <td><code>{{ f.filter }}</code></td>
                    <td>{{ f.description }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
          <div class="split__col">
            <h4>Display filter (Wireshark)</h4>
            <p class="split__sub">Post-capture · field-aware syntax</p>
            <div class="table-wrap">
              <table class="tbl">
                <tbody>
                  <tr v-for="f in displayFilters" :key="f.filter">
                    <td><code class="hl">{{ f.filter }}</code></td>
                    <td>{{ f.description }}</td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </section>

      <!-- Statistics that matter -->
      <section class="section">
        <h2>Statistics that matter</h2>
        <p class="prose">
          Most analysts spend their time in one of these views — they
          compress thousands of packets into something a human can reason
          about.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Menu</th>
                <th>What it gives you</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="s in stats" :key="s.menu">
                <td><code class="hl">{{ s.menu }}</code></td>
                <td>{{ s.purpose }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Real-world use cases -->
      <section class="section">
        <h2>Real-world investigations</h2>
        <p class="prose">
          Where the time investment pays off. Each of these is a routine
          SOC / DFIR workflow that boils down to a few filters and a
          stream-follow.
        </p>
        <div class="card-grid">
          <article v-for="u in useCases" :key="u.title" class="ucard">
            <h3>{{ u.title }}</h3>
            <p>{{ u.details }}</p>
          </article>
        </div>
      </section>

      <!-- Companion CLI -->
      <section class="section">
        <h2>Companion CLI tools</h2>
        <p class="prose">
          Installed alongside Wireshark — these are how you take packet
          analysis from the GUI to a pipeline.
        </p>
        <div class="card-grid card-grid--tight">
          <article v-for="t in cliTools" :key="t.name" class="tcard">
            <h3><code>{{ t.name }}</code></h3>
            <p>{{ t.description }}</p>
          </article>
        </div>
      </section>

      <!-- Mock walkthrough -->
      <section class="section">
        <h2>Mock walkthrough — triaging a phishing pcap</h2>
        <p class="prose">
          A user clicked a link in a suspicious email and the SOC
          captured the resulting traffic into <code>phish.pcap</code>.
          Goal: figure out what happened, what was downloaded, and which
          IOCs to push to the blocklist. Wireshark is the analyst's
          microscope here.
        </p>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">1</span>
            <h3>Open the pcap &amp; orient with a broad display filter</h3>
          </header>
          <p class="walk__desc">
            <code>File → Open</code> the capture. Wireshark dissects every
            packet and indexes them into the packet list. Apply a broad
            display filter to strip the noise — DNS and HTTP/S only.
          </p>
          <pre class="walk__cmd"><code>Display filter:  dns or http or tls.handshake.type == 1
# tls.handshake.type == 1 catches Client Hello — gives the SNI hostname
# even on encrypted connections.</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> display filter language, packet dissection, SNI extraction.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">2</span>
            <h3>Find top talkers via Conversations &amp; Endpoints</h3>
          </header>
          <p class="walk__desc">
            <em>Statistics → Conversations</em> lists every IP-pair (or
            TCP/UDP stream) ranked by bytes / packets. The
            <em>Endpoints</em> view does the same per IP. Sorting by
            bytes pulls the suspicious download host straight to the top.
          </p>
          <pre class="walk__cmd"><code>Statistics → Conversations → IPv4 tab → sort by Bytes
# 192.168.1.42 ↔ 185.130.5.231 :  4.2 MB transferred in 8s
# Right-click → Apply as Filter → Selected → A↔B</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> Statistics → Conversations / Endpoints, "Apply as Filter" pivot.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">3</span>
            <h3>Protocol Hierarchy for a quick traffic mix</h3>
          </header>
          <p class="walk__desc">
            <em>Statistics → Protocol Hierarchy</em> shows what fraction
            of the capture is each protocol. A small pcap dominated by
            HTTP file transfer (vs the usual mix of TLS / DNS / mDNS) is
            already suspicious.
          </p>
          <pre class="walk__cmd"><code>Statistics → Protocol Hierarchy
# Eth → IPv4 → TCP → HTTP        92.4%   ← unusual, almost everything is plaintext HTTP
# Eth → IPv4 → UDP → DNS          5.1%
# Eth → IPv4 → TCP → TLS          2.5%</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> Protocol Hierarchy, traffic-mix sanity check.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">4</span>
            <h3>Follow the HTTP stream to read the conversation</h3>
          </header>
          <p class="walk__desc">
            Right-click any HTTP packet to <em>Follow → HTTP Stream</em>.
            Wireshark reassembles the full request and response in one
            window. Now you can see the User-Agent, the URI, the
            <code>Content-Type</code> served back, and any redirects.
          </p>
          <pre class="walk__cmd"><code>Right-click → Follow → HTTP Stream

GET /update/setup.exe HTTP/1.1
Host: 185.130.5.231
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64)

HTTP/1.1 200 OK
Content-Type: application/octet-stream
Content-Length: 4393984
Content-Disposition: attachment; filename="setup.exe"
[binary payload follows]</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> Follow Stream, TCP reassembly, header inspection.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">5</span>
            <h3>Carve the dropped binary out of the pcap</h3>
          </header>
          <p class="walk__desc">
            <em>File → Export Objects → HTTP</em> lists every file
            transferred in HTTP responses. Save <code>setup.exe</code> to
            disk, then hash it for VirusTotal / sandbox submission.
          </p>
          <pre class="walk__cmd"><code>File → Export Objects → HTTP → save setup.exe

# Outside Wireshark:
sha256sum setup.exe
# 7c6f3a…  setup.exe   ← pivot this hash on VirusTotal</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> Export Objects, IOC extraction, downstream pivot.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">6</span>
            <h3>Save a slice for the report; merge captures if needed</h3>
          </header>
          <p class="walk__desc">
            Use <em>File → Export Specified Packets</em> to save just the
            filtered conversation as <code>incident.pcap</code> — much
            smaller than the full capture, and exactly what the report
            appendix needs. <code>mergecap</code> stitches multiple pcaps
            together by timestamp if you have captures from several
            sensors.
          </p>
          <pre class="walk__cmd"><code>File → Export Specified Packets → "Displayed" → incident.pcap

# Combine sensor captures on the CLI:
mergecap -w merged.pcap edge.pcap dmz.pcap user-vlan.pcap</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> filtered export, <code>mergecap</code>, evidence packaging.</p>
        </article>
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
        Reference data sourced from <code>database/wireshark.json</code>
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

/* Capture vs Display filter split */
.split {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}
@media (max-width: 900px) {
  .split {
    grid-template-columns: 1fr;
  }
}
.split__col h4 {
  margin: 0 0 2px;
  font-family: var(--sans);
  font-size: 11px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
}
.split__sub {
  font-size: 10.5px;
  color: var(--dm);
  font-style: italic;
  margin: 0 0 8px;
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

/* Use-case + CLI cards */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 12px;
}
.card-grid--tight {
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
}
.ucard,
.tcard {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 12px 14px;
}
.ucard h3,
.tcard h3 {
  margin: 0 0 6px;
  font-family: var(--sans);
  font-size: 12.5px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
}
.tcard h3 code {
  background: transparent;
  padding: 0;
  border: 0;
  color: var(--accent);
  font-family: var(--mono);
}
.ucard p,
.tcard p {
  margin: 0;
  color: var(--tx);
  font-size: 11.5px;
  line-height: 1.6;
}

/* Walkthrough */
.walk {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 10px;
  padding: 14px 16px;
  margin-bottom: 12px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.walk__head {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 0;
}
.walk__num {
  flex-shrink: 0;
  width: 26px;
  height: 26px;
  border-radius: 999px;
  background: color-mix(in srgb, var(--accent) 18%, transparent);
  border: 1px solid var(--accent);
  color: var(--accent);
  font-family: var(--sans);
  font-size: 11px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
}
.walk__head h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 13px;
  font-weight: 700;
  color: #fff;
}
.walk__desc {
  margin: 0;
  font-size: 12px;
  line-height: 1.65;
  color: var(--tx);
}
.walk__desc code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
}
.walk__desc strong {
  color: #fff;
}
.walk__desc em {
  color: var(--accent);
  font-style: normal;
}
.walk__cmd {
  margin: 0;
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 6px;
  padding: 10px 12px;
  font-family: var(--mono);
  font-size: 11.5px;
  color: var(--accent);
  overflow-x: auto;
  white-space: pre;
  line-height: 1.55;
}
.walk__feat {
  margin: 0;
  font-size: 11px;
  color: var(--dm);
  line-height: 1.5;
}
.walk__feat strong {
  color: var(--accent);
  font-family: var(--sans);
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.08em;
  margin-right: 4px;
}
.walk__feat code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 10.5px;
  color: var(--accent);
  margin: 0 1px;
}

/* Bullet lists */
.bulletlist {
  margin: 0;
  padding-left: 18px;
  display: flex;
  flex-direction: column;
  gap: 6px;
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  padding: 12px 14px 12px 30px;
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
