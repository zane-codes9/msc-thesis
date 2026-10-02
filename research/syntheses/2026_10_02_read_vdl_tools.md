**2026-10-02_read_vdl-tools**

  -------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  **QUESTION**   What does the vDL already hold that CanVS can use, and what does it change
  **READ**       Gasparrini, `CTS_smallarea`; Éric, `Extract mortality 2010 to 2023`, `extract_estimates_RDC`, `CTS QUEBEC`; Lulu, `1) Data Management` and `2) Settings and Cross-Basis Specifications`; the 09-28 Éric meeting; DAD Vetting Handbook (contents); Stat/Transfer User Manual (contents); CVSD RDC User Guide §5.4; VMA Researcher Manual; RDC / FRDC Confidentiality Handbook for Economic Microdata; RDC Confidentiality Vetting Request Form. Thirteen tracker rows (SOFTWARE), thirty-four shelf rows
  **FINDING**    The 21-lag cross-basis needs no full DA $\times$ day grid; lags come from a date-offset join onto the rows that exist
  **MOVED**      lag route, from grid rebuild to one join; outcome ruling, from re-extraction to rebuild; covariance gate, from banned to unsettled; `mortality_cts`, from filtered object to probable full grid
  **SHELVED**    thirty-four rows: vDL and StatCan documentation, contracts and agreements, data sources, R packages, repositories, reference tables, the HC-10114 workspace tree
  -------------- ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Finding

The 21-lag cross-basis needs no full DA $\times$ day grid. Éric and Lulu
build lags the same way. Each row looks up its lag days in a complete
exposure table, matched on DA and `DATE - k`, and off-series dates
return NA and drop from the fit. Gasparrini does the opposite. He pads
the grid with zero-death days and shifts within each area. The two
routes give the same coefficients under the conditional likelihood,
because strata with no deaths carry no information; only the
quasi-Poisson dispersion differs. The filtered rows already inside the
vDL are therefore enough. What the 21-lag matrix needs is a complete
Daymet-by-DA table, and that table is already in `Transfer/Incoming`.

# Per source

**Gasparrini, `CTS_smallarea`.** The reference code for the small-area
case time series, London MSOAs over two summers. It completes the area
$\times$ day grid with zeros, links gridded temperature by area-weighted
extraction, and compares temporal against spatial variation. The model
files `03.mainmod` and `04.intmod` were not read.

**Éric, `Extract mortality 2010 to 2023`.** The SAS program behind the
project's mortality data. Per year it merges the health and geography
files on `registration_number`, flags ten causes from the ICD code,
drops the geography file's DA and relinks every postal code through one
PCCF vintage. It then builds a time-stratified case-crossover file and,
from its cases, a warm-season case time series.

**Éric, `extract_estimates_RDC`.** The SMOKE Study's estimate and
release script. It fits smoke and non-smoke PM2.5 cross-bases with
temperature and humidity adjusted, and writes every figure and table as
CSV. It never writes coefficients or their covariance. Its comments
carry the reasoning of each version, the memory arithmetic of a
cross-basis on tens of millions of rows, and the rule that rounding is
for reading, not disclosure.

**Éric, `CTS QUEBEC`.** Practice code on Québec counts by RSS grouped
into public health units. It runs the full two-stage chain: a gnm fit
per unit, `crossreduce`, `mixmeta` with and without meta-predictors, a
Wald test, BLUPs, the minimum-risk temperature per unit inside p25--p90,
recentring and a pooled curve. Four bugs keep it from running as
written.

**Lulu, `1) Data Management` and `2) Settings`.** Her pipeline for PM2.5
and heat. She reads `mortality_cts` from SAS once in about 2.5 hours and
saves every later step as its own object. She builds DA $\times$ year
$\times$ month $\times$ weekday strata, drops strata with no deaths, and
joins DA-level PM2.5 by date offset. Her settings note that the lag
window goes to 21 by rebuilding the lags in Step 1.

**The 09-28 Éric meeting.** Three facts. His VM and Lulu's have about
256 GB against 16 for students. Every year is relinked through one PCCF
vintage. `mortality_cts` is all-cause only.

**DAD Vetting Handbook.** The vetting rulebook for this file, version
2.0, May 2025. Only the contents were read. Models sit on p. 16, Graphs
on p. 18, Heat Maps on p. 19 and Residuals With Geography on p. 42.

**Stat/Transfer User Manual.** The SAS-to-R converter in Researcher
Resources. Only the contents were read. Selecting Cases, Selecting
Variables and SAS Reading are the sections that could cut the 2.5-hour
read.

**CVSD RDC User Guide §5.4.** One section. It names no vetting rule for
the linked data and sends the reader to the RDC Analyst and the vetting
training materials.

**VMA Researcher Manual.** How output leaves the vDL. Files are staged
in the project's Vetting Requests folder from the VM, and the request is
filed in the VMA from a browser outside it. Statuses run Draft,
Submitted, Under review, Changes requested, Approved or Not approved,
and released files land in the project's outgoing storage.

**RDC / FRDC Confidentiality Handbook for Economic Microdata.** Written
for business data. It releases final outputs only, forbids suppression
of cells and of map areas, and lists coefficient variance-covariance
matrices among model outputs that can never be released. It allows a
non-numerical summary of signs and significance before the final
release.

**RDC Confidentiality Vetting Request Form.** The general form, revised
May 2020. Its checklist asks how linked data were linked, whether a
covariance matrix is requested and with what supporting counts, and
whether the output is final.

# Across rows

1.  **Non-accidental counts are a rebuild, not a re-extraction.**
    `smoke.mortality_2010_2023` holds every death record with
    `death_cause_4digits`, and the `noext` flag already exists (001--799
    or A00--R99). *(`mortality_cts`, Éric's extraction)*

2.  **The minimum-risk temperature needs a search window.** Éric reads
    it from the BLUPs inside p25--p90. On 09-30 the hot tail turned down
    past the p90 knot and pulled the minimum out of band. The plan says
    p1--p99. *(Éric's Québec example, Lulu's Step 1 object)*

3.  **Row-indexing a basis does not transfer.** Éric's `subrows()`
    slices one national basis because his basis is data-independent, a
    linear exposure with fixed lag knots. Ours puts knots at each CMA's
    p10, p75 and p90. *(Éric's release script, Éric's Québec example)*

4.  **No map area may be dropped.** Area suppression is not permitted; a
    DA that cannot be released is collapsed or redesigned, never
    blanked. *(RDC handbook, vetting request form)*

5.  **Non-numerical check-ins are written policy.** Before the final
    release, signs and significance can be vetted in place of numbers.
    *(RDC handbook, VMA manual)*

6.  **`mortality_cts` is probably already a full grid.** Lulu applies
    the zero-event filter herself after reading it, and the SAS step
    that builds `WORK.TSDSTIMESERIESDATA`, where zero days would be
    padded, is missing from the code. *(`mortality_cts`, Éric's
    extraction, Lulu's pipeline)*

7.  **Temperature arrives with the deaths.** `T_MEAN` and its lags sit
    inside `mortality_cts`; Lulu adds only PM2.5, from CanOSSEM.
    *(`mortality_cts`, Lulu's pipeline, Lulu's Step 1 object)*

# My model

The 21-lag matrix for Stage 1 is one join of Daymet-by-DA at `DATE - k`,
`k` in `0:21`, onto the filtered rows. The full-grid builder leaves the
critical path.

The outcome ruling costs a rebuild from a file that already exists,
whichever way it goes.

The minimum-risk temperature is searched inside a stated window. The
window is a written choice in the spec.

Each CMA builds its own cross-basis on its own series.

The November map collapses any unreleasable DA into a releasable unit
and drops none. The release carries surfaces, never coefficients or
covariance, as one final request.

# Contradictions

1.  **Cause.** Éric says `mortality_cts` is all-cause only. Lulu's Step
    1 object carries cause-specific columns. Settled by `names(full)`
    and her unread `Heat Only - Data Management.R`.

2.  **Covariance.** The general request form asks only for supporting
    counts when a covariance matrix is requested. The economic handbook
    bans coefficient covariance outright. CVSD §5.4 names no rule.
    Settled by the DAD Vetting Handbook, pp. 16, 18 and 19.

3.  **Age bands.** Lulu's object has five, the blueprint has four.
    Settled by the band edges in her `1) Data Management.R`.

# Corrections

1.  Lulu was thought to have joined temperature at health-region level.
    `T_MEAN` and its lags arrive inside `mortality_cts`. *(Lulu's
    pipeline)*

2.  The column positions `14:21` and `22:29` were read as belonging to
    `full`. They belong to the sibling `Mortality Data_Step 1`. *(Lulu's
    Step 1 object)*

3.  The 6 GB file that froze the VM was taken to be the Daymet file.
    Which file it was is not recorded. *(DA Daymet series)*

4.  Éric's Québec example was read as a possible spec. It is practice
    code with four bugs: an undefined `mod`, NULL names from
    `cities$PHU`, an undefined `cpall`, and lag columns taken by
    position. *(Éric's Québec example)*

# Open questions

1.  Does `mortality_cts` pad zero-death days, or does it hold only days
    with a death?

2.  At what resolution is the `T_MEAN` inside `mortality_cts`, DA or
    health region?

3.  Which rule binds coefficient covariance for CVSD output, the general
    form's or the economic handbook's?

4.  Does the rule against multiple iterations of output bind health
    projects as it binds economic ones?

5.  Is a per-DA surface a map of model-predicted values in the
    handbook's sense, or an area-level summary outside that rule?

6.  What window should the minimum-risk temperature search use for
    CanVS, and on what ground?

# People

**Éric Lavigne.** Two registers. His production code is exacting: a
pre-flight that stops on any missing column, mass-conservation checks on
every partition, change logs that explain why each version broke and
what the fix touches, and the reminder to pre-specify primary outcomes
and state the test in Methods. His practice code is loose and fast,
written to learn a method, bugs left in. He extracts in SAS and models
in R. His release habit is the conservative one, curves and burden out,
coefficients and covariance in.

**Lulu.** Writes to be read by a learner. Nearly every line carries a
comment saying what it does, alternatives stay in the file commented
out, and she flags what she does not understand rather than guessing
(*has 98 more variables but unsure how these were calculated*). She
saves every step as its own object, which is why her work fits a 16 GB
machine. Her headers credit the lineage, *Modifies code that Eric
created*.

**Kafi.** The curator. Placed Gasparrini's tutorial in the shared
workspace verbatim, so the group works from the reference code, not a
copy of a copy.

**Antonio Gasparrini.** The design's author, and his code is as spare as
his papers. Capitalised comment headers, one idea per block,
`data.table` and the native pipe, and order exploited deliberately
(*important for keeping the time series sequence*). He keeps the code
public and current on GitHub and says so in the first lines.

**The DAD Vetting Committee.** Risk-averse by charter. *If these goals
conflict, and where judgement must be applied, reducing the risk of
disclosure takes precedence.* Its documents prefer redesign to
suppression, final output to iteration, and words to numbers when a
number is not needed.
