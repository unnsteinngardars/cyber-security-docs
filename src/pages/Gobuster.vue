<script setup>
import data from "../../database/gobuster.json";

const modes = data.modes;
const commonFlags = data.commonFlags;
const wordlists = data.wordlists;
const recipes = data.recipes;
const tips = data.tips;
const limitations = data.limitations;
</script>

<template>
  <div class="page-shell" style="--accent: #00b4d8">
    <div class="page">
      <header class="hdr">
        <div class="icon">⊟</div>
        <div>
          <h1>Gobuster</h1>
          <div class="sub">
            Fast Go-based content discovery — directories, subdomains,
            vhosts, and S3 buckets via wordlist brute force.
          </div>
        </div>
        <a
          class="external"
          href="https://github.com/OJ/gobuster"
          target="_blank"
          rel="noopener noreferrer"
        >
          Open homepage ↗
        </a>
      </header>

      <div class="warning">
        <strong>Authorisation required</strong>
        Gobuster is a high-volume probe — every wordlist entry is a real
        request. Run it only against systems and CTF targets you have
        explicit written permission to test.
      </div>

      <div class="callout">
        <strong>What it is</strong>
        Gobuster takes a wordlist and tries every entry against a target
        as a path, subdomain, vhost, or bucket name. Single static Go
        binary, multi-threaded, with the filtering knobs needed to cut
        through wildcard responses and noise.
      </div>

      <!-- How it works -->
      <section class="section">
        <h2>How it works</h2>
        <p class="prose">
          Gobuster's loop is small: open <code>-t</code> goroutines, pull
          the next word from the list, build a request (HTTP path, DNS
          query, Host header, S3 URL — depending on mode), send it, and
          decide whether the response counts as a <em>hit</em> via
          status-code / response-size rules. Hits print to stdout (and to
          <code>-o</code> if set). It does no recursive walking, no
          intelligent guessing — quality of results is entirely a
          function of wordlist quality and your filtering choices.
        </p>
      </section>

      <!-- Anatomy -->
      <section class="section">
        <h2>Anatomy of an invocation</h2>
        <div class="anatomy">
          <code class="anatomy__cmd">
            gobuster <span class="hl">dir</span>
            <span class="hl">-u https://target/</span>
            <span class="hl">-w wordlist.txt</span>
            <span class="hl">-x php,html</span>
            <span class="hl">-t 50</span>
            <span class="hl">-k</span>
          </code>
          <div class="anatomy__legend">
            <div><code class="hl">dir</code> — mode (dir / dns / vhost / fuzz / s3 / gcs)</div>
            <div><code class="hl">-u</code> — target URL</div>
            <div><code class="hl">-w</code> — wordlist</div>
            <div><code class="hl">-x</code> — append extensions to each word</div>
            <div><code class="hl">-t</code> — concurrent goroutines</div>
            <div><code class="hl">-k</code> — skip TLS cert verification</div>
          </div>
        </div>
      </section>

      <!-- Modes -->
      <section class="section">
        <h2>Modes</h2>
        <p class="prose">
          The first positional argument selects a mode. Each mode runs a
          different request pattern and accepts a slightly different flag
          set — <code>gobuster &lt;mode&gt; --help</code> documents the
          mode-specific options.
        </p>
        <div class="modes">
          <article v-for="m in modes" :key="m.name" class="mode">
            <div class="mode__head">
              <h3>{{ m.name }}</h3>
            </div>
            <p class="mode__sum">{{ m.summary }}</p>
            <pre class="mode__cmd"><code>{{ m.example }}</code></pre>
          </article>
        </div>
      </section>

      <!-- Common flags -->
      <section class="section">
        <h2>Common flags</h2>
        <p class="prose">
          The flags that travel across every mode — wordlist, threading,
          status filtering, header / cookie injection.
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

      <!-- Wordlists -->
      <section class="section">
        <h2>Wordlists worth keeping handy</h2>
        <p class="prose">
          Most of these ship with <code>seclists</code> on Kali / Parrot.
          Layer them: a small fast pass first, the larger lists as a
          follow-up.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Path</th>
                <th>Use</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="w in wordlists" :key="w.path">
                <td><code class="hl">{{ w.path }}</code></td>
                <td>{{ w.purpose }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Recipes -->
      <section class="section">
        <h2>Recipes</h2>
        <p class="prose">
          Concrete invocations for the questions you'll actually be asked
          — fast first pass, subdomain enum, vhost discovery, status-code
          filtering, parameter fuzzing.
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
        <h2>Mock walkthrough — finding admin + a hidden vhost</h2>
        <p class="prose">
          A CTF target at <code>ctf.example.com → 10.10.42.7</code>
          where the public site is small but rumour says there's an admin
          panel and a dev vhost on the same IP. Walk: enumerate paths →
          enumerate subdomains → confirm with vhost mode → authenticated
          re-scan.
        </p>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">1</span>
            <h3>Quick directory scan with extensions</h3>
          </header>
          <p class="walk__desc">
            First pass with the medium raft list and the obvious web
            extensions. <code>-k</code> for self-signed labs;
            <code>-o</code> always.
          </p>
          <pre class="walk__cmd"><code>gobuster dir -u https://ctf.example.com/ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -x php,html,bak,txt -t 50 -k -o dir.txt
# /admin       (Status: 401)  ← interesting
# /backup.bak  (Status: 200)  ← jackpot</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>dir</code> mode, <code>-x</code> extensions, status-code surfacing.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">2</span>
            <h3>Filter wildcard 200s by length</h3>
          </header>
          <p class="walk__desc">
            On a second target the app returns 200 for every path. Find
            the size of a known-bad path with curl, then re-run with
            <code>--exclude-length</code> to silence the noise.
          </p>
          <pre class="walk__cmd"><code>curl -sk https://ctf.example.com/definitely-not-real | wc -c
# 1421

gobuster dir -u https://ctf.example.com/ -w common.txt \
  --exclude-length 1421 -t 50 -k -o dir-clean.txt</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> wildcard handling, <code>--exclude-length</code>, response-size diffing.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">3</span>
            <h3>Subdomain enumeration</h3>
          </header>
          <p class="walk__desc">
            DNS mode resolves each wordlist entry. Fast — ~5 minutes for
            the top-5 k. Pipe results into the next step.
          </p>
          <pre class="walk__cmd"><code>gobuster dns -d example.com \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  -t 100 -o subs.txt
# Found: api.example.com
# Found: dev.example.com  ← interesting</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>dns</code> mode, high concurrency on DNS.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">4</span>
            <h3>Vhost discovery on the same IP</h3>
          </header>
          <p class="walk__desc">
            DNS only finds public records. Many internal sites are
            virtual-hosted on the same IP without DNS — vhost mode tries
            wordlist entries as Host headers and looks for distinct
            responses.
          </p>
          <pre class="walk__cmd"><code>gobuster vhost -u https://10.10.42.7/ \
  -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt \
  --append-domain --domain example.com -k -o vhosts.txt
# Found: staging.example.com (Status: 200) [Size: 5421]
# Found: admin.example.com   (Status: 200) [Size: 3210]  ← not in DNS</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>vhost</code> mode, <code>--append-domain</code>, IP-based discovery.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">5</span>
            <h3>Authenticated re-scan inside the admin vhost</h3>
          </header>
          <p class="walk__desc">
            Logged into <code>admin.example.com</code> in a browser,
            grabbed the session cookie from devtools / Burp. Reuse it
            with <code>-c</code> to discover content that's invisible to
            unauthenticated requests.
          </p>
          <pre class="walk__cmd"><code>gobuster dir -u https://admin.example.com/ \
  -w raft-medium-directories.txt \
  -c "session=eyJ1c2VyIjoiYWRtaW4ifQ; csrf=abc" \
  -H "X-CSRF-Token: abc" \
  -x php,json,bak -t 30 -k -o admin-dir.txt</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>-c</code> cookie reuse, <code>-H</code> custom header, authenticated discovery.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">6</span>
            <h3>Parameter mining with <code>fuzz</code></h3>
          </header>
          <p class="walk__desc">
            Found <code>/api/v1/user</code> in the admin vhost; want to
            know which parameters it accepts. <code>fuzz</code> mode
            swaps <code>FUZZ</code> with each wordlist entry and
            response-size filtering surfaces the ones that change
            behaviour.
          </p>
          <pre class="walk__cmd"><code>gobuster fuzz -u "https://admin.example.com/api/v1/user?FUZZ=1" \
  -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt \
  -c "session=..." \
  --exclude-length 67 -t 30 -k</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>fuzz</code> mode, <code>FUZZ</code> placeholder, parameter discovery.</p>
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
        Reference data sourced from <code>database/gobuster.json</code>
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
.mode__head h3 {
  margin: 0; font-family: var(--sans); font-size: 12.5px; font-weight: 700; color: #fff;
}
.mode__sum { margin: 0; font-size: 11.5px; line-height: 1.6; color: var(--tx); }
.mode__cmd {
  margin: 0; background: var(--bg); border: 1px solid var(--bd);
  border-radius: 6px; padding: 8px 10px; font-family: var(--mono);
  font-size: 11px; color: var(--accent); overflow-x: auto; white-space: pre;
}

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
