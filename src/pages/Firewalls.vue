<script setup>
// Page is a single-topic explainer — content lives directly in the
// template rather than a separate JSON file.
</script>

<template>
  <div class="page-shell" style="--accent: #e36b3c">
    <div class="page">
      <header class="hdr">
        <div class="icon">▮</div>
        <div>
          <h1>Firewalls</h1>
          <div class="sub">
            The policy boundary between networks — what traffic gets in,
            what gets out, and what gets dropped on the floor.
          </div>
        </div>
      </header>

      <div class="callout">
        <strong>Mental model</strong>
        A firewall is a <em>bouncer at the door</em>. Every packet shows
        its ID — source, destination, port, protocol, sometimes the full
        request — and is checked against a guest list. Match a rule that
        says <em>allow</em> and you're in; match nothing (or a
        <em>deny</em>) and you're turned away. The list is read top-down,
        first match wins, and the last line is almost always
        <strong>deny everything else</strong>.
      </div>

      <!-- What a firewall actually does -->
      <section class="section">
        <h2>What a firewall actually does</h2>
        <p class="prose">
          A firewall sits between two trust zones — typically your
          internal network and the internet, or a DMZ and your private
          subnets — and enforces a set of rules on every packet that
          crosses the boundary. The rules describe traffic by
          <strong>5-tuple</strong>: source IP, destination IP, source
          port, destination port, and protocol. Modern firewalls extend
          that with application identity, user identity, URL category,
          and even the file content inside the flow. Whatever it
          inspects, the verdict is the same three options:
          <code>ALLOW</code>, <code>DENY</code> (silent drop), or
          <code>REJECT</code> (drop and tell the sender).
        </p>
        <p class="prose">
          Firewalls aren't just perimeter devices anymore — they live on
          every host (Windows Defender Firewall, <code>iptables</code>,
          <code>nftables</code>), inside every cloud account (AWS
          Security Groups, Azure NSGs), and as virtual appliances inside
          Kubernetes. The core idea — <em>policy at a network choke
          point</em> — hasn't changed since the 1990s.
        </p>
      </section>

      <!-- Common types -->
      <section class="section">
        <h2>The four common types</h2>
        <p class="prose">
          Firewalls are usually grouped by <em>how deep they look into
          the traffic</em>. The deeper the inspection, the smarter the
          policy — and the more CPU it costs. Real production firewalls
          often combine several of these techniques in one box.
        </p>

        <div class="fw-grid">
          <article class="fw-card">
            <div class="fw-card__head">
              <h3>Stateless</h3>
              <span class="fw-card__alias">Packet filter · ACL</span>
            </div>
            <p class="fw-card__summary">
              Inspects each packet in isolation against a static rule
              list. No memory of prior packets — every packet is judged
              on its own 5-tuple.
            </p>
            <p class="fw-card__use">
              <strong>Strengths:</strong> blazing fast, trivial to
              implement, runs at line rate on almost any hardware.
              <br />
              <strong>Weaknesses:</strong> can't tell a legitimate reply
              from a spoofed packet — to allow return traffic you must
              open whole port ranges, which is loose.
            </p>
            <div class="fw-card__example">
              <span class="fw-card__lbl">Where you'll see it</span>
              <code>Router ACLs · AWS NACLs · early ipchains</code>
            </div>
          </article>

          <article class="fw-card">
            <div class="fw-card__head">
              <h3>Stateful</h3>
              <span class="fw-card__alias">Connection tracking</span>
            </div>
            <p class="fw-card__summary">
              Tracks every connection in a state table
              (<code>NEW</code>, <code>ESTABLISHED</code>,
              <code>RELATED</code>) and only allows packets that belong
              to a flow it's already approved.
            </p>
            <p class="fw-card__use">
              <strong>Strengths:</strong> rules stay simple — allow the
              outbound, the reply is automatically permitted. Catches
              spoofed packets that don't match any tracked flow.
              <br />
              <strong>Weaknesses:</strong> still works at L3/L4 — has no
              clue what's <em>inside</em> the flow.
            </p>
            <div class="fw-card__example">
              <span class="fw-card__lbl">Where you'll see it</span>
              <code>iptables · pf · Cisco ASA · AWS Security Groups</code>
            </div>
          </article>

          <article class="fw-card">
            <div class="fw-card__head">
              <h3>Proxy</h3>
              <span class="fw-card__alias">Application gateway</span>
            </div>
            <p class="fw-card__summary">
              Terminates the connection, opens its own to the
              destination, and proxies the traffic in between — so it
              can inspect the full L7 request and rewrite or block by
              content.
            </p>
            <p class="fw-card__use">
              <strong>Strengths:</strong> deepest possible inspection,
              hides internal clients, can authenticate users, can cache.
              <br />
              <strong>Weaknesses:</strong> protocol-specific (one proxy
              per service), high latency, breaks end-to-end TLS unless
              the client trusts the proxy's CA.
            </p>
            <div class="fw-card__example">
              <span class="fw-card__lbl">Where you'll see it</span>
              <code>Squid · Zscaler · Symantec ProxySG · WAFs</code>
            </div>
          </article>

          <article class="fw-card">
            <div class="fw-card__head">
              <h3>Next-Generation (NGFW)</h3>
              <span class="fw-card__alias">L7-aware · UTM</span>
            </div>
            <p class="fw-card__summary">
              Stateful firewall + deep packet inspection + IPS + TLS
              inspection + application identity (App-ID) + user
              identity + threat-intel feeds, all in one device.
            </p>
            <p class="fw-card__use">
              <strong>Strengths:</strong> rules read like
              <em>"allow Finance to use Salesforce, block everyone else
              from any social-media app"</em> rather than IP/port
              soup. Native IPS and malware sandboxing.
              <br />
              <strong>Weaknesses:</strong> expensive, performance
              tanks under TLS inspection, complex to tune.
            </p>
            <div class="fw-card__example">
              <span class="fw-card__lbl">Where you'll see it</span>
              <code>Palo Alto PAN-OS · Fortinet FortiGate · Cisco Secure Firewall</code>
            </div>
          </article>
        </div>
      </section>

      <!-- Inbound vs outbound -->
      <section class="section">
        <h2>Inbound vs outbound rules</h2>
        <p class="prose">
          Direction is always <em>relative to the protected zone</em>.
          For a server, <strong>inbound</strong> is traffic arriving at
          its NIC and <strong>outbound</strong> is traffic the server
          itself initiates. For a perimeter firewall in front of an
          office, <strong>inbound</strong> is internet → LAN and
          <strong>outbound</strong> is LAN → internet. Get the
          perspective wrong and your rules will be backwards.
        </p>

        <div class="split">
          <div class="split__col split__col--in">
            <h4>Inbound (Ingress)</h4>
            <ul>
              <li>
                Traffic <strong>entering</strong> the protected zone —
                an external client trying to reach an internal service.
              </li>
              <li>
                Default policy is almost always <strong>deny all</strong>
                — only services you've explicitly published should be
                reachable.
              </li>
              <li>
                Typical allows: <code>TCP/443</code> to a web server,
                <code>TCP/22</code> from a jump-host range,
                <code>UDP/53</code> to a DNS resolver.
              </li>
              <li>
                A bad inbound rule = an open door. Tighten the source
                whenever possible (<code>0.0.0.0/0</code> is a smell).
              </li>
            </ul>
          </div>
          <div class="split__col split__col--out">
            <h4>Outbound (Egress)</h4>
            <ul>
              <li>
                Traffic <strong>leaving</strong> the protected zone —
                an internal host calling out to something else.
              </li>
              <li>
                Historically left wide open ("anything internal can
                talk to anywhere on the internet"). Modern practice is
                <strong>egress filtering</strong> — also default-deny.
              </li>
              <li>
                Typical allows: <code>TCP/443</code> to known SaaS
                ranges, <code>UDP/123</code> to NTP pools,
                <code>TCP/443</code> from build agents to package
                repos.
              </li>
              <li>
                Strict egress kills the easy half of malware C2 —
                beacons can't phone home if port 443 to random IPs is
                blocked.
              </li>
            </ul>
          </div>
        </div>

        <p class="prose" style="margin-top: 14px">
          Stateful firewalls remove the need to write a matching
          <em>reverse</em> rule — once you allow an outbound flow, the
          reply is automatically tracked as
          <code>ESTABLISHED</code> and permitted back in. With a
          stateless filter you have to allow the return path explicitly,
          which is one of the main reasons stateful firewalls won.
        </p>
      </section>

      <!-- Anatomy of a rule -->
      <section class="section">
        <h2>Anatomy of a rule</h2>
        <p class="prose">
          A rule is read top-to-bottom; first match wins. Every modern
          firewall ends with an implicit
          <code>deny any any</code>. Below is a small ruleset for a web
          server in a DMZ — note how outbound is also locked down.
        </p>

        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>#</th>
                <th>Action</th>
                <th>Direction</th>
                <th>Proto</th>
                <th>Source</th>
                <th>Destination</th>
                <th>Port</th>
                <th>Note</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td>1</td>
                <td><code class="hl">ALLOW</code></td>
                <td>Inbound</td>
                <td>TCP</td>
                <td><code>0.0.0.0/0</code></td>
                <td><code>10.0.1.10</code></td>
                <td><code>443</code></td>
                <td>Public HTTPS</td>
              </tr>
              <tr>
                <td>2</td>
                <td><code class="hl">ALLOW</code></td>
                <td>Inbound</td>
                <td>TCP</td>
                <td><code>10.0.99.0/24</code></td>
                <td><code>10.0.1.10</code></td>
                <td><code>22</code></td>
                <td>SSH from jump-host VLAN only</td>
              </tr>
              <tr>
                <td>3</td>
                <td><code class="hl">ALLOW</code></td>
                <td>Outbound</td>
                <td>UDP</td>
                <td><code>10.0.1.10</code></td>
                <td><code>10.0.0.2</code></td>
                <td><code>53</code></td>
                <td>Internal DNS only</td>
              </tr>
              <tr>
                <td>4</td>
                <td><code class="hl">ALLOW</code></td>
                <td>Outbound</td>
                <td>TCP</td>
                <td><code>10.0.1.10</code></td>
                <td><code>repo.internal</code></td>
                <td><code>443</code></td>
                <td>Patch / package server</td>
              </tr>
              <tr>
                <td>5</td>
                <td><code class="deny">DENY</code></td>
                <td>Any</td>
                <td>Any</td>
                <td><code>any</code></td>
                <td><code>any</code></td>
                <td><code>any</code></td>
                <td>Implicit final rule</td>
              </tr>
            </tbody>
          </table>
        </div>
        <p class="note">
          Rule order matters. A broad <code>ALLOW</code> placed above a
          tighter <code>DENY</code> defeats the deny — the firewall
          stops at the first match.
        </p>
      </section>

      <!-- Common solutions -->
      <section class="section">
        <h2>Common firewall solutions</h2>
        <p class="prose">
          Firewalls live in three rough tiers — host-based, open-source
          appliances, and enterprise NGFW platforms — plus the cloud
          providers' own native primitives.
        </p>

        <div class="sol-grid">
          <article class="sol-card">
            <h3>Host / built-in</h3>
            <ul>
              <li>
                <strong>iptables / nftables</strong> — the Linux kernel
                packet filter. <code>nftables</code> is the modern
                successor; <code>iptables</code> is still everywhere.
              </li>
              <li>
                <strong>UFW</strong> — Ubuntu's friendly wrapper around
                nftables/iptables for simple host rules.
              </li>
              <li>
                <strong>firewalld</strong> — RHEL/Fedora zone-based
                management layer, also on top of nftables.
              </li>
              <li>
                <strong>pf</strong> — OpenBSD / FreeBSD / macOS packet
                filter. Powerful syntax, used inside many appliances.
              </li>
              <li>
                <strong>Windows Defender Firewall</strong> — built into
                every modern Windows host, profile-aware (Domain /
                Private / Public), driven by Group Policy.
              </li>
            </ul>
          </article>

          <article class="sol-card">
            <h3>Open-source appliances</h3>
            <ul>
              <li>
                <strong>pfSense</strong> — FreeBSD + pf + a slick web
                UI. Hugely popular for home labs and small offices;
                Netgate sells supported appliances.
              </li>
              <li>
                <strong>OPNsense</strong> — community fork of pfSense
                with a faster release cadence and a more modern UI.
              </li>
              <li>
                <strong>OpenWrt</strong> — Linux-based router OS that
                turns commodity hardware into a capable firewall +
                router.
              </li>
              <li>
                <strong>VyOS</strong> — Linux router OS with a
                Junos-style CLI; popular as a virtual edge router.
              </li>
            </ul>
          </article>

          <article class="sol-card">
            <h3>Enterprise NGFW</h3>
            <ul>
              <li>
                <strong>Palo Alto Networks (PAN-OS)</strong> — the
                reference NGFW; App-ID, User-ID, and WildFire sandbox
                are the hallmarks.
              </li>
              <li>
                <strong>Fortinet FortiGate</strong> — strong
                price/performance, custom ASICs, very common in
                mid-market.
              </li>
              <li>
                <strong>Cisco Secure Firewall</strong> (formerly
                Firepower / ASA) — long-standing enterprise staple,
                tightly integrated with Cisco's stack.
              </li>
              <li>
                <strong>Check Point Quantum</strong> — pioneer of
                stateful inspection in the 90s, still a major
                enterprise vendor.
              </li>
              <li>
                <strong>Sophos XG · Juniper SRX · WatchGuard ·
                SonicWall</strong> — other widely deployed NGFW lines.
              </li>
            </ul>
          </article>

          <article class="sol-card">
            <h3>Cloud-native</h3>
            <ul>
              <li>
                <strong>AWS Security Groups</strong> — instance-level
                stateful firewall (allow-only). <strong>NACLs</strong>
                add a stateless subnet-level layer.
                <strong>AWS Network Firewall</strong> is the managed
                NGFW / IPS service.
              </li>
              <li>
                <strong>Azure NSG</strong> — stateful subnet/NIC ACLs.
                <strong>Azure Firewall</strong> is the managed L3-L7
                service with threat intel.
              </li>
              <li>
                <strong>GCP VPC firewall rules</strong> — global,
                stateful, target by tag or service account.
                <strong>Cloud Armor</strong> is the L7 / WAF layer.
              </li>
              <li>
                <strong>Cloudflare Magic Firewall · Zscaler ZIA</strong>
                — cloud-delivered firewalls / SASE platforms that
                replace on-prem boxes for distributed orgs.
              </li>
            </ul>
          </article>
        </div>
      </section>

      <!-- Limitations -->
      <section class="section">
        <h2>What firewalls don't fix</h2>
        <p class="prose">
          A firewall enforces a policy on traffic that crosses it — it
          can't see anything else. Lateral movement <em>inside</em> a
          flat network bypasses the perimeter entirely. Encrypted
          traffic without TLS inspection is opaque even to NGFWs.
          Allowed protocols can carry malicious payloads (DNS tunnels,
          C2 over HTTPS, malware in legitimate cloud-storage uploads).
          And a misconfigured rule — one stray <code>0.0.0.0/0</code>
          — can erase years of careful policy in a single line. Treat
          the firewall as <strong>one layer</strong>; pair it with
          segmentation, IDS/IPS, EDR, and good logging.
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
  color: #2ead6c;
  border-color: #2ead6c;
}
.tbl code.deny {
  color: #d14c4c;
  border-color: #d14c4c;
}

/* Firewall type grid */
.fw-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 12px;
}
.fw-card {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 14px 14px 12px;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.fw-card__head {
  display: flex;
  align-items: baseline;
  justify-content: space-between;
  gap: 10px;
  flex-wrap: wrap;
}
.fw-card__head h3 {
  margin: 0;
  font-family: var(--sans);
  font-size: 13px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
}
.fw-card__alias {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--accent);
}
.fw-card__summary {
  margin: 0;
  font-size: 12px;
  color: var(--tx);
  line-height: 1.6;
}
.fw-card__use {
  margin: 0;
  font-size: 11.5px;
  color: var(--dm);
  line-height: 1.6;
}
.fw-card__use strong {
  color: var(--tx);
  font-weight: 600;
}
.fw-card__summary code,
.fw-card__use code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
}
.fw-card__example {
  display: flex;
  align-items: center;
  gap: 8px;
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-radius: 6px;
  padding: 6px 10px;
  margin-top: 2px;
}
.fw-card__lbl {
  font-family: var(--sans);
  font-size: 9px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--dm);
}
.fw-card__example code {
  background: transparent;
  border: 0;
  padding: 0;
  font-family: var(--mono);
  font-size: 11.5px;
  color: var(--accent);
  white-space: normal;
}

/* Inbound / outbound split */
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
.split__col--in {
  border-left: 2px solid #4d8fd6;
}
.split__col--in h4 {
  color: #4d8fd6;
}
.split__col--out {
  border-left: 2px solid #e36b3c;
}
.split__col--out h4 {
  color: #e36b3c;
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
.split__col code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
}

/* Solutions grid */
.sol-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 12px;
}
.sol-card {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 10px;
  padding: 14px 14px 12px;
}
.sol-card h3 {
  margin: 0 0 8px;
  font-family: var(--sans);
  font-size: 13px;
  font-weight: 700;
  color: #fff;
  letter-spacing: 0.02em;
}
.sol-card ul {
  margin: 0;
  padding-left: 18px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.sol-card li {
  font-size: 11.5px;
  color: var(--tx);
  line-height: 1.6;
}
.sol-card strong {
  color: #fff;
  font-weight: 600;
}
.sol-card code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 5px;
  font-size: 11px;
  color: #9cdcfe;
}
</style>
