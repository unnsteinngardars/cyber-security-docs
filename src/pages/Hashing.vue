<script setup>
// Single-topic explainer — content inline, no JSON file.
</script>

<template>
  <div class="page-shell" style="--accent: #3ab9b0">
    <div class="page">
      <header class="hdr">
        <div class="icon">⌗</div>
        <div>
          <h1>Hashing</h1>
          <div class="sub">
            One-way fingerprints for data — what hash functions are, how
            they're used for integrity, and how to read a password hash on
            sight.
          </div>
        </div>
      </header>

      <div class="callout">
        <strong>Mental model</strong>
        A cryptographic hash is a <em>fingerprint</em>. Any input — a
        sentence, a 4 GB ISO, a password — produces a fixed-size output
        that's unique in practice, completely changes if you flip a
        single bit, and can't be reversed back to the input.
      </div>

      <!-- What is a hash -->
      <section class="section">
        <h2>What is a cryptographic hash?</h2>
        <p class="prose">
          A function <code>H(input) → output</code> that takes data of any
          size and returns a fixed-size value (a <em>digest</em>). Good
          ones satisfy three properties:
        </p>
        <div class="prop-grid">
          <article class="prop">
            <h3>Pre-image resistance</h3>
            <p>
              Given a digest <code>h</code>, it should be infeasible to
              find any input <code>m</code> such that <code>H(m) = h</code>.
              This is what makes a hash <em>one-way</em>.
            </p>
          </article>
          <article class="prop">
            <h3>Second pre-image resistance</h3>
            <p>
              Given an input <code>m₁</code>, it should be infeasible to
              find a different <code>m₂</code> with the same digest. Stops
              an attacker swapping a benign file for a malicious one with
              the same hash.
            </p>
          </article>
          <article class="prop">
            <h3>Collision resistance</h3>
            <p>
              It should be infeasible to find <em>any</em> two inputs with
              the same digest. Weaker hashes (MD5, SHA-1) have lost this
              property — you can craft two PDFs, two certificates, even
              two JPEGs that hash to the same value.
            </p>
          </article>
        </div>

        <p class="prose">
          Plus two practical traits: <strong>determinism</strong> (same
          input always produces the same output) and the
          <strong>avalanche effect</strong> (flipping a single input bit
          changes roughly half the output bits).
        </p>
      </section>

      <!-- General-purpose hash families -->
      <section class="section">
        <h2>General-purpose hash families</h2>
        <p class="prose">
          These are <em>fast</em> hashes designed for integrity and
          general-purpose use. Speed is exactly what makes them
          <strong>wrong</strong> for password storage — see the section
          below.
        </p>

        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Family</th>
                <th>Output (bits / hex chars)</th>
                <th>Status</th>
                <th>Use today</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><code class="hl">MD5</code></td>
                <td>128 / 32</td>
                <td><span class="bad">Broken</span></td>
                <td>Non-security checksums only (e.g. quick file dedup). Collisions are trivial.</td>
              </tr>
              <tr>
                <td><code class="hl">SHA-1</code></td>
                <td>160 / 40</td>
                <td><span class="bad">Broken</span></td>
                <td>Legacy git object IDs only. Real collisions exist (SHAttered, 2017).</td>
              </tr>
              <tr>
                <td><code class="hl">SHA-2</code> (SHA-256, SHA-512, …)</td>
                <td>256 / 64 · 512 / 128</td>
                <td><span class="ok">Strong</span></td>
                <td>The default workhorse — TLS certs, signatures, file integrity, JWTs.</td>
              </tr>
              <tr>
                <td><code class="hl">SHA-3</code> (Keccak)</td>
                <td>variable</td>
                <td><span class="ok">Strong</span></td>
                <td>Sponge construction — drop-in alternative to SHA-2 with different math.</td>
              </tr>
              <tr>
                <td><code class="hl">BLAKE2 / BLAKE3</code></td>
                <td>variable</td>
                <td><span class="ok">Strong</span></td>
                <td>Modern, faster than SHA-2, used in WireGuard, IPFS, Cargo, restic.</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Integrity -->
      <section class="section">
        <h2>Hashing for integrity</h2>
        <p class="prose">
          The original use case. Anywhere you need to prove data hasn't
          changed, you compare hashes. Cheap to compute, cheap to compare,
          and a single bit-flip in the input is detected.
        </p>

        <h3 class="subhead">Verifying a download</h3>
        <p class="prose">
          The standard pattern when distributing software: publish the
          file, publish its SHA-256 alongside, ideally sign the hash with
          a known key.
        </p>
        <pre class="snippet"><code>$ sha256sum ubuntu-24.04.iso
e240e4b8f2... ubuntu-24.04.iso

$ cat SHA256SUMS
e240e4b8f2... ubuntu-24.04.iso

$ gpg --verify SHA256SUMS.gpg SHA256SUMS</code></pre>
        <p class="prose">
          The hash by itself only proves the file matches what the
          publisher posted — it doesn't prove the publisher is real. The
          GPG signature on the SHA256SUMS file is what closes the loop.
        </p>

        <h3 class="subhead">HMAC — keyed integrity</h3>
        <p class="prose">
          A plain hash detects accidental changes; <strong>HMAC</strong>
          (Hash-based Message Authentication Code) detects intentional
          ones. <code>HMAC(key, message) → tag</code>. Anyone can verify
          the tag, but only someone with the key can produce a valid one.
          Used in TLS record protection, JWT signatures (HS256), API
          request signing (AWS SigV4), session cookies.
        </p>

        <h3 class="subhead">Other integrity uses</h3>
        <ul class="bulletlist">
          <li>
            <strong>Code signing</strong> — signed software is a hash of
            the binary encrypted with the publisher's private key.
          </li>
          <li>
            <strong>Git</strong> — every commit, tree, and blob is named
            by the SHA-1 (now SHA-256) of its content. The whole DAG is a
            Merkle tree, so tampering with history is detectable.
          </li>
          <li>
            <strong>Merkle trees</strong> — the structure that powers Git,
            BitTorrent, ZFS, blockchains, and certificate transparency
            logs. A hash of hashes lets you prove a single leaf is in a
            huge dataset with a tiny proof.
          </li>
          <li>
            <strong>Content addressing</strong> — IPFS, Docker layers,
            Nix store paths. The hash <em>is</em> the identifier, so
            duplicate content is naturally deduplicated.
          </li>
        </ul>
      </section>

      <!-- Password hashing -->
      <section class="section">
        <h2>Password hashing — a different beast</h2>
        <p class="prose">
          A general-purpose hash is the
          <strong>wrong</strong> tool for storing passwords. SHA-256 on a
          GPU runs at ~10 billion hashes per second — an attacker who
          gets your database can crunch through billions of guesses
          before lunch. A <em>password hashing function</em> (a kind of
          KDF — key derivation function) is engineered to be
          <strong>deliberately slow</strong> and
          <strong>memory-hard</strong>, so each guess costs the attacker
          real time and silicon.
        </p>

        <p class="prose">
          Three things every password hash needs:
        </p>
        <ul class="bulletlist">
          <li>
            <strong>Salt</strong> — random per-password value mixed into
            the hash so identical passwords don't produce identical
            digests, and so attackers can't precompute rainbow tables.
          </li>
          <li>
            <strong>Work factor</strong> — a tunable cost (rounds, memory,
            parallelism) that's increased over time as hardware speeds
            up.
          </li>
          <li>
            <strong>Modern algorithm</strong> — Argon2id or yescrypt for
            new systems; bcrypt is fine but ageing; PBKDF2 is the
            FIPS-friendly fallback. Avoid plain SHA-256, MD5-crypt, LM,
            and NTLM.
          </li>
        </ul>
      </section>

      <!-- Identifying password hashes -->
      <section class="section">
        <h2>Reading a password hash on sight</h2>
        <p class="prose">
          Most modern password hashes use the <strong>Modular Crypt
          Format</strong> (MCF): a single string starting with a
          <code>$</code>-delimited prefix that names the algorithm, then
          parameters, then the salt and digest. A glance at the prefix
          tells you exactly what you're looking at.
        </p>

        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Prefix</th>
                <th>Algorithm</th>
                <th>Status</th>
                <th>Where you'll see it</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><code class="hl">$1$</code></td>
                <td>MD5-crypt</td>
                <td><span class="bad">Weak</span></td>
                <td>Old Linux <code>/etc/shadow</code>, legacy NAS firmware.</td>
              </tr>
              <tr>
                <td><code class="hl">$2$ · $2a$ · $2b$ · $2y$</code></td>
                <td>bcrypt</td>
                <td><span class="ok">Acceptable</span></td>
                <td>Web apps, Ruby/Node libraries, OpenBSD. <code>$2b$</code> is the modern variant.</td>
              </tr>
              <tr>
                <td><code class="hl">$5$</code></td>
                <td>SHA-256-crypt</td>
                <td><span class="warn">Adequate</span></td>
                <td>Linux <code>/etc/shadow</code> on older distros.</td>
              </tr>
              <tr>
                <td><code class="hl">$6$</code></td>
                <td>SHA-512-crypt</td>
                <td><span class="warn">Adequate</span></td>
                <td>Linux <code>/etc/shadow</code> default for many years (RHEL/CentOS, older Ubuntu).</td>
              </tr>
              <tr>
                <td><code class="hl">$7$</code></td>
                <td>scrypt</td>
                <td><span class="ok">Strong</span></td>
                <td>Less common in shadow files, more in app frameworks.</td>
              </tr>
              <tr>
                <td><code class="hl">$y$</code></td>
                <td>yescrypt</td>
                <td><span class="ok">Strong</span></td>
                <td><strong>Default on modern Linux</strong> — Debian 11+, Fedora 35+, Ubuntu 22.04+.</td>
              </tr>
              <tr>
                <td><code class="hl">$argon2i$ · $argon2d$ · $argon2id$</code></td>
                <td>Argon2 (PHC winner, 2015)</td>
                <td><span class="ok">Strong</span></td>
                <td>New web app frameworks. <strong>Argon2id is the recommended default for new systems.</strong></td>
              </tr>
              <tr>
                <td><code class="hl">$pbkdf2-sha256$</code></td>
                <td>PBKDF2-HMAC-SHA256</td>
                <td><span class="warn">FIPS-only</span></td>
                <td>Compliance-driven environments; not memory-hard.</td>
              </tr>
              <tr>
                <td><code class="hl">{SSHA}</code></td>
                <td>Salted SHA-1 (LDAP)</td>
                <td><span class="bad">Weak</span></td>
                <td>OpenLDAP, older directory servers.</td>
              </tr>
            </tbody>
          </table>
        </div>

        <h3 class="subhead">Worked example — a yescrypt entry</h3>
        <pre class="snippet"><code>$y$j9T$F5Jx5fExrKuPp53xLKQ./0$pXm/Y3HnQc3HJEfn9Hk5RJ7G8fRmcD6cTPo9vk7cN29</code></pre>
        <div class="parts">
          <div class="part"><span>$y$</span><em>algorithm — yescrypt</em></div>
          <div class="part"><span>j9T</span><em>parameter block (cost, memory, parallelism)</em></div>
          <div class="part"><span>F5Jx5fExrKuPp53xLKQ./0</span><em>salt</em></div>
          <div class="part"><span>pXm/Y3HnQc3HJEfn9Hk5RJ7G8fRmcD6cTPo9vk7cN29</span><em>digest</em></div>
        </div>

        <h3 class="subhead">Raw hex hashes</h3>
        <p class="prose">
          When there's no <code>$</code> prefix, you're looking at a raw
          hex digest. Length tells you the algorithm:
        </p>

        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Hex chars</th>
                <th>Likely hash</th>
                <th>Notes</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><code class="hl">32</code></td>
                <td>MD5 · NTLM · LM</td>
                <td>NTLM/LM live in Windows SAM and AD; LM is upper-cased and split into 7-byte halves — laughably weak.</td>
              </tr>
              <tr>
                <td><code class="hl">40</code></td>
                <td>SHA-1</td>
                <td>Old git object IDs, legacy LDAP <code>{SHA}</code>.</td>
              </tr>
              <tr>
                <td><code class="hl">56</code></td>
                <td>SHA-224</td>
                <td>Rare in the wild.</td>
              </tr>
              <tr>
                <td><code class="hl">64</code></td>
                <td>SHA-256</td>
                <td>Common — file checksums, sha256sum, modern git.</td>
              </tr>
              <tr>
                <td><code class="hl">96</code></td>
                <td>SHA-384</td>
                <td>TLS certificate signatures.</td>
              </tr>
              <tr>
                <td><code class="hl">128</code></td>
                <td>SHA-512</td>
                <td>Strongest of the SHA-2 family.</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Tooling -->
      <section class="section">
        <h2>Identification tooling</h2>
        <ul class="bulletlist">
          <li>
            <strong><code>hashid</code></strong> / <strong><code>hash-identifier</code></strong> —
            CLI tools that take a hash and guess the algorithm based on
            length, prefix, and character set.
          </li>
          <li>
            <strong><code>hashcat --identify</code></strong> — Hashcat
            itself can detect modes from a sample. Each algorithm has a
            <code>-m</code> mode number (e.g. bcrypt is <code>3200</code>,
            NTLM is <code>1000</code>, sha256crypt is <code>7400</code>,
            yescrypt is <code>30000</code>).
          </li>
          <li>
            <strong>Hashcat example hashes</strong> at
            <code>hashcat.net/wiki/doku.php?id=example_hashes</code> — the
            canonical reference when you're trying to match a string to a
            mode number.
          </li>
        </ul>
      </section>

      <!-- Pitfalls -->
      <section class="section">
        <h2>Pitfalls</h2>
        <ul class="bulletlist bulletlist--warn">
          <li>
            Never hash passwords with a fast general-purpose hash (MD5,
            SHA-1, SHA-256). Always use a password-hashing function.
          </li>
          <li>
            Never hash passwords without a per-user salt. Same password
            ≠ same hash.
          </li>
          <li>
            Don't truncate cryptographic hashes to "save space" — you
            collapse the security margin proportionally.
          </li>
          <li>
            Don't use MD5 or SHA-1 anywhere security-relevant. Real-world
            collisions have been used to forge a CA certificate
            (Flame, 2012) and break PDF and JPEG integrity.
          </li>
          <li>
            Don't roll your own. Use the libraries
            (<code>libsodium</code>, language-native bindings to bcrypt
            / Argon2 / yescrypt).
          </li>
        </ul>
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
.bulletlist code {
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

/* Property cards */
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
  margin: 0;
  color: var(--tx);
  font-size: 11.5px;
  line-height: 1.6;
}
.prop code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 0 5px;
  font-size: 11px;
  color: #9cdcfe;
}
.prop em {
  color: var(--accent);
  font-style: normal;
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
.tbl strong {
  color: #fff;
}

/* Status pills inside tables */
.ok,
.warn,
.bad {
  display: inline-block;
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  border-radius: 999px;
  padding: 2px 8px;
}
.ok {
  background: color-mix(in srgb, #2ead6c 16%, transparent);
  color: #2ead6c;
  border: 1px solid #2ead6c;
}
.warn {
  background: color-mix(in srgb, #e8952a 16%, transparent);
  color: #e8952a;
  border: 1px solid #e8952a;
}
.bad {
  background: color-mix(in srgb, #d14c4c 16%, transparent);
  color: #d14c4c;
  border: 1px solid #d14c4c;
}

/* Code snippet */
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

/* Hash anatomy */
.parts {
  display: flex;
  flex-direction: column;
  gap: 6px;
  background: var(--sf);
  border: 1px solid var(--bd);
  border-radius: 8px;
  padding: 10px 12px;
  margin-bottom: 12px;
}
.part {
  display: grid;
  grid-template-columns: 240px 1fr;
  gap: 12px;
  font-size: 11.5px;
  align-items: baseline;
}
@media (max-width: 700px) {
  .part {
    grid-template-columns: 1fr;
    gap: 2px;
  }
}
.part span {
  font-family: var(--mono);
  color: var(--accent);
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 6px;
  word-break: break-all;
}
.part em {
  color: var(--tx);
  font-style: normal;
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
