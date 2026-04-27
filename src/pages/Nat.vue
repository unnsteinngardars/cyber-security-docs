<script setup>
// Page is a single-topic explainer — content lives directly in the
// template rather than a separate JSON file.
</script>

<template>
  <div class="page-shell" style="--accent: #2ead6c">
    <div class="page">
      <header class="hdr">
        <div class="icon">⇄</div>
        <div>
          <h1>NAT — Network Address Translation</h1>
          <div class="sub">
            How a router rewrites IP addresses (and ports) so many private
            hosts can share a single public address — the glue that's kept
            IPv4 alive.
          </div>
        </div>
      </header>

      <div class="callout">
        <strong>Mental model</strong>
        NAT is a <em>receptionist</em>. Every internal host's outbound packet
        is logged ("Alice on line 1") and stamped with the office's single
        public number. Replies come back to that number, and the
        receptionist looks up who originally placed the call and forwards
        the response.
      </div>

      <!-- Why NAT exists -->
      <section class="section">
        <h2>Why NAT exists</h2>
        <p class="prose">
          IPv4 has only <code>2³² ≈ 4.3 billion</code> addresses and we
          burned through them years ago. NAT is the workaround: instead of
          giving every device a unique public address, an organisation uses
          <strong>private RFC 1918 space</strong> internally
          (<code>10.0.0.0/8</code>, <code>172.16.0.0/12</code>,
          <code>192.168.0.0/16</code>) and translates to one or a few public
          addresses at the edge. A single home router with one ISP-issued
          IP can comfortably hide a phone, laptop, smart TV, console, and a
          dozen IoT devices behind it.
        </p>
      </section>

      <!-- How NAT works -->
      <section class="section">
        <h2>How NAT actually works</h2>
        <p class="prose">
          The router keeps a <strong>translation table</strong>. When an
          inside host sends an outbound packet, the router rewrites the
          source IP (and, for PAT, the source port) to its own public
          address, records the mapping, and forwards the packet. When a
          reply arrives, it looks up the destination port in the table to
          find the original inside host, rewrites the destination back, and
          delivers it. No mapping → packet is dropped.
        </p>

        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Proto</th>
                <th>Inside source</th>
                <th>Inside dest</th>
                <th>Outside source (rewritten)</th>
                <th>Outside dest</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><code class="hl">TCP</code></td>
                <td><code>10.0.0.5:51000</code></td>
                <td><code>93.184.216.34:443</code></td>
                <td><code class="hl">203.0.113.1:40123</code></td>
                <td><code>93.184.216.34:443</code></td>
              </tr>
              <tr>
                <td><code class="hl">UDP</code></td>
                <td><code>10.0.0.7:55020</code></td>
                <td><code>1.1.1.1:53</code></td>
                <td><code class="hl">203.0.113.1:40124</code></td>
                <td><code>1.1.1.1:53</code></td>
              </tr>
              <tr>
                <td><code class="hl">TCP</code></td>
                <td><code>10.0.0.5:51001</code></td>
                <td><code>140.82.114.4:22</code></td>
                <td><code class="hl">203.0.113.1:40125</code></td>
                <td><code>140.82.114.4:22</code></td>
              </tr>
            </tbody>
          </table>
        </div>
        <p class="note">
          Notice the inside destination never changes — only the source
          side is rewritten on the way out, and only the destination side
          is rewritten on the way back in.
        </p>
      </section>

      <!-- Types of NAT -->
      <section class="section">
        <h2>Flavours of NAT</h2>
        <p class="prose">
          "NAT" is an umbrella term — the specific behaviour depends on
          how mappings are allocated and whether ports are involved. PAT
          (port translation, also called NAPT or "overload") is what most
          people actually mean when they say NAT today.
        </p>

        <div class="nat-grid">
          <article class="nat-card">
            <div class="nat-card__head">
              <h3>Static NAT</h3>
              <span class="nat-card__alias">1-to-1 NAT</span>
            </div>
            <p class="nat-card__summary">
              Permanent fixed mapping between one private address and one
              public address.
            </p>
            <p class="nat-card__use">
              <strong>Use:</strong> publishing an internal server (web,
              mail) on a dedicated public IP. Both directions work — the
              host is reachable from the outside.
            </p>
            <div class="nat-card__example">
              <span class="nat-card__lbl">Example</span>
              <code>10.0.0.10 ⇄ 203.0.113.10</code>
            </div>
          </article>

          <article class="nat-card">
            <div class="nat-card__head">
              <h3>Dynamic NAT</h3>
              <span class="nat-card__alias">Pool NAT</span>
            </div>
            <p class="nat-card__summary">
              Maps each internal host to the next free address from a pool
              of public IPs.
            </p>
            <p class="nat-card__use">
              <strong>Use:</strong> when you have fewer public IPs than
              internal hosts but still want unique outside addresses.
              Mappings expire when idle and are returned to the pool.
            </p>
            <div class="nat-card__example">
              <span class="nat-card__lbl">Example</span>
              <code>10.0.0.0/24 → pool { 203.0.113.10–203.0.113.20 }</code>
            </div>
          </article>

          <article class="nat-card">
            <div class="nat-card__head">
              <h3>PAT / NAPT</h3>
              <span class="nat-card__alias">Overload · Masquerade</span>
            </div>
            <p class="nat-card__summary">
              Many internal hosts share a single public IP — uniqueness is
              preserved by rewriting the source port.
            </p>
            <p class="nat-card__use">
              <strong>Use:</strong> what every home router does. Lets
              thousands of devices share one ISP-issued public address.
            </p>
            <div class="nat-card__example">
              <span class="nat-card__lbl">Example</span>
              <code>10.0.0.5:51000 → 203.0.113.1:40123</code>
            </div>
          </article>

          <article class="nat-card">
            <div class="nat-card__head">
              <h3>Destination NAT</h3>
              <span class="nat-card__alias">DNAT · Port forwarding</span>
            </div>
            <p class="nat-card__summary">
              Rewrites the destination address (and optionally port) of
              inbound traffic to reach an internal service.
            </p>
            <p class="nat-card__use">
              <strong>Use:</strong> exposing one port on the public IP to
              an internal host, e.g. forwarding TCP/443 on the WAN to
              <code>10.0.0.10:443</code>.
            </p>
            <div class="nat-card__example">
              <span class="nat-card__lbl">Example</span>
              <code>203.0.113.1:443 → 10.0.0.10:443</code>
            </div>
          </article>

          <article class="nat-card">
            <div class="nat-card__head">
              <h3>CGNAT</h3>
              <span class="nat-card__alias">Carrier-Grade NAT · LSN</span>
            </div>
            <p class="nat-card__summary">
              ISPs run a second NAT layer — subscribers get a
              <code>100.64.0.0/10</code> address, then share a public IP at
              the carrier edge.
            </p>
            <p class="nat-card__use">
              <strong>Use:</strong> mobile networks and modern fixed-line
              ISPs that have run out of public IPv4. Expect double-NAT,
              breaks inbound services for end users.
            </p>
            <div class="nat-card__example">
              <span class="nat-card__lbl">Example</span>
              <code>10.0.0.5 → 100.64.13.7 → 198.51.100.42</code>
            </div>
          </article>
        </div>
      </section>

      <!-- Pros & Cons -->
      <section class="section">
        <h2>Trade-offs</h2>
        <p class="prose">
          NAT was a pragmatic hack that ended up everywhere. It buys
          address conservation and an accidental firewall, but it breaks
          the original internet model where every host could reach every
          other host directly.
        </p>

        <div class="split">
          <div class="split__col split__col--good">
            <h4>Strengths</h4>
            <ul>
              <li>
                Conserves public IPv4 — one address can serve hundreds of
                devices.
              </li>
              <li>
                Implicit firewall: unsolicited inbound traffic has no
                mapping and is dropped.
              </li>
              <li>Hides internal addressing from the outside world.</li>
              <li>
                Lets you renumber inside without coordinating with your
                ISP.
              </li>
            </ul>
          </div>
          <div class="split__col split__col--bad">
            <h4>Weaknesses</h4>
            <ul>
              <li>
                Breaks end-to-end connectivity — peer-to-peer apps need
                <strong>STUN / TURN / ICE</strong> or UPnP.
              </li>
              <li>
                Stateful: the NAT box must remember every flow;
                resource-bound and a target for state-table exhaustion.
              </li>
              <li>
                Complicates protocols that embed addresses in the payload
                (FTP, SIP, H.323) — needs ALGs.
              </li>
              <li>
                Logging and attribution are harder — many users behind one
                IP.
              </li>
              <li>Double-NAT (CGNAT) compounds every problem above.</li>
            </ul>
          </div>
        </div>
      </section>

      <!-- NAT and security -->
      <section class="section">
        <h2>NAT vs a firewall</h2>
        <p class="prose">
          A common misconception: <em>"I'm behind NAT, so I'm safe."</em>
          NAT does drop unsolicited inbound traffic by default — there's
          no mapping for it — but that's a side effect, not a security
          feature. NAT does not inspect packet content, doesn't enforce
          policy on outbound traffic, and any host that initiates outbound
          can be reached on its return path. Treat NAT as
          <strong>defence-in-depth</strong>, not a substitute for a real
          stateful firewall with explicit rules.
        </p>
      </section>

      <!-- IPv6 -->
      <section class="section">
        <h2>What about IPv6?</h2>
        <p class="prose">
          With <code>2¹²⁸</code> addresses, IPv6 has no shortage to solve,
          so NAT mostly disappears — every device can have a globally
          routable address. Hosts are still protected by a stateful
          firewall (almost every IPv6 home router defaults to "block
          inbound"), but the address-rewriting machinery is gone. NAT66
          exists but is discouraged. End-to-end connectivity returns,
          which is great news for VoIP, gaming, and peer-to-peer
          protocols that NAT made painful.
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
.note {
  font-size: 11px;
  color: var(--dm);
  font-style: italic;
  margin: 8px 0 0;
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

/* NAT type grid */
.nat-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 12px;
}
.nat-card {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 14px 14px 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.nat-card__head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 10px;
  flex-wrap: wrap;
}
.nat-card__head h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 13px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
}
.nat-card__alias {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--accent);
}
.nat-card__summary {
  margin: 0;
  font-size: 12px;
  color: var(--tx);
  line-height: 1.6;
}
.nat-card__use {
  margin: 0;
  font-size: 11.5px;
  color: var(--dm);
  line-height: 1.6;
}
.nat-card__use code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
}
.nat-card__use strong {
  color: var(--tx);
  font-weight: 600;
}
.nat-card__summary code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
}
.nat-card__example {
  display: flex;
  align-items: center;
  gap: 8px;
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 6px;
  padding: 6px 10px;
  margin-top: 2px;
}
.nat-card__lbl {
  font-family: var(--sans);
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--dm);
}
.nat-card__example code {
  background: transparent;
  border: 0;
  padding: 0;
  font-family: var(--mono);
  font-size: 11.5px;
  color: var(--accent);
}

/* Pros/cons split */
.split {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
}
@media (max-width: 640px) {
  .split {
    grid-template-columns: 1fr;
  }
}
.split__col {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-radius: 10px;
  padding: 12px 14px;
}
.split__col h4 {
  margin: 0 0 8px;
  font-family: var(--sans);
  font-size: 10px;
  font-weight: 700;
  letter-spacing: 0.12em;
  text-transform: uppercase;
}
.split__col--good {
  border-left: 2px solid #2ead6c;
}
.split__col--good h4 {
  color: #2ead6c;
}
.split__col--bad {
  border-left: 2px solid #d14c4c;
}
.split__col--bad h4 {
  color: #d14c4c;
}
.split__col ul {
  margin: 0;
  padding-left: 18px;
  display: flex;
  flex-direction: column;
  gap: 5px;
}
.split__col li {
  font-size: 11.5px;
  color: var(--tx);
  line-height: 1.6;
}
.split__col strong {
  color: #fff;
}
</style>
