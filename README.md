# XCOM · Photon Cross-Section Explorer

A single-file, browser-based tool for exploring photon attenuation and designing radiation shielding, built on the **NIST XCOM** photon cross-section database and **NIST SRD 126 Table 4** (mass attenuation and mass energy-absorption coefficients).

Everything runs on your own computer, in the browser. There is no server, no build step and no installation, and your data is never uploaded.

---

## Features

The app has eight views:

| View | What it does |
|---|---|
| **Explorer** | Plot μ/ρ and its interaction channels (coherent, incoherent, photoelectric, pair production) and μ<sub>en</sub>/ρ from 1 keV to 100 GeV for any element, compound or mixture. Includes a probe readout, a tabulated curve and CSV export. |
| **Shielding** | Calculate μ, transmission, HVL, TVL and the thickness needed to reach a target transmission. Also covers material comparison, multilayer stacks, layer contributions, buildup, secondary-radiation and K-edge indicators, stack optimization and layer-order comparison. |
| **Shield 2D** | Trace parallel or point-source rays through a multilayer slab stack to a strip detector at oblique incidence. Includes the 2D intensity field, depth profile, energy transmission profile, a 3D surface of I/I₀(z, E), and channel fractions. |
| **Shield 3D** | Trace an N × N ray grid from a parallel, point or isotropic source onto a 3D detector: rectangle, disk, cylinder, sphere, pixel array or user-defined polygon. Panels show the detector-plane footprint, the detector response (fluence, kerma, dose, counts and more) and per-layer interaction analysis. |
| **Compare** | Overlay up to four materials on one energy-dependent plot, with numerical readouts. |
| **Edges** | Find elements whose absorption edge lies near a target energy, for example to choose a K-edge filter or contrast agent. |
| **Theory** | 29 sections on the physics, equations, algorithms and limitations of the model. |
| **Manual** | A control-by-control guide to every view. |

Materials can be entered as:
- **Elements**, Z = 1–100, picked from a periodic table
- **Compounds** in XCOM notation, e.g. `H2O`, `CaWO4`, `SiO2`. Parentheses aren't accepted.
- **Mixtures**, as formula and weight-fraction pairs. Weights are normalized automatically.
- **NIST Table 4 materials**: 48 directly tabulated compounds and mixtures, 1 keV–20 MeV
- **Presets** for common shielding materials, tissues, polymers, scintillators, semiconductors and more

---

## Quick start

1. Download `xcom.html`.
2. Open it in a modern browser (Chrome, Edge, Firefox or Safari).

You can also serve it locally:

```bash
python3 -m http.server 8000
# then open http://localhost:8000/xcom.html
```

To host it on **GitHub Pages**, rename the file to `index.html`, push it to your repository and turn on Pages under *Settings → Pages*.

> **Internet connection:** on first load the page fetches Plotly, MathJax, pako and its web fonts from public CDNs. All data and calculations are already inside the file.

---

## Data sources

| Dataset | Use |
|---|---|
| [NIST XCOM: Photon Cross Sections Database](https://physics.nist.gov/PhysRefData/Xcom/html/xcom1.html), M. J. Berger & J. H. Hubbell | Partial and total cross sections for Z = 1–100, 1 keV–100 GeV |
| [NIST SRD 126 Table 4](https://physics.nist.gov/PhysRefData/XrayMassCoef/tab4.html), J. H. Hubbell & S. M. Seltzer | μ/ρ and μ<sub>en</sub>/ρ for 48 compounds and mixtures |
| [Zelenyy/nist-calculators](https://github.com/Zelenyy/nist-calculators/tree/master/xcom/data) | Machine-readable copy of the XCOM tables |

The tables are stored inside the HTML as base64-encoded gzip data and decoded in the browser with [pako](https://github.com/nodeca/pako).

---

## Physics model

- **Interpolation:** log–log linear on the XCOM energy grid, which NIST makes denser around absorption edges. Pair production is set to zero below its thresholds (1.022 MeV for the nuclear field, 2.044 MeV for the electron field).
- **Compounds and mixtures:** additivity rule, (μ/ρ) = Σ w<sub>i</sub> (μ/ρ)<sub>i</sub>.
- **Attenuation:** narrow-beam Beer–Lambert law, I/I₀ = exp(−Σ μ<sub>i</sub> x<sub>i</sub> / cos θ), applied to infinite horizontal slabs.
- **Broad beam:** an approximate geometric-progression (GP) buildup factor B(E, x).
- **Detector response:** point-estimate conversion of the transmitted primary beam, using μ/ρ and μ<sub>en</sub>/ρ.

### Limitations

This is a teaching and engineering-estimate tool, **not** a certified shielding-design code.

- The buildup factor is an approximation within the app. It isn't a tabulated ANSI/ANS-6.4.3 value or a Monte Carlo result.
- Secondary radiation (fluorescence, Compton-scattered photons, 511 keV annihilation photons) is reported as an indicator only. It isn't transported.
- Detector response doesn't model pulse-height spectra, energy resolution or dead time.
- Mixture presets use representative compositions.
- The angular-divergence setting in Shield 3D is shown in the readouts but isn't yet applied to ray generation.

For regulatory or safety-critical work, check results against authoritative data and a validated transport code such as MCNP, Geant4 or PHITS.

---

## Project structure

`xcom.html` is self-contained and organized in this order:

```
1. <head>   metadata, CDN libraries (Plotly 2.27, MathJax 3.2, pako 2.1), stylesheet
2. <body>   markup for the eight views
3. Data     NIST XCOM and SRD 126 Table 4 tables (gzip + base64)
4. Engine   window.XCOM: interpolation and element/compound/mixture attenuation
5. UI       one IIFE per view; each owns its state and wires its own controls
```

The main engine API:

```js
XCOM.elementAttenuation("Pb", 1e6);          // energy in eV → { energy, coherent, incoherent, photoelectric, pairAtom, pairElectron, totalWithoutCoherent, total, muen } in cm²/g
XCOM.compoundAttenuation("H2O", 662e3);
XCOM.mixtureAttenuation([{ formula: "H2O", weight: 0.965 }, { formula: "NaCl", weight: 0.035 }], 1e6);
XCOM.parseFormula("CaWO4");                  // → [["Ca", 1], ["W", 1], ["O", 4]]
XCOM.ENERGY_MIN; XCOM.ENERGY_MAX;            // 1e3 eV, 1e11 eV
```

---

## Contributing

Issues and pull requests are welcome. Before submitting a change:

1. Open the page and switch through all eight views. The browser console should show no errors.
2. If you change physics, compare a few values with the NIST XCOM web calculator. For example, Pb at 1 MeV should give μ/ρ ≈ 0.0710 cm²/g and μ<sub>en</sub>/ρ ≈ 0.0365 cm²/g.
3. Keep the app in a single file with no build step.

---

## License

Copyright (C) 2026 [Your Name]

This program is free software: you can redistribute it and/or modify it under the terms of the
**GNU General Public License** as published by the Free Software Foundation, either version 3 of the
License, or (at your option) any later version. See [`LICENSE`](LICENSE) for the full text.

This program is distributed in the hope that it will be useful, but **without any warranty**; without
even the implied warranty of merchantability or fitness for a particular purpose.

SPDX-License-Identifier: `GPL-3.0-or-later`

**Third-party components** keep their own licenses: Plotly.js (MIT), MathJax (Apache-2.0) and pako
(MIT and Zlib), all loaded from a CDN at runtime. NIST data are provided by the U.S. National Institute
of Standards and Technology; see NIST's
[terms of use](https://www.nist.gov/open/copyright-fair-use-and-licensing-statements-srd-data-software-and-technical-series-publications).

## Citation

If you use this tool in published work, please cite the underlying data:

> Berger, M. J., Hubbell, J. H., Seltzer, S. M., et al. *XCOM: Photon Cross Sections Database*, NIST Standard Reference Database 8 (XGAM). https://doi.org/10.18434/T48G6X
>
> Hubbell, J. H., & Seltzer, S. M. *Tables of X-Ray Mass Attenuation Coefficients and Mass Energy-Absorption Coefficients*, NIST Standard Reference Database 126. https://doi.org/10.18434/T4D01F
