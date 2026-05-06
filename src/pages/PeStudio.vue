<script setup>
// Page is a single-topic explainer — content lives directly in the
// template rather than a separate JSON file.
</script>

<template>
  <div class="page-shell" style="--accent: #3aa39f">
    <div class="page">
      <header class="hdr">
        <div class="icon">▤</div>
        <div>
          <h1>PEStudio</h1>
          <div class="sub">
            Static-analysis triage tool for Windows binaries — opens a PE
            without executing it and surfaces colour-coded risk
            indicators across every interesting structure.
          </div>
        </div>
        <a
          class="external"
          href="https://www.winitor.com/"
          target="_blank"
          rel="noopener noreferrer"
        >
          Open homepage ↗
        </a>
      </header>

      <div class="callout">
        <strong>What it is</strong>
        PEStudio is a free Windows tool by <em>Marc Ochsenmeier</em>
        (Winitor) that loads an EXE/DLL/SYS and parses every part of
        the <strong>PE format</strong> — headers, sections, imports,
        exports, resources, TLS callbacks, strings, signatures — into a
        tabbed UI. Anything suspicious is highlighted; anything benign
        is hidden behind a click. The default first stop on a malware
        sample before opening Ghidra or x64dbg.
      </div>

      <!-- How it works -->
      <section class="section">
        <h2>How it works</h2>
        <p class="prose">
          A Windows executable is a <strong>PE (Portable Executable)</strong>
          file — a stack of headers and sections describing the code,
          data, imports, and resources the loader needs to run it.
          PEStudio walks that structure entirely <em>statically</em>:
          the file is opened read-only, never executed. Each tab on the
          left exposes one part of the format, and PEStudio cross-checks
          the contents against curated blocklists (suspicious APIs,
          known-bad strings, blacklisted libraries) and external feeds
          (VirusTotal hash lookup, MITRE ATT&CK mapping). Indicators
          worth attention turn <span class="hot">red</span>; benign
          values stay grey.
        </p>
        <p class="note">
          PEStudio does not unpack, decrypt, or emulate. If a sample is
          packed (UPX, Themida, custom), the headers and strings you see
          are the packer's, not the underlying malware's. Run Detect It
          Easy first; unpack; then re-open in PEStudio.
        </p>
      </section>

      <!-- Tabs -->
      <section class="section">
        <h2>The tabs that matter</h2>
        <p class="prose">
          PEStudio's left rail lists every parsed structure. The seven
          below are the ones you'll touch in nearly every triage —
          taken in order, they answer <em>"what code is in this file,
          where will execution start, what does it pull in, what does it
          carry, and who signed it?"</em>
        </p>

        <div class="tab-grid">
          <article class="tab-card">
            <div class="tab-card__head">
              <h3>Sections</h3>
              <span class="tab-card__alias">.text · .data · .rdata · .rsrc</span>
            </div>
            <p class="tab-card__summary">
              Lists every section in the PE — name, virtual address,
              raw size, virtual size, entropy, and characteristics
              (executable / writable / readable).
            </p>
            <p class="tab-card__use">
              <strong>What to look for:</strong>
              non-standard section names (<code>.vmp0</code>,
              <code>UPX0</code>, <code>aspack</code>) signal a packer;
              <strong>high entropy</strong> (≈ 7.0+) on a section means
              compressed/encrypted contents — almost always a packer or
              embedded payload; a section that's both
              <strong>writable AND executable</strong> is a classic
              shellcode-stage red flag; mismatch between raw size and
              virtual size suggests packing or stripped overlays.
            </p>
          </article>

          <article class="tab-card">
            <div class="tab-card__head">
              <h3>Imports</h3>
              <span class="tab-card__alias">DLLs · API calls</span>
            </div>
            <p class="tab-card__summary">
              The Import Address Table — every external function the
              binary will call, grouped by the DLL that exports it
              (<code>kernel32.dll!CreateRemoteThread</code>,
              <code>ws2_32.dll!connect</code>, …).
            </p>
            <p class="tab-card__use">
              <strong>What to look for:</strong> imports are the
              clearest fingerprint of <em>capability</em>. PEStudio
              flags suspicious APIs against a curated list —
              <code>VirtualAllocEx</code> + <code>WriteProcessMemory</code>
              + <code>CreateRemoteThread</code> screams process
              injection; <code>InternetOpenA</code> /
              <code>WinHttpSendRequest</code> means network beacon;
              <code>SetWindowsHookEx</code> + <code>GetAsyncKeyState</code>
              suggests a keylogger; <code>CryptEncrypt</code> +
              <code>FindFirstFileW</code> hints ransomware. A nearly
              empty import table usually means dynamic resolution via
              <code>LoadLibrary</code>+<code>GetProcAddress</code> —
              suspicious in itself.
            </p>
          </article>

          <article class="tab-card">
            <div class="tab-card__head">
              <h3>Exports</h3>
              <span class="tab-card__alias">Externally callable functions</span>
            </div>
            <p class="tab-card__summary">
              Functions this file <em>provides</em> to other modules —
              normal for DLLs, unusual for EXEs.
            </p>
            <p class="tab-card__use">
              <strong>What to look for:</strong> ordinal-only exports
              with no symbolic names (a known evasion against casual
              analysis), exports named after well-known DLLs (a sign of
              <em>DLL side-loading</em> bait — drop your malicious
              <code>version.dll</code> next to a signed exe and the
              loader picks yours), and the exact set of exports an
              installer / dropper will call into. Common malicious
              export names: <code>DllRegisterServer</code> (run via
              <code>regsvr32</code>), <code>ServiceMain</code> (auto-run
              under <code>svchost</code>), <code>Start</code>,
              <code>Run</code>, <code>Open</code>.
            </p>
          </article>

          <article class="tab-card">
            <div class="tab-card__head">
              <h3>Resources</h3>
              <span class="tab-card__alias">.rsrc · icons · RCDATA</span>
            </div>
            <p class="tab-card__summary">
              Embedded data the binary carries inside its own file —
              icons, version info, manifests, dialogs, string tables,
              and arbitrary binary blobs (RCDATA).
            </p>
            <p class="tab-card__use">
              <strong>What to look for:</strong> droppers love stashing
              their second-stage payload as an RCDATA resource —
              PEStudio shows the size and entropy so you can spot a
              packed PE inside the resource section (high entropy,
              several hundred KB, type RCDATA). Version info that
              doesn't match the file's actual purpose (a "Microsoft
              Windows" copyright on an unsigned binary) is a red flag.
              The icon resource often gives away a lure — a Word/PDF
              icon on an EXE means social-engineering disguise.
            </p>
          </article>

          <article class="tab-card">
            <div class="tab-card__head">
              <h3>TLS Callback</h3>
              <span class="tab-card__alias">Thread Local Storage</span>
            </div>
            <p class="tab-card__summary">
              Functions the Windows loader runs <em>before</em>
              <code>main()</code> / <code>DllMain</code> as part of
              setting up Thread Local Storage. PEStudio lists every TLS
              callback address registered by the binary.
            </p>
            <p class="tab-card__use">
              <strong>What to look for:</strong> any presence at all is
              worth a second look — most legitimate user-mode binaries
              don't register TLS callbacks. Malware uses them for two
              reasons: (1) <em>anti-debug</em> — TLS callbacks fire
              before the debugger's initial breakpoint at the entry
              point, so a debugger that breaks "on entry" already
              executed the callback; and (2) <em>early staging</em> —
              decryption stubs and persistence routines run before any
              tool watching the entry point can intercept them. PEStudio
              flags TLS callbacks prominently for exactly this reason.
            </p>
          </article>

          <article class="tab-card">
            <div class="tab-card__head">
              <h3>Strings</h3>
              <span class="tab-card__alias">ASCII + Unicode</span>
            </div>
            <p class="tab-card__summary">
              Every printable ASCII and UTF-16 string PEStudio extracts
              from the file, with built-in filters for blacklisted
              keywords, URLs, file paths, registry keys, and regular
              expressions.
            </p>
            <p class="tab-card__use">
              <strong>What to look for:</strong> hard-coded URLs and IPs
              (C2 candidates), <code>HKLM\\SOFTWARE\\Microsoft\\Windows\\
              CurrentVersion\\Run</code>-style registry paths
              (persistence), mutex names (sandbox-evasion / single-instance
              guards), error strings hinting at the developer's language
              or framework, and PDB paths leaking project directory
              names. PEStudio's <em>blacklisted strings</em> list
              auto-flags terms like <code>cmd.exe /c</code>,
              <code>powershell -enc</code>, <code>VirtualAlloc</code>,
              and known C2 markers.
            </p>
            <p class="note">
              Strings live in the binary's data sections — if the
              sample is packed, you'll see the packer's strings, not
              the payload's. Run FLOSS for stack/obfuscated strings.
            </p>
          </article>

          <article class="tab-card">
            <div class="tab-card__head">
              <h3>Certificate</h3>
              <span class="tab-card__alias">Authenticode signature</span>
            </div>
            <p class="tab-card__summary">
              The Authenticode signature block — issuer, subject,
              serial, validity dates, certificate chain, and the
              signature's verification status against the embedded
              hash.
            </p>
            <p class="tab-card__use">
              <strong>What to look for:</strong> unsigned binaries are
              not automatically suspicious, but they shouldn't claim to
              be signed in their version-info; a present signature that
              <strong>fails to verify</strong> means the file was
              modified after signing — a textbook tampering / trojanised
              installer indicator. Watch for stolen or revoked
              certificates (NVIDIA, Realtek, and FoxConn certs have all
              been abused historically), recently-issued certificates
              from cheap CAs, and signers whose subject CN doesn't match
              the publisher named in the version block. Cross-check
              thumbprints against known-bad lists.
            </p>
          </article>
        </div>
      </section>

      <!-- Risk indicators -->
      <section class="section">
        <h2>Risk indicators &amp; the dashboard</h2>
        <p class="prose">
          Across every tab, PEStudio colour-codes findings against its
          own curated blocklists. The <strong>Indicators</strong> tab
          rolls all of them up into a single ranked summary — a
          one-screen verdict that's the first thing experienced analysts
          look at. Each finding links back to the tab/row it came from.
          PEStudio also computes the file's MD5/SHA-1/SHA-256 and
          (optionally) submits the hash — never the file — to
          VirusTotal so you can pivot to existing detections without
          uploading the sample.
        </p>
      </section>

      <!-- Workflow -->
      <section class="section">
        <h2>A typical triage walk</h2>
        <ol class="walk">
          <li>
            <strong>Indicators</strong> — read the rolled-up summary,
            look at hash &amp; VT count.
          </li>
          <li>
            <strong>Sections</strong> — check entropy and characteristics
            for packing.
          </li>
          <li>
            <strong>Imports</strong> — scan flagged APIs to guess
            capability.
          </li>
          <li>
            <strong>TLS callbacks &amp; entry point</strong> — note any
            pre-main code paths.
          </li>
          <li>
            <strong>Strings</strong> — pull URLs, registry keys, mutex
            names.
          </li>
          <li>
            <strong>Resources</strong> — look for embedded payloads
            (high-entropy RCDATA).
          </li>
          <li>
            <strong>Certificate</strong> — verify signature, check the
            signer.
          </li>
          <li>
            Decide: drop into Ghidra / x64dbg, or detonate in a sandbox.
          </li>
        </ol>
      </section>

      <!-- Limits -->
      <section class="section">
        <h2>What PEStudio doesn't do</h2>
        <p class="prose">
          PEStudio is purely static. It doesn't unpack, doesn't
          emulate, doesn't decompile. Packed samples will look mostly
          empty until unpacked. Encrypted strings stay encrypted —
          reach for <strong>FLOSS</strong>. Behaviour comes from
          dynamic analysis — <strong>Procmon</strong>,
          <strong>Process Hacker</strong>, sandboxes (Cuckoo,
          ANY.RUN, Joe Sandbox). Code semantics come from a
          disassembler — <strong>Ghidra</strong>, <strong>IDA</strong>,
          or <strong>Binary Ninja</strong>. PEStudio's job is to make
          the next ten minutes of static triage cheap, not to replace
          the rest of the toolchain.
        </p>
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
.callout strong em {
  color: var(--tx);
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
.prose .hot {
  color: #d14c4c;
  font-weight: 600;
}
.note {
  font-size: 11px;
  color: var(--dm);
  font-style: italic;
  margin: 8px 0 0;
}

/* Tab grid */
.tab-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 12px;
}
.tab-card {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 14px 14px 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.tab-card__head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 10px;
  flex-wrap: wrap;
}
.tab-card__head h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 13px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
}
.tab-card__alias {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--accent);
}
.tab-card__summary {
  margin: 0;
  font-size: 12px;
  color: var(--tx);
  line-height: 1.6;
}
.tab-card__use {
  margin: 0;
  font-size: 11.5px;
  color: var(--dm);
  line-height: 1.65;
}
.tab-card__use strong {
  color: var(--tx);
  font-weight: 600;
}
.tab-card__use em {
  color: var(--accent);
  font-style: normal;
}
.tab-card__summary code,
.tab-card__use code,
.tab-card__use + .note code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
}
.tab-card .note {
  margin-top: 6px;
}
.tab-card .note code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 10.5px;
  color: #9cdcfe;
}

/* Walk-through list */
.walk {
  margin: 0;
  padding-left: 22px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.walk li {
  font-size: 12px;
  color: var(--tx);
  line-height: 1.6;
}
.walk li strong {
  color: #fff;
  font-weight: 600;
}
.walk li code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
}
</style>
