# Phase 1: Benchmark Test Cases — OsdagBridge Software Testing
# For: Himanshu Soni | FOSSEE Winter Internship 2026
# Task: OsdagBridge Software Testing & Issue Reporting (Task 6)

## How to use this file
Each test case has exact inputs. Launch OsdagBridge, enter these values,
run the design, and capture evidence (screenshots, PDF report, tracebacks).
Mark Pass/Fail and note any mismatches.

## Before starting
Launch the app:
  source ~/miniconda3/etc/profile.d/conda.sh && conda activate osdagbridge-env
  cd /home/hsce/OsdagBridge/src && python -m osdagbridge.desktop

---

## BENCHMARK CASES (8 cases — verify correctness against expected behavior)

### Case B1: Default Design (Baseline)
**Purpose:** Establish baseline. The default template should produce a valid design.
**Inputs:** Don't change anything. Just click Design with whatever defaults load.
**Expected:** Design completes. 3D CAD renders. Report generates. No errors.
**Capture:** Screenshot of output dock, 3D CAD, generated PDF report.
**Pass/Fail:** ___

### Case B2: Minimum Span (20m)
**Purpose:** Verify boundary behavior at the minimum software limit.
**Inputs:**
  - Span: 20 m
  - Carriageway Width: 7.5 m
  - All other inputs: leave as default
**Expected:** Design completes. Check that no "span must be between" error appears.
**Capture:** Screenshot of inputs, output dock, 3D CAD, PDF report.
**Pass/Fail:** ___

### Case B3: Maximum Span (45m)
**Purpose:** Verify boundary behavior at the maximum software limit.
**Inputs:**
  - Span: 45 m
  - Carriageway Width: 7.5 m
  - All other inputs: leave as default
**Expected:** Design completes. Check especially that the design doesn't hang
  (existing issue #133 reports >10 min hang at 45m with median + footpaths).
**Capture:** Screenshot of inputs, output dock, 3D CAD, PDF report.
**Note:** Time how long the design takes.
**Pass/Fail:** ___

### Case B4: Custom Steel Material Properties
**Purpose:** Verify custom material inputs are preserved and used in calculations.
**Inputs:**
  - Span: 30 m
  - Carriageway Width: 7.5 m
  - Open Additional Inputs → Material Properties tab
  - For Girder: set Grade to "Custom"
    - Yield Strength Fy: 350 MPa
    - Ultimate Strength Fu: 490 MPa
    - Modulus of Elasticity E: 200 GPa
    - Modulus of Rigidity G: 77 GPa
    - Poisson's Ratio: 0.3
    - Density: 78.5 kN/m³
    - Thermal Coefficient: 12 (×10⁻⁶/°C)
  - Click Save, then Design
**Expected:** Design completes. Check the PDF report: do the custom values appear?
  Check the input dictionary (if accessible): are custom values stored?
  Existing issues #20, #41, #43 report custom values being ignored.
**Capture:** Screenshots of Material Properties dialog (before save), output dock, PDF report.
**Pass/Fail:** ___

### Case B5: Custom Concrete Deck Properties
**Purpose:** Verify custom concrete/deck material inputs are preserved.
**Inputs:**
  - Span: 30 m
  - Carriageway Width: 7.5 m
  - Open Additional Inputs → Material Properties tab
  - For Deck: set Grade to "Custom"
    - Density: 24 kN/m³
    - fck: 40 MPa
    - fctm: 3.5 MPa
    - Ecm: 33 GPa
    - Thermal Coefficient: 10 (×10⁻⁶/°C)
  - Click Save, then Design
**Expected:** Design completes. Custom deck values should be used.
  Existing issue #41 reports custom deck density never reaches create_deck_load.
**Capture:** Screenshots of Material Properties dialog, output dock, PDF report.
**Pass/Fail:** ___

### Case B6: Multi-Run State Persistence (Span Change)
**Purpose:** Verify the app doesn't hold stale state between consecutive runs.
  This is the #1 bug class the task explicitly asks for.
**Inputs:**
  - Run 1: Span = 25 m, Carriageway = 7.5 m. Click Design. Save the PDF report.
  - Unlock the design.
  - Run 2: Change Span to 35 m. Click Design. Save the PDF report.
  - Unlock the design.
  - Run 3: Change Span back to 25 m. Click Design. Save the PDF report.
**Expected:** Run 3's PDF report must be IDENTICAL to Run 1's PDF report.
  If any value differs, state contamination exists.
  Existing issues: #125 (stale girder segment_table), #111 (Additional Inputs
  save changes design even without edits), #85 (3D CAD not displayed after
  unlock and redesign).
**Capture:** All 3 PDF reports, screenshots of output dock for each run.
**Pass/Fail:** ___

### Case B7: Triangulation — UI vs Report vs Data Dict
**Purpose:** Verify the same value appears consistently across UI, PDF report,
  and internal data dictionaries. This is the bug class the task's own
  example describes.
**Inputs:**
  - Span: 30 m
  - Carriageway Width: 7.5 m
  - Skew Angle: 10°
  - Click Design
**Expected:**
  - The span (30m) should appear identically in: UI input field, PDF report,
    and any saved .osi file.
  - The overall bridge width should be consistent across: 2D CAD overlay text,
    3D CAD overlay, Results Table, and saved .osi dictionary.
  - Existing issue #124: Overall Bridge Width shows 25.45m (CAD), 26.65m
    (Results Table), 26.85m (.osi) — three different values!
**Capture:** Screenshots of 2D CAD (with measurements), Results Table, 3D CAD
  overlay text, saved .osi file contents. Note every mismatch.
**Pass/Fail:** ___

### Case B8: Round-Trip Serialization
**Purpose:** Verify save → reload → rerun produces identical results.
**Inputs:**
  - Span: 28 m, Carriageway Width: 7.5 m, Skew Angle: 5°
  - Click Design. Save the project as an .osi file.
  - Close OsdagBridge. Reopen it.
  - Load the .osi file. Click Design again.
**Expected:** The reloaded design must produce the exact same PDF report
  as the original run.
  Existing issues: #131 (yaml.dump vs safe_load mismatch), #71 (Save Input
  overwrites deck/wearing-course thickness with defaults), #96 (loading .osi
  skips custom material validation).
**Capture:** Both PDF reports (original and reloaded). Screenshots of any
  differences in the UI after reload.
**Pass/Fail:** ___

---

## HOW TO CAPTURE EVIDENCE

For each test case, create a folder:
  /home/hsce/osdagbridge-screening-task/test-cases/B{1-8}/

Put in that folder:
  - inputs.png (screenshot of inputs before Design)
  - outputs.png (screenshot of output dock after Design)
  - cad.png (screenshot of 3D CAD view)
  - report.pdf (the generated PDF design report)
  - notes.txt (any observations, error messages, timing)

For multi-run cases (B6), create subfolders: B6/run1/, B6/run2/, B6/run3/

---

## AFTER TESTING
Send me all screenshots and PDFs. I will:
1. Compare values across UI, report, and CAD for each case
2. Identify every mismatch as a potential bug
3. Trace root causes in the source code
4. File GitHub issues via Antigravity
5. Build the findings table for the report