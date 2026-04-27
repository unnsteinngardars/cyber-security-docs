<script setup>
import data from "../../database/ip-addressing.json";

const privateRanges = data.privateRanges;
const specialRanges = data.specialRanges;
const cidrTable = data.cidrTable;

const exampleAddress = {
  address: "192.168.10.130",
  binary: "11000000.10101000.00001010.10000010",
  octets: [
    { dec: "192", bin: "11000000" },
    { dec: "168", bin: "10101000" },
    { dec: "10",  bin: "00001010" },
    { dec: "130", bin: "10000010" }
  ]
};

const subnetExample = {
  cidr: "192.168.10.0/26",
  mask: "255.255.255.192",
  network: "192.168.10.0",
  firstHost: "192.168.10.1",
  lastHost: "192.168.10.62",
  broadcast: "192.168.10.63",
  totalAddresses: 64,
  usableHosts: 62,
  binaryNote: "/26 means the first 26 bits identify the network; the last 6 bits identify the host."
};
</script>

<template>
  <div class="page-shell" style="--accent: #3a8dd6">
    <div class="page">
      <header class="hdr">
        <div class="icon">⌘</div>
        <div>
          <h1>IP Addresses &amp; Subnets</h1>
          <div class="sub">
            How IPv4 addresses are structured, what subnet masks do, and how
            CIDR carves a network into smaller pieces.
          </div>
        </div>
      </header>

      <div class="callout">
        <strong>Mental model</strong>
        An IP address is the <em>house number</em>; the subnet mask is the
        <em>street</em>. The mask tells your machine which addresses are
        next door (deliver locally) and which need a router to reach.
      </div>

      <!-- IPv4 Anatomy -->
      <section class="section">
        <h2>IPv4 anatomy</h2>
        <p class="prose">
          An IPv4 address is <strong>32 bits</strong>, written as four
          decimal numbers (0–255) separated by dots — the
          <em>dotted-decimal</em> notation. Each number is one
          <strong>octet</strong> (8 bits). That's
          <code>2³² = 4,294,967,296</code> total addresses, of which large
          chunks are reserved (loopback, multicast, private use), so the
          public IPv4 space is much smaller in practice.
        </p>

        <div class="anatomy">
          <div class="anatomy__addr">{{ exampleAddress.address }}</div>
          <div class="anatomy__grid">
            <div
              v-for="(o, i) in exampleAddress.octets"
              :key="i"
              class="anatomy__cell"
            >
              <div class="anatomy__dec">{{ o.dec }}</div>
              <div class="anatomy__bin">{{ o.bin }}</div>
              <div class="anatomy__lbl">Octet {{ i + 1 }}</div>
            </div>
          </div>
        </div>
      </section>

      <!-- Public vs Private -->
      <section class="section">
        <h2>Public vs private</h2>
        <p class="prose">
          Most addresses on the internet are <strong>public</strong> —
          globally routable and unique. Three ranges are reserved as
          <strong>private</strong> (RFC 1918) for use inside organisations;
          they're not routed on the public internet, so the same
          <code>192.168.1.10</code> exists in millions of homes
          simultaneously. NAT translates between private inside addresses
          and a public outside address at the edge router.
        </p>

        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Range (CIDR)</th>
                <th>Addresses</th>
                <th>Count</th>
                <th>Use</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="r in privateRanges" :key="r.range">
                <td><code>{{ r.range }}</code></td>
                <td><code>{{ r.addresses }}</code></td>
                <td>{{ r.count }}</td>
                <td>{{ r.use }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Special / Reserved Ranges -->
      <section class="section">
        <h2>Reserved &amp; special-purpose ranges</h2>
        <p class="prose">
          Beyond public/private, parts of the IPv4 space are reserved for
          specific behaviours. Recognise these on sight — they show up
          constantly in firewall rules, DHCP troubleshooting, and packet
          captures.
        </p>

        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Range</th>
                <th>Name</th>
                <th>Purpose</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="r in specialRanges" :key="r.range">
                <td><code>{{ r.range }}</code></td>
                <td>{{ r.name }}</td>
                <td>{{ r.purpose }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Subnet masks & CIDR -->
      <section class="section">
        <h2>Subnet masks &amp; CIDR</h2>
        <p class="prose">
          A <strong>subnet mask</strong> splits a 32-bit IP address into a
          <em>network</em> portion (the leading 1-bits) and a
          <em>host</em> portion (the trailing 0-bits). All hosts that share
          the same network bits are on the same subnet and can talk
          directly; anything else must go through a router.
        </p>
        <p class="prose">
          <strong>CIDR notation</strong> (Classless Inter-Domain Routing)
          replaces the older Class A/B/C system with a flexible
          <code>/N</code> suffix that says how many leading bits are network
          bits. <code>192.168.1.0/24</code> means the first 24 bits are the
          network; the last 8 bits identify the host — giving 256 addresses
          (254 usable, since the first is the network address and the last
          is the broadcast).
        </p>

        <div class="formula">
          <div class="formula__row">
            <span class="formula__lbl">Total addresses</span>
            <code>2<sup>(32 − prefix)</sup></code>
          </div>
          <div class="formula__row">
            <span class="formula__lbl">Usable hosts</span>
            <code>2<sup>(32 − prefix)</sup> − 2</code>
            <span class="formula__note">
              (subtract network and broadcast, except /31 P2P and /32 host routes)
            </span>
          </div>
        </div>
      </section>

      <!-- CIDR cheat-sheet -->
      <section class="section">
        <h2>CIDR cheat-sheet</h2>
        <p class="prose">
          The prefix lengths you'll actually see in the wild — from
          provider aggregation (/8–/22) down through the LAN block (/24)
          and into individual subnets and point-to-point links.
        </p>

        <div class="table-wrap">
          <table class="tbl">
            <thead>
              <tr>
                <th>Prefix</th>
                <th>Subnet mask</th>
                <th>Wildcard</th>
                <th>Total addresses</th>
                <th>Usable hosts</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="r in cidrTable" :key="r.prefix">
                <td><code class="hl">{{ r.prefix }}</code></td>
                <td><code>{{ r.mask }}</code></td>
                <td><code>{{ r.wildcard }}</code></td>
                <td>{{ r.totalHosts }}</td>
                <td>{{ r.usableHosts }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </section>

      <!-- Worked example -->
      <section class="section">
        <h2>Worked example — {{ subnetExample.cidr }}</h2>
        <p class="prose">{{ subnetExample.binaryNote }}</p>

        <div class="example">
          <div class="example__row">
            <span class="example__lbl">CIDR</span>
            <code class="hl">{{ subnetExample.cidr }}</code>
          </div>
          <div class="example__row">
            <span class="example__lbl">Subnet mask</span>
            <code>{{ subnetExample.mask }}</code>
          </div>
          <div class="example__row">
            <span class="example__lbl">Network address</span>
            <code>{{ subnetExample.network }}</code>
            <span class="example__note">first address — identifies the subnet</span>
          </div>
          <div class="example__row">
            <span class="example__lbl">First usable host</span>
            <code>{{ subnetExample.firstHost }}</code>
          </div>
          <div class="example__row">
            <span class="example__lbl">Last usable host</span>
            <code>{{ subnetExample.lastHost }}</code>
          </div>
          <div class="example__row">
            <span class="example__lbl">Broadcast</span>
            <code>{{ subnetExample.broadcast }}</code>
            <span class="example__note">last address — sent to all hosts on the subnet</span>
          </div>
          <div class="example__row">
            <span class="example__lbl">Total addresses</span>
            <code>{{ subnetExample.totalAddresses }}</code>
          </div>
          <div class="example__row">
            <span class="example__lbl">Usable hosts</span>
            <code>{{ subnetExample.usableHosts }}</code>
          </div>
        </div>
      </section>

      <!-- IPv6 brief -->
      <section class="section">
        <h2>What about IPv6?</h2>
        <p class="prose">
          IPv6 addresses are <strong>128 bits</strong> — written as eight
          groups of four hex digits separated by colons
          (<code>2001:0db8:85a3::8a2e:0370:7334</code>). The same CIDR
          notation applies (e.g. <code>/64</code> is the standard subnet
          size). With 2¹²⁸ addresses there's no realistic exhaustion, so
          NAT mostly disappears and every device can have a globally
          routable address. Subnetting concepts carry over — the network
          and host split, mask-based reachability, prefix delegation.
        </p>
      </section>

      <footer class="ft">
        Reference data sourced from <code>database/ip-addressing.json</code>
      </footer>
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

/* IPv4 anatomy */
.anatomy {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-radius: 10px;
  padding: 14px 16px;
  margin-top: 8px;
}
.anatomy__addr {
  font-family: var(--mono);
  font-size: 18px;
  font-weight: 600;
  color: #fff;
  text-align: center;
  letter-spacing: 0.04em;
  margin-bottom: 12px;
}
.anatomy__grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 8px;
}
@media (max-width: 640px) {
  .anatomy__grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
.anatomy__cell {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-top: 2px solid var(--accent);
  border-radius: 6px;
  padding: 10px 8px;
  text-align: center;
}
.anatomy__dec {
  font-family: var(--mono);
  font-size: 16px;
  color: var(--accent);
  font-weight: 700;
}
.anatomy__bin {
  font-family: var(--mono);
  font-size: 11px;
  color: var(--tx);
  margin: 4px 0 6px;
  letter-spacing: 0.05em;
}
.anatomy__lbl {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 600;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: var(--dm);
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

/* Formula box */
.formula {
  background: var(--sf2);
  border: 1px solid var(--bd);
  border-left: 2px solid var(--accent);
  border-radius: 8px;
  padding: 12px 14px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.formula__row {
  display: flex;
  align-items: baseline;
  gap: 10px;
  flex-wrap: wrap;
  font-size: 12px;
  color: var(--tx);
}
.formula__lbl {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--accent);
  min-width: 130px;
}
.formula__row code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 6px;
  font-size: 12px;
  color: #9cdcfe;
}
.formula__note {
  font-size: 11px;
  color: var(--dm);
  font-style: italic;
}

/* Worked example */
.example {
  background: var(--sf);
  border: 1px solid var(--bd);
  border-left: 3px solid var(--accent);
  border-radius: 10px;
  padding: 14px 16px;
  display: flex;
  flex-direction: column;
  gap: 6px;
}
.example__row {
  display: grid;
  grid-template-columns: 160px auto 1fr;
  gap: 12px;
  align-items: baseline;
  font-size: 12px;
  color: var(--tx);
}
@media (max-width: 640px) {
  .example__row {
    grid-template-columns: 1fr;
    gap: 2px;
  }
}
.example__lbl {
  font-family: var(--sans);
  font-size: 9.5px;
  font-weight: 700;
  letter-spacing: 0.1em;
  text-transform: uppercase;
  color: var(--dm);
}
.example__row code {
  background: var(--bg);
  border: 1px solid var(--bd);
  border-radius: 4px;
  padding: 1px 6px;
  font-size: 12px;
  color: #9cdcfe;
  justify-self: start;
}
.example__row code.hl {
  color: var(--accent);
  border-color: var(--accent);
}
.example__note {
  font-size: 11px;
  color: var(--dm);
  font-style: italic;
}
</style>
