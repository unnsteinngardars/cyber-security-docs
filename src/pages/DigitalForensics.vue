<script setup>
import toolsData from "../../database/tools.json";

const forensicsToolNames = ["Autopsy", "DumpIt", "Volatility"];
const forensicsTools = forensicsToolNames
  .map((name) => toolsData.find((t) => t.name === name))
  .filter(Boolean);
</script>

<template>
  <div class="page-shell" style="--accent: #4f8cc9">
    <div class="page">
      <header class="hdr">
        <div class="icon">🔍</div>
        <div>
          <h1>Digital Forensics</h1>
          <div class="sub">
            The discipline of identifying, preserving, examining, and presenting
            digital evidence in a way that holds up in court — and answers
            <em>what happened</em> after an incident.
          </div>
        </div>
      </header>

      <div class="callout">
        <strong>Mental model</strong>
        Digital forensics is <em>science applied under legal constraints</em>.
        Every step has to be reproducible and defensible — if a peer can't
        repeat your work and reach the same conclusion, the evidence loses
        weight in court. <em>Preserve first, analyse second</em>.
      </div>

      <!-- Methodology -->
      <section class="section">
        <h2>Methodology — the four-phase model</h2>
        <p class="prose">
          The canonical reference is <strong>NIST SP 800-86</strong>, which
          breaks the forensic process into four sequential phases. Each phase
          feeds the next; cutting corners early (e.g. analysing on the live
          disk instead of an image) usually destroys evidence you can't get
          back.
        </p>

        <div class="prop-grid">
          <article class="prop">
            <h3>1. Collection</h3>
            <p>
              Identify, label, record, and <strong>acquire</strong> the data
              from possible sources while preserving integrity. Bit-for-bit
              images of disks, full memory captures, network packet captures,
              log exports. The output is an immutable working copy with hashes
              (MD5/SHA-256) computed and recorded.
            </p>
          </article>
          <article class="prop">
            <h3>2. Examination</h3>
            <p>
              Forensically process the collected data using automated and
              manual methods to <strong>extract data of interest</strong> while
              still preserving integrity. File-system parsing, carving, hash
              filtering against known-good/known-bad sets, keyword indexing,
              decoding of artefacts.
            </p>
          </article>
          <article class="prop">
            <h3>3. Analysis</h3>
            <p>
              Interpret the extracted data to <strong>answer the
              investigative question</strong> — who did what, when, from
              where, and how. Build a timeline, correlate across sources,
              attribute actions to user accounts and processes, identify
              indicators of compromise.
            </p>
          </article>
          <article class="prop">
            <h3>4. Reporting</h3>
            <p>
              Document findings, methods, tools, and conclusions for the
              audience — incident commanders, legal counsel, or a court. A
              good report lets a peer reproduce every step from acquisition
              to conclusion. Include limitations, assumptions, and gaps
              honestly.
            </p>
          </article>
        </div>
      </section>

      <!-- Categories -->
      <section class="section">
        <h2>Categories — what to collect</h2>
        <p class="prose">
          The collection phase isn't a single act. Different evidence lives in
          different places, decays at different rates, and needs different
          tools. The <em>order of volatility</em> matters — capture
          fastest-fading sources first (RAM, network state) before slower
          sources (disk, archived logs).
        </p>

        <div class="prop-grid">
          <article class="prop">
            <h3>Disk forensics</h3>
            <p>
              Bit-for-bit imaging of HDDs, SSDs, USB drives, and other
              persistent storage. Recover deleted files, parse file systems
              (NTFS, ext4, APFS), examine MFT/journal artefacts, registry
              hives, browser history, and shadow copies. The bedrock category.
            </p>
          </article>
          <article class="prop">
            <h3>Memory forensics</h3>
            <p>
              Capture and analyse <strong>RAM</strong> from a running system.
              Surfaces running processes, open network connections, loaded
              DLLs, command-line arguments, decrypted data, in-memory malware,
              and encryption keys that disappear at shutdown.
            </p>
          </article>
          <article class="prop">
            <h3>Network forensics</h3>
            <p>
              PCAPs, NetFlow, firewall and proxy logs, DNS records, IDS
              alerts. Reconstruct sessions, identify C2 beacons, lateral
              movement, exfiltration, and the external infrastructure an
              attacker touched.
            </p>
          </article>
          <article class="prop">
            <h3>Mobile forensics</h3>
            <p>
              Smartphones and tablets. Logical, file-system, and physical
              extractions via tools like Cellebrite UFED or Magnet AXIOM —
              SMS, call logs, location, app data, deleted records. Highly
              vendor-, OS-, and lock-state-dependent.
            </p>
          </article>
          <article class="prop">
            <h3>Database forensics</h3>
            <p>
              Examine RDBMS state and history — transaction logs, redo logs,
              audit tables, deleted rows recoverable from page slack. Common
              in fraud, insider, and breach investigations where the
              database itself was the target.
            </p>
          </article>
          <article class="prop">
            <h3>Email forensics</h3>
            <p>
              Headers (Received chain, Message-ID, Authentication-Results),
              body content, attachments, sender authentication
              (SPF / DKIM / DMARC), and mail-server logs. The starting point
              for phishing, BEC, and harassment cases.
            </p>
          </article>
          <article class="prop">
            <h3>Malware forensics</h3>
            <p>
              Static and dynamic analysis of suspicious binaries — strings,
              imports, signatures, sandbox detonation, debugger / disassembler
              work. Produces IOCs (hashes, C2 domains, mutexes) and TTPs that
              feed the rest of the investigation.
            </p>
          </article>
          <article class="prop">
            <h3>Cloud forensics</h3>
            <p>
              Evidence in IaaS / PaaS / SaaS — CloudTrail, Azure Activity Log,
              GCP Audit Logs, snapshot images, container artefacts. Complicated
              by multi-tenancy, ephemeral resources, jurisdictional limits,
              and reliance on the provider's APIs and retention.
            </p>
          </article>
        </div>
      </section>

      <!-- Evidence acquisition -->
      <section class="section">
        <h2>Evidence acquisition</h2>
        <p class="prose">
          Acquisition is the highest-stakes step in the entire process. A
          mistake here — writing to the source disk, breaking custody, acting
          without authority — can render every later finding inadmissible.
          Three controls do the heavy lifting: <strong>authorization</strong>,
          <strong>chain of custody</strong>, and <strong>write blockers</strong>.
        </p>

        <h3 class="subhead">Proper authorization</h3>
        <p class="prose">
          Acquiring or examining data without legal authority is at best
          inadmissible evidence and at worst a crime (unauthorised access,
          wiretap, privacy violations). Confirm — and document — the basis
          before you touch anything.
        </p>
        <ul class="bulletlist">
          <li>
            <strong>Search warrant</strong> — issued by a court, scoped to
            specific premises, devices, or data. Operate strictly within the
            scope; over-collection can taint the case.
          </li>
          <li>
            <strong>Court order / subpoena</strong> — for third-party data
            (cloud providers, ISPs, banks). Defines what the provider must
            produce and the legal mechanism for compelling it.
          </li>
          <li>
            <strong>Consent</strong> — voluntary, informed, and revocable.
            Get it in writing and record the scope: this device, these
            accounts, for this purpose.
          </li>
          <li>
            <strong>Employer right to monitor</strong> — usually grounded in
            an acceptable-use policy, employment contract, or banner notice.
            Confirm with HR and legal before imaging an employee's machine.
          </li>
          <li>
            <strong>Incident-response engagement</strong> — written letter of
            engagement / SOW that authorises the IR team to acquire and
            analyse organisational systems and data.
          </li>
          <li>
            <strong>Statutory authority</strong> — law-enforcement powers,
            regulator inspection rights, lawful intercept regimes. Strictly
            jurisdiction-specific.
          </li>
        </ul>

        <h3 class="subhead">Chain of custody</h3>
        <p class="prose">
          The <strong>chain of custody</strong> is the documented chronology
          of every person who handled an item of evidence — when, why, and
          what they did with it. A break in the chain (an unaccounted-for
          period, a missing signature, an unsealed bag) is a defence
          attorney's first lever and often enough to exclude the evidence.
        </p>
        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Field</th>
                <th>What to record</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><strong>Case / evidence ID</strong></td>
                <td>Unique identifier tying this item to the investigation.</td>
              </tr>
              <tr>
                <td><strong>Description</strong></td>
                <td>Make, model, serial number, capacity, condition, any
                  visible damage or markings.</td>
              </tr>
              <tr>
                <td><strong>Date &amp; time of seizure</strong></td>
                <td>Local time and timezone, plus where it was seized
                  from and from whom.</td>
              </tr>
              <tr>
                <td><strong>Hashes</strong></td>
                <td>MD5 / SHA-256 of the acquired image, computed at
                  acquisition and recorded immediately.</td>
              </tr>
              <tr>
                <td><strong>Custodian transfers</strong></td>
                <td>Every hand-off — who released, who received, when,
                  and why. Both parties sign.</td>
              </tr>
              <tr>
                <td><strong>Storage</strong></td>
                <td>Where the item is stored between transfers — evidence
                  locker, safe, sealed bag with tamper-evident tape.</td>
              </tr>
              <tr>
                <td><strong>Actions performed</strong></td>
                <td>What was done to the item, by whom, with which tool
                  and version, against which working copy.</td>
              </tr>
            </tbody>
          </table>
        </div>
        <p class="prose">
          The practical rules: <em>image the original once, then work only
          on copies</em>; <em>verify hashes before and after every action</em>;
          <em>store the original in a sealed, signed evidence bag</em>; and
          <em>log every access</em>, even your own.
        </p>

        <h3 class="subhead">Write blockers</h3>
        <p class="prose">
          A <strong>write blocker</strong> sits between the investigator's
          workstation and the source media and allows reads but blocks every
          write — so simply plugging in a suspect drive can't update access
          times, mount a journal, or trigger Windows to write a recycle bin
          entry. Without one, the source-disk hash will change the moment
          you connect it, and the integrity argument is gone.
        </p>
        <div class="prop-grid">
          <article class="prop">
            <h3>Hardware write blockers</h3>
            <p>
              Physical bridge devices (Tableau, WiebeTech, CRU) sitting on
              the SATA / SAS / USB / NVMe / PCIe bus. They intercept and
              drop write commands at the protocol level. Considered the
              gold standard — vendor-tested, court-accepted.
            </p>
          </article>
          <article class="prop">
            <h3>Software write blockers</h3>
            <p>
              OS-level controls — Linux's <code>blockdev --setro</code>,
              Windows registry settings for USB read-only, mounting an
              image read-only via a forensic distro (CAINE, SIFT, Tsurugi).
              Cheaper and more flexible, but configuration must be verified
              every time.
            </p>
          </article>
          <article class="prop">
            <h3>Verify the block</h3>
            <p>
              Always test before relying on it: hash the source, attempt a
              write, hash again. Record the device and firmware version of
              the blocker. A misconfigured or untested blocker is worse than
              none — it gives false confidence.
            </p>
          </article>
        </div>
      </section>

      <!-- Tools -->
      <section class="section">
        <h2>Tools</h2>
        <p class="prose">
          A small core stack covers most of the work — one per major data
          source. Each entry below is sourced from
          <code>database/tools.json</code> and appears in the full
          <router-link
            class="catlink"
            :to="{ name: 'page', params: { slug: 'tools' } }"
          >Tools catalogue</router-link>.
        </p>

        <div class="prop-grid">
          <article v-for="t in forensicsTools" :key="t.name" class="prop">
            <h3>
              <a
                class="prop-link"
                :href="t.homepage"
                target="_blank"
                rel="noopener noreferrer"
              >
                {{ t.name }} <span aria-hidden="true">↗</span>
              </a>
            </h3>
            <p>{{ t.shortDescription }}</p>
          </article>
        </div>
      </section>
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
.subhead {
  margin: 18px 0 8px;
  font-family: var(--sans);
  font-size: 12px;
  font-weight: 700;
  color: var(--accent);
  text-transform: uppercase;
  letter-spacing: 0.08em;
}

.prose {
  color: var(--tx);
  font-size: 12.5px;
  line-height: 1.7;
  margin: 0 0 12px;
}
.prose code,
.bulletlist code,
.prop code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11.5px;
  color: #9cdcfe;
}
.prose strong,
.bulletlist strong {
  color: #fff;
}
.prose em {
  color: var(--accent);
  font-style: normal;
}

.prop-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(240px, 1fr));
  gap: 10px;
  margin-bottom: 12px;
}
.prop {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 12px 14px;
}
.prop h3 {
  margin: 0 0 6px;
  font-family: var(--sans);
  font-size: 12px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
}
.prop p {
  margin: 0 0 6px;
  color: var(--tx);
  font-size: 11.5px;
  line-height: 1.6;
}
.prop p:last-child {
  margin-bottom: 0;
}
.prop strong {
  color: #fff;
}
.prop-link {
  color: #fff;
  text-decoration: none;
}
.prop-link:hover {
  color: var(--accent);
}
.catlink {
  color: var(--accent);
  text-decoration: none;
  border-bottom: 1px solid color-mix(in srgb, var(--accent) 40%, transparent);
}
.catlink:hover {
  border-bottom-color: var(--accent);
}

.table-wrap {
  overflow-x: auto;
  border: 1px solid var(--bd);
  border-radius: 8px;
  background: var(--sf);
  margin-bottom: 12px;
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
.tbl strong {
  color: #fff;
}

.bulletlist {
  margin: 0 0 12px;
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
</style>
