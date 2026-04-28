<script setup>
import data from "../../database/burp-suite.json";

const editions = data.editions;
const tools = data.tools;
const intruderAttacks = data.intruderAttacks;
const extensions = data.extensions;
const tips = data.tips;
const limitations = data.limitations;
</script>

<template>
  <div class="page-shell" style="--accent: #ff6633">
    <div class="page">
      <header class="hdr">
        <div class="icon">⌗</div>
        <div>
          <h1>Burp Suite</h1>
          <div class="sub">
            The intercepting HTTP/S proxy and web-app testing workbench
            from PortSwigger — every web pentest's default companion.
          </div>
        </div>
        <a
          class="external"
          href="https://portswigger.net/burp"
          target="_blank"
          rel="noopener noreferrer"
        >
          Open homepage ↗
        </a>
      </header>

      <div class="warning">
        <strong>Authorisation required</strong>
        Burp lets you intercept and modify any traffic the configured
        browser produces. Only point it at applications, APIs, and hosts
        you own or have explicit written permission to test.
      </div>

      <div class="callout">
        <strong>What it is</strong>
        Burp Suite is a man-in-the-middle proxy that sits between your
        browser and the target web app, plus a workbench of tools for
        manually inspecting, replaying, fuzzing, and analysing the
        traffic. Community is free for manual testing; <em>Professional</em>
        adds the active vulnerability scanner and the BApp Store.
      </div>

      <!-- How it works -->
      <section class="section">
        <h2>How it works</h2>
        <p class="prose">
          Burp runs an HTTP proxy on <code>127.0.0.1:8080</code>. You
          configure a browser (or the bundled Burp browser) to route
          through it, install Burp's CA certificate so HTTPS pages don't
          warn, and from that point on every request and response is
          visible in the <strong>Proxy → HTTP history</strong>. From there
          you push interesting requests into <strong>Repeater</strong>
          (manual replay), <strong>Intruder</strong> (parameterised
          fuzzing), <strong>Sequencer</strong> (token randomness),
          <strong>Decoder</strong>, and <strong>Comparer</strong>. The
          <strong>Target</strong> tab keeps a sitemap of what's been seen
          and a <em>scope</em> definition that filters which hosts the
          other tools act on.
        </p>
      </section>

      <!-- Setup flow -->
      <section class="section">
        <h2>Getting traffic into Burp</h2>
        <div class="anatomy">
          <code class="anatomy__cmd">
            <span class="hl">1.</span> Start Burp → Proxy listens on
            <span class="hl">127.0.0.1:8080</span><br />
            <span class="hl">2.</span> Browser proxy →
            <span class="hl">HTTP/HTTPS = 127.0.0.1:8080</span><br />
            <span class="hl">3.</span> Visit
            <span class="hl">http://burp</span> → download CA cert<br />
            <span class="hl">4.</span> Install cert as a trusted root in
            the browser<br />
            <span class="hl">5.</span> Set scope on Target → Scope before
            you click anything else
          </code>
          <div class="anatomy__legend">
            <div><code class="hl">Bundled Burp browser</code> — skips steps 2-4 entirely</div>
            <div><code class="hl">FoxyProxy</code> — convenient browser proxy switcher</div>
            <div><code class="hl">Scope</code> — filters what every other Burp tool acts on</div>
          </div>
        </div>
      </section>

      <!-- Editions -->
      <section class="section">
        <h2>Editions</h2>
        <p class="prose">
          PortSwigger ships three editions. Most learning, CTFs, and bug
          bounty work happens on Community; Professional becomes
          worthwhile once your time is more expensive than the licence.
        </p>
        <div class="modes">
          <article v-for="e in editions" :key="e.name" class="mode">
            <div class="mode__head">
              <h3>{{ e.name }}</h3>
            </div>
            <p class="mode__sum">{{ e.summary }}</p>
            <p class="mode__when"><strong>Use when:</strong> {{ e.useWhen }}</p>
          </article>
        </div>
      </section>

      <!-- Tool tour -->
      <section class="section">
        <h2>The toolset</h2>
        <p class="prose">
          Burp's UI is a tab strip across the top — each tab is a separate
          tool with its own purpose. The flow is roughly: <em>Proxy</em>
          to capture, <em>Target</em> to organise, <em>Repeater</em> to
          probe, <em>Intruder</em> to scale, <em>Scanner</em> to
          automate.
        </p>
        <div class="tool-stack">
          <article v-for="t in tools" :key="t.name" class="tool">
            <h3>{{ t.name }}</h3>
            <p class="tool__sum">{{ t.summary }}</p>
            <ul class="tool__feats">
              <li v-for="f in t.keyFeatures" :key="f">{{ f }}</li>
            </ul>
          </article>
        </div>
      </section>

      <!-- Intruder attack types -->
      <section class="section">
        <h2>Intruder attack types</h2>
        <p class="prose">
          The four attack types control how payloads are fed into the
          marked insertion points. Picking the right one is most of
          getting Intruder right.
        </p>
        <div class="modes">
          <article v-for="a in intruderAttacks" :key="a.name" class="mode">
            <div class="mode__head">
              <h3>{{ a.name }}</h3>
            </div>
            <p class="mode__sum">{{ a.useCase }}</p>
            <p class="mode__when"><strong>How:</strong> {{ a.details }}</p>
          </article>
        </div>
      </section>

      <!-- Recommended extensions -->
      <section class="section">
        <h2>Recommended extensions (BApp Store)</h2>
        <p class="prose">
          A small set of extensions covers the gaps in vanilla Burp. All
          are free and install in one click from
          <strong>Extensions → BApp Store</strong> (Pro only).
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Extension</th>
                <th>What it adds</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="x in extensions" :key="x.name">
                <td><code class="hl">{{ x.name }}</code></td>
                <td>{{ x.purpose }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Mock walkthrough -->
      <section class="section">
        <h2>Mock walkthrough — finding an IDOR</h2>
        <p class="prose">
          A short concrete loop using only Community-tier tools against a
          fictional <code>shop.example.com</code> that exposes
          <code>/api/orders/&lt;id&gt;</code> without proper authorisation
          checks.
        </p>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">1</span>
            <h3>Capture the baseline request</h3>
          </header>
          <p class="walk__desc">
            Log in as a normal user, click into one of your own orders.
            Find the request in <strong>Proxy → HTTP history</strong>:
          </p>
          <pre class="walk__cmd"><code>GET /api/orders/8421 HTTP/2
Host: shop.example.com
Cookie: session=eyJ1aWQiOjQyfQ...
Authorization: Bearer eyJhbGc...</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> Proxy, HTTP history, scope filtering.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">2</span>
            <h3>Probe in Repeater</h3>
          </header>
          <p class="walk__desc">
            Right-click → <em>Send to Repeater</em>. Change the order ID
            to a neighbouring value and resend. If you get a different
            user's data back with a 200, you've found IDOR.
          </p>
          <pre class="walk__cmd"><code>GET /api/orders/8420 HTTP/2   → 200 OK { "user": "alice@example.com", ... }
GET /api/orders/8419 HTTP/2   → 200 OK { "user": "bob@example.com",  ... }</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> Repeater, response diffing, manual verification.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">3</span>
            <h3>Scale with Intruder (Sniper)</h3>
          </header>
          <p class="walk__desc">
            Send the request to Intruder, mark the ID as the only payload
            position, attach a numeric range
            <code>1–10000</code>, and grep-extract the <code>user</code>
            field from the response so you can sort the table by who you
            leaked.
          </p>
          <pre class="walk__cmd"><code>Positions: GET /api/orders/§8421§
Payload set: Numbers, 1 to 10000, step 1
Attack type: Sniper
Grep - extract: "user":"([^"]+)"</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> Intruder, Sniper, payload sets, grep-extract.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">4</span>
            <h3>Confirm with a second account (Autorize)</h3>
          </header>
          <p class="walk__desc">
            Install <code>Autorize</code>, paste a second user's
            <code>Authorization</code> header into it, and let it replay
            every request you made. Anywhere the second user's response
            equals the first user's response, you've confirmed broken
            access control.
          </p>
          <pre class="walk__cmd"><code>Autorize → Configuration:
  Authorization header for low-priv user: Bearer eyJhbGc...lowpriv
  Filter: Scope items only
  Status: ENABLED
→ Browse normally; results table flags "Bypassed!" rows.</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> BApp Store extensions, automated authorisation testing.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">5</span>
            <h3>Document the finding</h3>
          </header>
          <p class="walk__desc">
            Save the relevant request/response from Repeater and the
            Intruder attack export. In Pro, the <em>Issues</em> panel
            tracks reproduction steps; in Community, screenshots of
            Repeater plus the exported Intruder CSV form the evidence
            bundle.
          </p>
          <pre class="walk__cmd"><code># Pro: File → Save project copy
# Community: Right-click request → Save item
#            Intruder → Save → Results table</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> Project save (Pro), evidence export, reporting.</p>
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
        Reference data sourced from <code>database/burp-suite.json</code>
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
  font-size: 12.5px;
  color: var(--tx);
  display: block;
  word-break: break-word;
  line-height: 1.7;
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

/* Card grids (editions, intruder attacks) */
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
.mode__head h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 12.5px;
  font-weight: 700;
  color: #fff;
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

/* Tool stack */
.tool-stack {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.tool {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 10px;
  padding: 12px 14px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.tool h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 13px;
  font-weight: 700;
  color: #fff;
}
.tool__sum {
  margin: 0;
  font-size: 12px;
  line-height: 1.65;
  color: var(--tx);
}
.tool__feats {
  margin: 0;
  padding-left: 18px;
  display: flex;
  flex-direction: column;
  gap: 3px;
  font-size: 11.5px;
  color: var(--dm);
  line-height: 1.55;
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
