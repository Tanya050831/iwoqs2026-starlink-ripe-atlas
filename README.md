# Inferring Starlink Latency Structure from Public RIPE Atlas Measurements


---

## About the datasets



Starlink measurement papers typically fall into a few camps:

| Approach | What it measures | Examples | Outsider reproducibility |
|----------|------------------|----------|---------------------------|
| **Owned dish / lab setup** | End-to-end RTT, throughput, handovers, bufferbloat | Michel et al. (IMC'22); Mohan et al., *A Multifaceted Look at Starlink Performance* (WWW'24) | Low — requires hardware at the edge |
| **Browser / crowd extensions** | User-side connectivity traces | Kassem et al. (IMC'22) | Medium — depends on volunteer installs |
| **Active terrestrial scanning** | Starlink ground/PoP network from outside | Wu et al., *Beneath the Heavens* (IWQoS'25) | Medium — needs scanning infrastructure |
| **Third-party platforms** | Re-analysis of existing probes | Richter et al. (IFIP Networking'25); Izhikevich et al. (SIGMETRICS'24) | Higher — if data/API are public |
| **This work** | **Passive** use of RIPE Atlas **built-in** root-DNS pings from Starlink AS probes | — | **High** — public API, no dish, no new measurement credits |

We deliberately chose inputs that are **public, re-downloadable, and free of proprietary Starlink access**:

1. **RIPE Atlas (Dataset 1)** — the **only** source of observed latency in this study. Built-in pings run continuously from volunteer probes; we filter probes on **AS14593** and aggregate RTT. 

2. **Public infrastructure corpus (Dataset 2)** — encodes **ground-system geography** using only disclosed filings and BGP/peering records. It lets us test a simple hypothesis from the literature (ground-station / PoP proximity vs. latency) **without** owning topology data from SpaceX. Prior work (e.g., ground-infrastructure proximity studies, *Beneath the Heavens*, gateway-distance analyses) uses richer private or scanned ground maps; we trade completeness for **auditability**.

3. **Starlink TLE catalogue (Dataset 3)** — standard **orbital ephemeris** for the constellation (CelesTrak). Many LEO studies (e.g., LENS, orbital visibility work) combine TLEs with latency to reason about satellite geometry. In our poster paper, TLEs are listed for completeness and optional SGP4 propagation; **Findings F1–F3 do not require TLEs**—the main results use Atlas RTT and $d_{\text{pub}}$ only.


---

## Public data sources (names, links, how to obtain)

No frozen dataset ships with this repository; everything below can be fetched from public endpoints.

### Dataset 1 — RIPE Atlas built-in pings (primary)

| | |
|--|--|
| **Name** | RIPE Atlas — built-in IPv4 ping measurements to DNS root anchors |
| **Operator** | [RIPE NCC](https://www.ripe.net/) |
| **Platform** | <https://atlas.ripe.net/> |
| **API manual** | <https://atlas.ripe.net/docs/apis/rest-api-manual/> |
| **Built-in measurements** | <https://atlas.ripe.net/docs/getting-started/built-in-measurements> |
| **Licence** | [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/) |

**What we use**

- Probes with `asn_v4 = 14593` (SpaceX Starlink): list via  
  `GET https://atlas.ripe.net/api/v2/probes/?asn_v4=14593&status=1&page_size=500`  
  Per-probe detail (includes `geometry`):  
  `GET https://atlas.ripe.net/api/v2/probes/<probe_id>/`
- Built-in ping results (~240 s cadence) for msm-ids **1001, 1004, 1010** → `k.root-servers.net`, `f.root-servers.net`, `b.root-servers.net`:  
  `GET https://atlas.ripe.net/api/v2/measurements/<msm_id>/results/`  
  (filter by `probe_ids` and time window in client code)
- **Paper window:** 7 days ending 2026-05-20 → **897,949** samples, **122** probes with sufficient data, **39** countries

**How to obtain:** public `GET` requests only; no Atlas credits required for reading these results. Register a RIPE NCC Access account only if you need higher rate limits or custom measurements.

---

### Dataset 2 — Public Starlink-related infrastructure corpus ($d_{\text{pub}}$)

| | |
|--|--|
| **Name** | Curated corpus of **42** publicly disclosed entities (gateways, inferred PoPs/peering points, IXPs) |
| **Role in paper** | For each probe, $d_{\text{pub}}$ = great-circle km to the **nearest** entity |
| **Licence** | Coordinates derived from public filings and routing data; curated list is our compilation |

**Source channels (manual curation)**

| Source | URL | Use |
|--------|-----|-----|
| FCC IBFS / ULS earth-station filings (Starlink Services LLC) | <https://wireless2.fcc.gov/UlsApp/UlsSearch/> · <https://www.fcc.gov/icfs/> | U.S. gateway coordinates |
| FCC public notices (STA filings) | [DOC-403340A1](https://docs.fcc.gov/public/attachments/DOC-403340A1.pdf) · [DOC-412437A2](https://docs.fcc.gov/public/attachments/DOC-412437A2.pdf) | Additional disclosed sites |
| BGP / AS14593 routing view | <https://bgp.tools/as/14593> | Non-U.S. peering / attachment hints |
| PeeringDB (AS14593) | <https://www.peeringdb.com/net/18747> | PoP / IX presence |
| National regulators & press releases | (varies by country) | Cross-check for newer gateways (e.g., Africa sites) |

**How to obtain:** there is **no single official SpaceX global gateway list**. Researchers typically **hand-compile** from the sources above. Coverage is dense in **NA/EU** and partial elsewhere—an intentional limitation discussed in the paper.

---

### Dataset 3 — Starlink TLE catalogue (optional / auxiliary)

| | |
|--|--|
| **Name** | CelesTrak Starlink group TLE snapshot |
| **Maintainer** | [CelesTrak](https://celestrak.org/) (Dr. T. S. Kelso) |
| **Download URL** | <https://celestrak.org/NORAD/elements/gp.php?GROUP=starlink&FORMAT=tle> |
| **Attribution** | <https://celestrak.org/webmaster.php> |
| **Paper snapshot** | 2026-05-20 → **10,360** catalogued elements |

**How to obtain**

```bash
curl -s -o starlink.tle \
  "https://celestrak.org/NORAD/elements/gp.php?GROUP=starlink&FORMAT=tle"
```

Anonymous HTTP GET; no registration for current TLEs. Historical ephemeris: register at <https://www.space-track.org/>.

**Role in this poster:** listed for completeness; **not used in Findings F1–F3** in the published analysis. Other Starlink studies often propagate TLEs (e.g., via SGP4) to relate latency to satellite visibility—we leave that as future work.

---

## Citation

```bibtex
@inproceedings{tu2026starlink,
  author    = {Tu, Jiaxian and Li, Yuxuan and Xu, Shenghe and Mei, Lifan},
  title     = {Inferring Starlink Latency Structure from Public {RIPE} Atlas Measurements},
  booktitle = {IEEE International Symposium on Quality of Service (IWQoS)},
  year      = {2026},
  note      = {Poster}
}
```

## Contact

- {Jiaxian.Tu23, Yuxuan.Li2302}@student.xjtlu.edu.cn  
- shenghexu@gmail.com · Lifan.Mei@xjtlu.edu.cn · 

## Licence

RIPE Atlas data: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). CelesTrak TLEs: U.S. government public-domain data republished with attribution. Paper text and figures: see repository licence.
