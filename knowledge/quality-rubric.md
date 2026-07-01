# Quality Rubric — How the Agent Judges "Best Quality"

This is the **foundation** of the sourcing agent: the operational definition of quality and
the rules the agent uses to judge, score, and rank any garment listing. Everything else
(search, translation, ranking, buy/skip decisions) hangs off this.

The machine-readable encoding lives in:
- `data/quality-model.yaml` — the material-agnostic scoring framework (axes, weights, decisions)
- `data/materials/*.yaml` — per-material thresholds (`cashmere.yaml`, `cotton.yaml`, `silk.yaml`)

This document is the human-readable rationale + citations. When they disagree, the YAML wins
(it is what the agent loads); update both together.

> **Design principle:** "Best quality" is **not** the highest price, the biggest brand, or the
> heaviest garment. It is *fiber quality × honest construction × verified authenticity ÷ price*.
> The whole point is to buy the fiber the luxury houses buy, without the label markup — so the
> agent must reason about **fibers and specs**, not marketing.

---

## The scoring pipeline

1. **Authenticity gate (first, hard).** Real cashmere/cotton/silk has a fiber-cost floor and a
   spec fingerprint. A listing that trips a `hard_fail` rule (e.g. imitation fiber, "100%" claim
   contradicted by composition, price below the fiber floor) is **REJECTED** — score 0. A
   `suspect` signal (recycled fiber, undisclosed blend, no specs, unverifiable "Egyptian"/"6A")
   routes to **VERIFY**, never auto-buy.
2. **Four weighted axes** produce a 0–100 score:
   - **fiber_quality (0.40)** — the intrinsic fiber. The dominant driver.
   - **construction (0.25)** — how fiber becomes cloth (ply/gauge/knit, weave, finishing).
   - **spec_transparency (0.20)** — does the seller publish the specs a real mill lists?
   - **value (0.15)** — quality-per-price vs the material's expected band.
3. **Score → tier → decision** (`exceptional/strong/acceptable/weak/poor` → `buy/verify/skip`).

Why these weights: across all three fibers, the single largest quality determinant is the raw
fiber (micron+length for cashmere, staple length for cotton, variety+momme for silk).
Construction can elevate or waste good fiber but cannot create it. Spec transparency is weighted
heavily on purpose — on Taobao/1688 the *absence* of technical specs is itself a strong negative
signal, because legitimate mills list them.

---

## Cashmere (`data/materials/cashmere.yaml`)

**Definition (not just grading).** To legally be "cashmere" (CCMI / US FTC 16 CFR / SFA): mean
fiber diameter **≤19 µm**, CV ≤24%, and ≤3% by weight over 30 µm. Grading sits on top of this.

**Fiber quality — the two levers:**
- **Micron (diameter, finer = softer):** Grade A **<15.5–16 µm**, Grade B **16–19 µm**, Grade C
  **19–30 µm**. **Baby cashmere ≈13.5 µm** (kid goats <1yr) is a separate, top category — the
  Loro-tier "supersoft." Origin is a proxy when micron isn't listed: Alashan/Mongolian ~15.0–15.5 µm.
- **Staple length (longer = less pilling, more durable):** Grade A **≥34 mm** (best 36 mm+),
  B **28–34 mm**, C **<28 mm**. Fibers under ~30 mm migrate to the surface and pill; short-staple
  garments visibly degrade after ~15–20 wears.

**Construction ≠ quality (critical):** **Ply (股) is not a grade.** 2-ply isn't "better" than
1-ply — it's *heavier/warmer*. Judge ply, yarn count (Nm, e.g. `2/26NM`), gauge (针; 7/9/12GG),
and GSM (克重) against the **weight tier the buyer wants**, not as more=better. Weight tiers:
base 180–220 GSM (1/light-2-ply), mid 260–320 (2-ply), heavy 350–450 (3–4-ply). *(The lightweight
GSM ceiling is disputed across sources, ~190 vs ~280.)*

**Authenticity — this market is actively adulterated:**
- **仿羊绒** = imitation (acrylic) → hard fail. **再生羊绒** = recycled/regenerated (ground-up
  fibers <35 mm → pills, weak; often 60–70% recycled + 30–40% virgin) → suspect.
- **China rule GB/T 29862-2013:** a product may be labeled "100%/纯/全山羊绒" only if cashmere
  content is **≥95%** (≤5% incidental wool, no synthetic/other-animal fiber).
- **Real, documented fraud:** A Feb 2025 CCTV 每周质量报告 investigation bought 7 livestream
  garments sold as "100% 山羊绒" — **all 7 contained zero cashmere** (tested as acrylic/polyester/
  nylon/viscose), sold at ~¥100–200.
- **Price floor:** raw Grade-A fiber ~$80–150/kg, processed yarn ~$150–300/kg; a sweater carries
  ~0.3–0.5 kg of fiber. "Cashmere" priced like wool ($10–30/kg equivalent) is fake/heavily blended.
- **Trust signal to prefer:** a seller-provided **质检报告** (fiber-composition test report) from a
  **CMA/CNAS-accredited** lab, tied to that specific product. Buyer-side burn test (burnt-hair smell
  + soft ash = real; melts to hard bead + chemical smell = synthetic) and the squiggle test are
  indicative only.
- *A little initial pilling is normal for real cashmere; heavy pilling after 15–20 wears = low grade.*

---

## Cotton (`data/materials/cotton.yaml`)

**Fiber quality = staple length.** Extra-long-staple (ELS) floor is **1-3/8 in = 34.9 mm**
(industry/trade; USDA's looser ELS is >1.25 in / 31.75 mm — use 34.9 mm). Upland commodity
(~90% of world crop) is ~25–32 mm; top Giza can exceed 40 mm.
- **Prefer Supima over "Egyptian."** Supima = trademarked American Pima ELS (~35 mm) with a
  **licensed, traceable** supply chain → a Supima claim is verifiable. "Egyptian cotton" is an
  **unregulated** label; DNA testing found **~89%** of goods sold as such were non-compliant.

**Construction:** **精梳 (combed)** removes ~15–20% short fibers (noil) → smoother, stronger,
less pilling (~15–25% pricier) — a real quality signal vs **普梳 (carded)**. **丝光 (mercerized)**
adds sheen, +20–25% tensile strength, and ~25% deeper/faster dye uptake. For knits/tees judge by
**GSM**: light 100–150, mid 150–180, heavy 180–240+ (premium tees 180–220+). For woven shirting,
**thread count 200–400 is the honest sweet spot**; single-ply tops out ~400–450, so advertised
**800–1500+ is inflated multi-ply counting** and a *negative* signal.

---

## Silk (`data/materials/silk.yaml`)

**The spec triad:** **momme + variety + weave.** Momme is the one *objective, measurable* lever;
grade (6A) is mostly marketing.
- **Momme (姆米; 1 momme = 4.34 g/m²):** <20 lightweight, 20–28 midweight, >28 heavy. Apparel/
  bedding mainstream is **19–25mm**; **22mm** most popular, **25mm** the premium sweet spot;
  19mm loses its finish faster under friction.
- **Variety:** **桑蚕丝 (mulberry, Bombyx mori, ~8 µm)** = highest quality, for anything on skin.
  **柞蚕丝 (tussah/wild, ~70 µm)** is coarse, pills, and weakens — fine for fillings/outerwear only.
- **Grade (6A/5A…, GB/T 1797):** applies **only to raw silk before weaving**, is seller-declared and
  unaudited — real 6A is **<1% of production** yet appears on **>20%** of products. Give "6A" little
  weight without a test report.
- **Weave:** charmeuse (缎; glossy front/matte back; best drape+sheen), crepe de chine (双绉;
  pebbled, wrinkle-resistant), habotai (电力纺; lightest, for linings/scarves).
- **仿真丝** = imitation silk (polyester) → hard fail.

---

## Extending to new materials

Add a `data/materials/<name>.yaml` following the same shape: `fiber_quality` (the intrinsic
levers), `construction`, `authenticity` (hard_fail / suspect / price_floor), and `listing_specs`
(the Chinese terms the agent extracts). No change to `quality-model.yaml` is needed — the scoring
framework is material-agnostic by design. Then add a short section here with citations.

---

## Sources & confidence notes

Cross-checked across multiple independent sources per claim. Several primary vendor/standards pages
return HTTP 403 to automated fetch, so some figures come from search-engine extracts corroborated by
3+ sources. Known disagreements are flagged inline (cashmere lightweight GSM ceiling; USDA vs trade
ELS threshold; silk momme ranges by fabric).

**Cashmere:** [SFA — what makes cashmere](https://sustainablefibre.org/what-makes-cashmere-cashmere/) ·
[Alpine Cashmere grades](https://www.alpinecashmere.com/blogs/everything-cashmere/grades-of-cashmere-explained) ·
[Diamond Knitland grades](https://diamondknitland.com/cashmere-quality-grades/) ·
[Diamond Knitland ply](https://diamondknitland.com/cashmere-ply/) ·
[Jet&Bo ply & gauge](https://www.jetandbo.com/blogs/all-about-cashmere/cashmere-ply-and-gauge-explained) ·
[Papini — regenerated cashmere](https://papinicashmere.com/regenerated-cashmere/) ·
[GOBI — spot fake / pilling](https://www.gobicashmere.com/blogs/news/how-to-spot-fake-cashmere/) ·
[Oats & Rice — real cashmere](https://www.oatsandrice.com/what-is-real-cashmere) ·
CCTV 每周质量报告 Feb 2025 (via [163.com](https://www.163.com/dy/article/JOJD3B0H05118I96.html)) ·
GB/T 29862-2013, GB 18267, FZ/T 01057-2007, ISO 17751-1/2 (standards) ·
[PLOS One — cashmere/yak DNA ID](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4720366/).

**Cotton:** [ITC Cotton Guide (ELS)](https://cottonguide.org/) ·
[Supima FAQ](https://supima.com/) ·
[SELVANE — how cotton quality is measured](https://www.selvane.co/blogs/knowledge/) ·
[fabric-supplier — yarn count & GSM](https://fabric-supplier.com/fabric-weight/) ·
[SewGuide / NapLab — thread count inflation] ·
[Britannica — mercerization](https://www.britannica.com/technology/mercerization).

**Silk:** [Mulberry Park — momme & grades](https://mulberryparksilks.com/blogs/mulberry/what-is-momme-in-silk-fabric) ·
[Mayfairsilk — mulberry vs tussah](https://mayfairsilk.com/blogs/general/mulberry-silk-vs-tussah-silk-differences-prices-pros-cons) ·
[SILKSILKY — 6A/5A grades](https://silksilky.com/blogs/silksilky-living/silk-grades-6a-5a-4a-explained) ·
[SilkUA — grade vs momme (marketing caveat)](https://www.silkua.com/mulberry-silk-grades-vs-momme-private-label/) ·
[SELVANE — charmeuse vs crepe vs habotai](https://www.selvane.co/blogs/knowledge/charmeuse-vs-crepe-de-chine-vs-habotai-silk-weave-guide) · GB/T 1797 (raw silk grading standard).
