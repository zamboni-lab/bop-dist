# bop!

**`bop`** (`BOP! v1.1.7`) is a standalone tandem mass spectrometry CLI combining five top-level commands:

- **`annotate`**: Takes a precursor compound (SMILES, 2D/3D `.mol`/`.sdf`, 3D `.xyz`, batch `.smi`/`.txt`/`.tsv`/`.csv`/`.sdf` file, or a licensed closed-world `PubChem` lookup via `--formula` or `--precursor-mz`) together with observed MS/MS fragment `m/z` values, and outputs ranked candidate fragment SMILES structures, elemental formulas, mass errors, and confidence scores.
- **`predict`**: Generates MS/MS spectra for CID (and licensed alternative activation methods) across user-specified collision energies (`-e`/`--energy`) from a compound, batch file, or closed-world **`PubChem`** isomers supplied by the licensing server (`--formula` or `--precursor-mz`).
- **`--depict`** (`-d`, available on both `annotate` and `predict`): Renders tight, vertically elongated **portrait fragmentation trees** to `.svg`, `.png`, `.jpg`/`.jpeg`, `.pdf`, `.html`, `.dot`, or `.mmd` (emitting `<stem>_0001.<ext>`, etc. for batch/isomer sets).
- **`activate`**, **`deactivate`** (alias `reset`), **`status`**: Claims a one-time license token (`<host:port>/<64-hex-secret>`) into `bop.lic`, inspects active licensing status, and removes local license files.

**Copyright & License**: (c) Zamboni Lab, ETH Zurich. See [`LICENSE`](LICENSE).

---

## Binaries

| File | Platform | Status |
| :--- | :--- | :--- |
| `bop.exe` | Windows x86-64 | Self-contained licensed CLI (`v1.1.7`: `annotate`, `predict`, `activate`, `deactivate`, `status`, `--depict`) |
| `bop` | Linux x86-64 / WSL2 | Self-contained licensed CLI (`v1.1.7`: `annotate`, `predict`, `activate`, `deactivate`, `status`, `--depict`) |
| `bop-macos-arm` | macOS Apple Silicon | Self-contained licensed CLI (unified CLI; pending `v1.1.7` rebuild) |

`bop.exe`, `bop`, and `bop-macos-arm` are single self-contained executables requiring no external data or model files.

---

## Licensing & Active Mode Log

Every run prints its active licensing state to `stderr` (`[bop] License: ...`):

- **Unlicensed (Public Free Mode, no `bop.lic`)**:
  - Allows **`annotate` only** for **1 compound at a time** (by SMILES or single-compound `.mol`/`.sdf`/`.xyz` file, default CID mode, `--geom 3d`), with a fixed **60-second delay**.
  - Fails closed (exit code `77`) on `predict`, batch inputs (`--input`, multi-record `.sdf`), `--formula`, `--precursor-mz`, `.msp.zip`, `.lib3`, `--msn`, non-`cid` `--method`, and `--2d`/`--geom 2d`.
- **Licensed (`bop.lic` via `bop activate`)**:
  - Unlocks `predict`, large-scale batch `annotate` and `predict` with no 60-second delay, closed-world `PubChem` isomer retrieval (`--formula`, `--precursor-mz`), NIST ZIP64 `.msp.zip` and `masster`-compatible `SQLite3` `.lib3` spectral library export, `--depict`, all activation methods (`cid`, `hcd`, `it-cid`, `ead`, `exd`, `eid`, `ei`, `ecd`, `etd`, `uvpd`), and multi-stage `--msn` pipelines according to server-enforced quotas and policy limits.

---

## Quick Start

### 1. Public Free `annotate` (1 compound, 60-second delay)

```powershell
# Windows (PowerShell / CMD) - Citrulline with included peak list + portrait SVG tree:
.\bop.exe annotate -c "NC(=O)NCCCC(N)C(=O)O" -f examples\citrulline_fragments.txt -d citrulline.svg

# Linux / WSL2 (Bash):
chmod +x ./bop
./bop annotate -c "NC(=O)NCCCC(N)C(=O)O" -m "159.0764, 158.0924, 141.0659, 116.0706, 113.0709, 70.0651" -d citrulline.png
```

### 2. Activate, Check Status, or Deactivate/Reset a License

```powershell
.\bop.exe activate <host:port>/<64-hex-secret>
.\bop.exe status
.\bop.exe deactivate   # or: .\bop.exe reset
```

### 3. Licensed `predict`

```powershell
# Predict multi-energy CID spectra for a SMILES string -> .msp.zip + portrait tree depiction:
.\bop.exe predict -c "CC(=O)Oc1ccccc1C(=O)O" -e 15,25,40 -o aspirin.msp.zip -d aspirin_pred.svg

# Predict closed-world PubChem isomers for a formula -> SQLite3 .lib3 + depictions:
.\bop.exe predict --formula C9H8O4 -e 20,35 --include-frag-smiles -o c9h8o4.lib3 -d c9h8o4_pred.svg

# Predict from precursor m/z:
.\bop.exe predict --precursor-mz 181.04954 --polarity pos -e 20 -o cand.lib3
```

### 4. Licensed Batch & Structure `annotate`

```powershell
# Batch SMILES file -> .lib3 spectral library + numbered portrait depictions:
.\bop.exe annotate --input compounds.smi -m "163.0390, 121.0284, 93.0335" -o results.lib3 -d trees.svg

# Closed-world PubChem annotation from formula:
.\bop.exe annotate --formula C9H8O4 -m "163.0390, 138.0311, 121.0284, 93.0335" -F json -o report.json
```

---

## Command-Line Reference

### Global Options & Top-Level Commands

```text
BOP! v1.1.7 - Zamboni Lab, ETH Zurich

Usage: bop.exe [OPTIONS] <COMMAND>

Commands:
  annotate    Annotate MS/MS fragment m/z values with ranked SMILES and formulas
  predict     Predict MS/MS spectra (requires active license)
  activate    Claim and save a one-time license token (<host:port>/<64-hex-secret>) into bop.lic
  deactivate  Remove local license certificate file(s) and return to Unlicensed Public Free Mode [alias: reset]
  status      Inspect license state, details, and daily quota usage
  help        Print this message or the help of the given subcommand(s)

Options:
      --license <LICENSE>          Explicit path to bop.lic
      --license-server <ENDPOINT>  Licensing server endpoint override (<host>:<port> or http://...)
      --threads <THREADS>          Optional cap on Rayon worker threads
  -h, --help                       Print help
  -V, --version                    Print version
```

### License Management Options

- `bop activate <TOKEN> [-o, --out <OUT>]`: Save the claimed certificate to a custom path (defaults to `--license` or `./bop.lic`).
- `bop deactivate [-f, --file <FILE>]` (alias `bop reset`): Remove the specified or discovered local certificate file.
- `bop status`: Print active mode, certificate metadata, daily query usage and remaining quota, servers, and enabled features.

### `bop annotate` Key Options

```text
  -c, --compound <COMPOUND>              SMILES or structure file (.mol, .sdf, .xyz) [aliases: --smiles, --mol]
      --input <INPUT>                    Batch file (.smi, .txt, .tsv, .csv, .mol, .sdf, .xyz; licensed)
      --formula <FORMULA>                Precursor formula for closed-world PubChem lookup via server (licensed)
      --precursor-mz <PRECURSOR_MZ>      Precursor m/z for closed-world PubChem lookup via server (licensed)
  -m, --mz <MZ>...                       Fragment m/z values or peak list file [aliases: --fragments, --peaks]
  -f, --mz-file <MZ_FILE>                Peak list file (.txt, .csv, .tsv, .mgf, .msp)
  -a, --adduct <ADDUCT>                  Precursor adduct(s) (e.g. "[M+H]+", "[M-H]-", "[M+Na]+")
      --polarity <POLARITY>              Ion polarity when --adduct is omitted (pos, neg, both, ei)
      --formula-scope <FORMULA_SCOPE>    Elemental formula scope for --precursor-mz [default: common]
      --max-inferred-formulas <COUNT>    Max candidate formulas per --precursor-mz query [default: 5]
      --max-isomers <COUNT>              Max PubChem isomers retrieved per query [default: 500]
      --limit <LIMIT>                    Optional cap on total compounds processed
      --geom <GEOM>                      auto, 2d, 3d [default: 3d; unlicensed: 3d only]
      --2d / --3d                        Force 2D (licensed) or 3D input geometry
      --method <METHOD>                  cid, hcd, it-cid, ead, exd, eid, ei, ecd, etd, uvpd [default: cid]
  -e, --energies <ENERGIES>              Collision energies (eV / V / nm) [default: 14,18,24]
      --msn <MSN>                        Multi-stage pipeline (e.g. "CID 25 @ 159.08 -> CID 30"; licensed)
  -t, --tol <TOL>                        Mass matching tolerance in Daltons [default: 0.05]
  -k, --top-k <TOP_K>                    Max candidate structures per fragment m/z [default: 3]
  -F, --format <FORMAT>                  default, table, smiles, csv, tsv, json, msp, msp.zip, lib3, svg, png, jpg, jpeg, pdf, html, dot, mermaid
  -o, --out <OUT>                        Output path (auto-detects extension)
  -d, --depict <DEPICT>                  Portrait tree image/document path (aliases: --tree, --image, --plot)
      --include-frag-smiles              Include fragment SMILES in .msp, .msp.zip, or .lib3 exports
```

### `bop predict` Key Options

```text
  -c, --compound <COMPOUND>              SMILES or structure file (.mol, .sdf, .xyz)
      --input <INPUT>                    Batch compound file (.smi, .txt, .tsv, .csv, .mol, .sdf, .xyz)
      --formula <FORMULA>                Precursor formula for closed-world PubChem lookup via server
      --precursor-mz <PRECURSOR_MZ>      Precursor m/z for closed-world PubChem lookup via server
  -e, --energy <ENERGY>                  Comma-separated CID collision energies (V / eV) [default: 20]
  -a, --adduct <ADDUCT>                  Precursor adduct(s)
      --polarity <POLARITY>              pos, neg, both, ei [default: pos]
      --formula-scope <FORMULA_SCOPE>    Elemental formula scope for --precursor-mz [default: common]
      --max-inferred-formulas <COUNT>    Max candidate formulas per --precursor-mz query [default: 5]
      --max-isomers <COUNT>              Max PubChem isomers retrieved per query [default: 500]
      --limit <LIMIT>                    Optional cap on total compounds processed
      --geom <GEOM>                      auto, 2d, 3d [default: 3d]
      --2d / --3d                        Force 2D or 3D input geometry
      --method <METHOD>                  cid, hcd, it-cid, ead, exd, eid, ei, ecd, etd, uvpd [default: cid]
      --no-isotopes                      Disable M+1 / M+2 natural isotope envelopes
      --include-frag-smiles              Include fragment SMILES annotations in exports and depictions
  -F, --format <FORMAT>                  default, table, msp, msp.zip, lib3, mgf, csv, json, svg, png, jpg, jpeg, pdf, html, dot, mermaid
  -o, --out <OUT>                        Output path (.msp.zip, .lib3, .msp, .mgf, .csv, .json, depiction)
  -d, --depict <DEPICT>                  Portrait tree image/document path
```

---

## Included Example Files (`examples/`)

- **2D `.mol` (`examples/aspirin_2d.mol`)**:
  ```powershell
  .\bop.exe annotate -c examples\aspirin_2d.mol -m "163.0390, 139.0390, 121.0284, 93.0335, 43.0178" -d aspirin.svg
  ```
- **3D `.xyz` (`examples/caffeine_3d.xyz`)**:
  ```powershell
  .\bop.exe annotate -c examples\caffeine_3d.xyz -m "195.0877, 138.0662, 110.0713, 83.0604, 69.0447" -k 1
  ```
- **Negative ion mode (`--adduct "[M-H]-"`)**:
  ```powershell
  .\bop.exe annotate -c "NC(CCCC(=O)O)C(=O)O" --adduct "[M-H]-" -m "142.0504, 116.0712, 98.0606"
  ```
- **Sodiated `[M+Na]+` precursors**:
  ```powershell
  .\bop.exe annotate -c "OCC1OC(O)C(O)C(O)C1O" --adduct "[M+Na]+" -m "203.0526, 185.0420, 143.0315, 83.0104"
  ```
- **Portrait tree formats (`.svg`, `.png`, `.jpg`, `.pdf`, `.html`, `.dot`, `.mmd`)**:
  ```powershell
  .\bop.exe annotate -c "CC(=O)Oc1ccccc1C(=O)O" -m "163.0390, 139.0390, 121.0284, 93.0335" -d aspirin_tree.png
  ```
  PDF output uses Python with `matplotlib`/RDKit/Pillow when available; SVG, PNG, JPEG, HTML, DOT, and Mermaid render built-in with zero external dependencies.
