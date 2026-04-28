<script setup>
import data from "../../database/hydra.json";

const commonFlags = data.commonFlags;
const modules = data.modules;
const formAttackParts = data.formAttackParts;
const recipes = data.recipes;
const tips = data.tips;
const limitations = data.limitations;
</script>

<template>
  <div class="page-shell" style="--accent: #8e44ad">
    <div class="page">
      <header class="hdr">
        <div class="icon">⚡</div>
        <div>
          <h1>Hydra</h1>
          <div class="sub">
            The classic online (network) login brute-forcer — fifty-plus
            protocols, parallelised, stop-on-success, and as loud as you'd
            expect.
          </div>
        </div>
        <a
          class="external"
          href="https://github.com/vanhauser-thc/thc-hydra"
          target="_blank"
          rel="noopener noreferrer"
        >
          Open homepage ↗
        </a>
      </header>

      <div class="warning">
        <strong>Authorisation required</strong>
        Hydra hammers live services with credentials. Most production
        targets lock accounts, alert on failure spikes, or log the source
        IP forever. Use only against systems and CTF targets you have
        explicit written permission to test.
      </div>

      <div class="callout">
        <strong>What it is</strong>
        Hydra is an <em>online</em> credential brute-forcer — distinct
        from John / hashcat which work on captured hashes. It takes a
        username (or list), a password (or list / generator / combo), and
        a service URL, then opens many concurrent connections trying each
        combination until one logs in.
      </div>

      <!-- How it works -->
      <section class="section">
        <h2>How it works</h2>
        <p class="prose">
          Hydra opens <strong>-t</strong> concurrent network connections
          to the target service and feeds each one a candidate (login,
          password) pair drawn from your wordlists. The protocol module
          (<code>ssh</code>, <code>http-post-form</code>, <code>smb</code>,
          ...) knows how to send the right authentication frames and tell
          success from failure. A successful login is logged; with
          <code>-f</code> it stops the current host, with <code>-F</code>
          it stops the entire run. There is no offline component — every
          attempt is a real network round-trip.
        </p>
      </section>

      <!-- Anatomy -->
      <section class="section">
        <h2>Anatomy of an invocation</h2>
        <div class="anatomy">
          <code class="anatomy__cmd">
            hydra <span class="hl">-L users.txt</span>
            <span class="hl">-P pass.txt</span>
            <span class="hl">-t 16</span>
            <span class="hl">-f</span>
            <span class="hl">-o hits.txt</span>
            <span>ssh://10.0.0.5</span>
          </code>
          <div class="anatomy__legend">
            <div><code class="hl">-L users.txt</code> — usernames file</div>
            <div><code class="hl">-P pass.txt</code> — passwords file</div>
            <div><code class="hl">-t 16</code> — concurrent attempts</div>
            <div><code class="hl">-f</code> — stop on first success (per host)</div>
            <div><code class="hl">-o hits.txt</code> — write found credentials to disk</div>
            <div><code>ssh://10.0.0.5</code> — protocol://target[:port][/path]</div>
          </div>
        </div>
      </section>

      <!-- Common flags -->
      <section class="section">
        <h2>Common flags</h2>
        <p class="prose">
          The flag set you'll reach for daily. <code>hydra -h</code>
          prints the full list; <code>hydra -U module</code> shows
          per-module options (e.g. the form spec for
          <code>http-post-form</code>).
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

      <!-- Modules -->
      <section class="section">
        <h2>Service modules</h2>
        <p class="prose">
          Each service Hydra speaks is a <em>module</em>. The module name
          is the URL scheme, optionally followed by a path or per-module
          spec. <code>hydra -U &lt;module&gt;</code> documents the module's
          quirks.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Module</th>
                <th>What it targets</th>
                <th>URL syntax</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="m in modules" :key="m.name">
                <td><code class="hl">{{ m.name }}</code></td>
                <td>{{ m.summary }}</td>
                <td><code>{{ m.syntax }}</code></td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Form attack -->
      <section class="section">
        <h2>The <code>http-post-form</code> spec</h2>
        <p class="prose">
          The trickiest module — and the one that catches everyone the
          first time. The argument is a <strong>colon-delimited</strong>
          string with three or more parts: the path, the body template,
          and the failure / success marker. Capture the real form post in
          Burp first, then translate it.
        </p>
        <div class="modes">
          <article v-for="p in formAttackParts" :key="p.label" class="mode">
            <div class="mode__head">
              <h3>{{ p.label }}</h3>
              <code class="hl">{{ p.value }}</code>
            </div>
            <p class="mode__sum">{{ p.explain }}</p>
          </article>
        </div>
        <p class="prose" style="margin-top: 12px;">
          Joined: <code>"/login.php:user=^USER^&amp;pass=^PASS^:F=incorrect"</code>
        </p>
      </section>

      <!-- Recipes -->
      <section class="section">
        <h2>Recipes</h2>
        <p class="prose">
          Concrete invocations for the questions that actually come up —
          credential lists, combo files, password sprays, brute generators.
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
        <h2>Mock walkthrough — SSH credential discovery on a CTF box</h2>
        <p class="prose">
          A typical lab scenario: nmap shows OpenSSH on
          <code>10.10.11.42:22</code>, the box's web app leaks two
          usernames (<code>admin</code>, <code>jess</code>), and the room
          implies weak passwords. Goal: get an SSH login without spraying
          rockyou into the void.
        </p>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">1</span>
            <h3>Free guesses with <code>-e nsr</code></h3>
          </header>
          <p class="walk__desc">
            Before any wordlist runs, try the three "obvious" cases per
            user: empty password, password equal to the username, and the
            username reversed. Three free shots that catch lazy defaults.
          </p>
          <pre class="walk__cmd"><code>echo -e "admin\njess" > users.txt
hydra -L users.txt -e nsr -t 4 ssh://10.10.11.42</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>-e nsr</code>, conservative <code>-t</code>, login lists.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">2</span>
            <h3>Targeted wordlist with low concurrency</h3>
          </header>
          <p class="walk__desc">
            OpenSSH's <code>MaxStartups</code> kicks in fast. Drop
            <code>-t</code> to 4 so we don't get throttled or blacklisted
            by fail2ban. <code>-f</code> stops on the first success.
          </p>
          <pre class="walk__cmd"><code>hydra -L users.txt -P /usr/share/wordlists/rockyou.txt \
      -t 4 -f -o hits.txt ssh://10.10.11.42</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>-L</code>/<code>-P</code> lists, throttled <code>-t</code>, <code>-f</code>, <code>-o</code> output.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">3</span>
            <h3>Switch to a password spray</h3>
          </header>
          <p class="walk__desc">
            If the wordlist run started locking accounts, flip the matrix:
            one likely password against many users. Far less likely to
            trip lockouts because each user sees a single failed attempt.
          </p>
          <pre class="walk__cmd"><code>hydra -L users.txt -p Spring2026! \
      -t 1 -W 5 -o spray.txt ssh://10.10.11.42</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> single <code>-p</code> spray, <code>-W</code> delay between connections.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">4</span>
            <h3>Sweep the subnet</h3>
          </header>
          <p class="walk__desc">
            Once the credential is known, see if the same pair re-uses
            elsewhere. Build a target list with nmap, point Hydra at it
            with <code>-M</code>, and stop globally on the first hit.
          </p>
          <pre class="walk__cmd"><code>nmap -p 22 --open 10.10.11.0/24 -oG - \
  | awk '/Ports: 22\/open/{print $2}' > ssh-hosts.txt

hydra -l jess -p Summer2026! -M ssh-hosts.txt -F -o reuse.txt ssh</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>-M</code> target list, <code>-F</code> global stop, recon → brute pipeline.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">5</span>
            <h3>Verify and use the find</h3>
          </header>
          <p class="walk__desc">
            Hydra's hit isn't the end — confirm the credential by hand
            with the actual client. False positives happen when a service
            returns the same response on success and failure.
          </p>
          <pre class="walk__cmd"><code>cat hits.txt
# [22][ssh] host: 10.10.11.42 login: jess password: Summer2026!

ssh jess@10.10.11.42
# Welcome to Ubuntu 22.04.3 LTS</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> reading the output file, manual verification step.</p>
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
        Reference data sourced from <code>database/hydra.json</code>
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
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px; padding: 12px 14px;
  font-size: 11.5px; color: var(--dm); margin-bottom: 24px;
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
.section h2 code {
  background: var(--bg); border: 1px solid var(--bd);
  border-radius: 4px; padding: 1px 6px; font-size: 13px; color: var(--accent);
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
