# ADMIXTOOLS qpAdm Analysis

Perform ancient DNA ancestry modeling using qpAdm from the ADMIXTOOLS suite. This skill covers data preparation, model design, execution, validation, and interpretation for population genetics research.

## Core Concepts

**qpAdm** tests whether a target individual/population can be modeled as a mixture of source populations, using outgroup populations to constrain the model.

**Key Components**:
- **Target**: Individual or population to model
- **Left (Sources)**: Candidate ancestral populations (k=2-4 typically)
- **Right (Outgroups)**: Diverse populations constraining the model (8+ recommended)
- **Output**: Mixture coefficients, standard errors, p-value for model fit

---

## Data Preparation Pipeline

### 1. Convert 23andMe to PLINK

```bash
# Raw conversion
plink --23file genome_raw.txt FAM001 IND01 --make-bed --out target_raw
# Output: ~631K variants (v5 chip)

# Filter to autosomes
plink --bfile target_raw --chr 1-22 --make-bed --out target_auto
# Output: ~608K variants

# SNP quality filtering
plink --bfile target_auto --snps-only just-acgt --biallelic-only strict --make-bed --out target_biallelic
# Output: ~603K variants
```

### 2. Remove Strand-Ambiguous SNPs

A/T and C/G SNPs cannot be reliably strand-aligned:

```bash
# Identify ambiguous SNPs
awk 'BEGIN{FS="\t"} {
  a=$5; b=$6;
  if ((a=="A"&&b=="T")||(a=="T"&&b=="A")||(a=="C"&&b=="G")||(a=="G"&&b=="C"))
    print $2
}' target.bim > ambiguous_snps.txt

# Exclude them
plink --bfile target_biallelic --exclude ambiguous_snps.txt --make-bed --out target_clean
# Typically removes ~500-600 SNPs
```

### 3. Convert to EIGENSTRAT Format

Create `convertf.par`:
```
genotypename: target_clean.bed
snpname: target_clean.bim
indivname: target_clean.fam
outputformat: EIGENSTRAT
genotypeoutname: target.geno
snpoutname: target.snp
indivoutname: target.ind
```

Run conversion:
```bash
convertf -p convertf.par
```

**Fix .ind file** (may show "???" for population):
```bash
echo "IND01 M TARGET" > target.ind
```

### 4. Merge with Reference (AADR)

Create `mergeit.par`:
```
geno1: AADR_1240k_public.geno
geno2: target.geno
snp1: AADR_1240k_public.snp
snp2: target.snp
ind1: AADR_1240k_public.ind
ind2: target.ind
genooutfilename: merged.geno
snpoutfilename: merged.snp
indoutfilename: merged.ind
docheck: YES
hashcheck: YES
```

Run merge:
```bash
mergeit -p mergeit.par
```

**Expected overlap**: 100K-200K SNPs between 23andMe and AADR 1240K panel

---

## Reference Data: AADR

**Allen Ancient DNA Resource (AADR)** - curated compendium of published ancient genomes.

**Panel Options**:
| Panel | SNPs | Best For |
|-------|------|----------|
| 1240K | ~1.23M | Ancient DNA (maximizes aDNA coverage) |
| HO | ~600K | Modern populations, smaller datasets |

**Download**: https://reich.hms.harvard.edu/allen-ancient-dna-resource-aadr-downloadable-genotypes-present-day-and-ancient-dna-data

---

## qpAdm Parameter File

Create `qpadm.par`:
```
genotypename: /path/to/merged.geno
snpname: /path/to/merged.snp
indivname: /path/to/merged.ind
popleft: left.txt
popright: right.txt
allsnps: YES
details: YES
inbreed: NO
```

**Critical Parameters**:
- `allsnps: YES` - Use all available SNPs (not intersection across all populations)
- `details: YES` - Print detailed output with coefficients
- `inbreed: NO` - **CRITICAL** for single-individual populations

### Left File (Target + Sources)

```
# left.txt
TARGET
India_Harappan.AG
Russia_MLBA_Sintashta.AG
India_GreatAndaman_100BP.SG
```

First line is always the target. Remaining lines are source populations.

### Right File (Outgroups)

```
# right.txt - Diverse global anchors
Mbuti.DG
Russia_UstIshim_IUP.DG
Russia_Kostenki14_UP.SG
Russia_MA1_UP.SG
Han.DG
Papuan.DG
Karitiana.DG
Ethiopia_4500BP.SG
```

**Outgroup Selection Principles**:
- Include deep splits (Mbuti, Papuan) for basal diversity
- Include structured populations (WHG, CHG, EHG) to disambiguate West Eurasian components
- 8+ outgroups recommended; 10-12 for complex models
- Avoid populations that could be admixed with sources

---

## Execution

```bash
# Activate environment
conda activate qpadm_env

# Run qpAdm
qpAdm -p qpadm.par > qpadm_output.txt
```

---

## Output Interpretation

### Key Output Lines

```
# Best coefficients (mixture proportions)
best coefficients:     0.534     0.173     0.293

# Standard errors
std. errors:     6.244     3.377     2.913

# Summary line with p-value
summ: TARGET    3      0.373182     0.534     0.173     0.293
#                      ^^^^^^^^
#                      p-value
```

### Nested Model Tests

```
    fixed pat  wt  dof     chisq       tail prob
          000  0     5     5.363        0.373182    # Full model
          001  1     6     7.546        0.273261    # No source 3
          010  1     6     5.455        0.354065    # No source 2
          100  1     6     8.123        0.228445    # No source 1
```

- `000` = full model with all sources
- `001` = model without third source (tests if source 3 is necessary)
- If nested model also passes (p ≥ 0.05), that source may not be required

---

## Model Quality Criteria

### Pass Criteria

| Criterion | Threshold | Meaning |
|-----------|-----------|---------|
| p-value | ≥ 0.05 (conservative), ≥ 0.01 (exploratory) | Model not rejected |
| Coefficients | Within [-0.02, 1.02] | Feasible proportions |
| Coefficient sum | Within [0.97, 1.03] | Proportions sum to ~100% |

### Failure Modes

| Issue | Symptom | Cause |
|-------|---------|-------|
| Infeasible | Negative coefficients | Sources overlap in drift space |
| Rejected | p < 0.01 | Model doesn't fit the data |
| Overfitting | Coefficients > 100% | Sources not independent |

---

## qpWave: Determine Number of Streams

Run qpWave **before** qpAdm to determine how many ancestral streams are needed:

```
f4rank: 0 dof: 24 chisq: 2129.943 tail: 0       # Rank 0: REJECTED
f4rank: 1 dof: 14 chisq: 66.847   tail: 7.1e-9  # Rank 1: REJECTED
f4rank: 2 dof: 6  chisq: 2.038    tail: 0.916   # Rank 2: ACCEPTED
```

**Interpretation**: Rank 2 acceptance (p=0.92) means 3 independent ancestry streams are sufficient.

---

## F4-Statistics Validation

Use qpDstat to directly test ancestry hypotheses:

```bash
qpDstat -p qpdstat.par
```

**Test Pattern**: `f4(Outgroup, Target; Pop_A, Pop_B)`
- Negative Z-score (< -3): Target closer to Pop_B
- Positive Z-score (> 3): Target closer to Pop_A
- |Z| < 3: No significant difference

**Example Tests**:
```
f4(Mbuti, TARGET; IVC, Steppe)     Z=0.06   # No preference
f4(Mbuti, TARGET; Steppe, Iran_N)  Z=-3.9   # Closer to Iran_N
f4(Mbuti, TARGET; Andaman, Papuan) Z=-4.3   # Closer to Papuan (flag!)
```

---

## Robustness Analysis

### Outgroup Sensitivity

Test multiple outgroup configurations:

```python
base_right = ['Mbuti.DG', 'Han.DG', 'Papuan.DG', 'Karitiana.DG']
structure_pops = ['Luxembourg_Mesolithic', 'Georgia_Kotias', 'Israel_Natufian']

# Test expanded outgroup sets
for combo in combinations(structure_pops, 2):
    right_set = base_right + list(combo)
    run_qpadm(left, right_set)
```

### Proxy Sensitivity

Swap equivalent proxies:
- IVC: Rakhigarhi ↔ Shahr-i-Sokhta
- AASI: Great Andamanese ↔ Irula
- Steppe: Sintashta ↔ Yamnaya

### Panel Comparison

Compare 1240K vs HO panels on common SNP set:

```bash
# Extract common SNPs
comm -12 <(cut -f2 aadr_1240k.snp | sort) <(cut -f2 aadr_ho.snp | sort) > common_snps.txt
```

---

## Component Robustness Metrics

```python
def compute_robustness(component_values: List[float]) -> str:
    """Grade component stability across model configurations."""
    median = statistics.median(component_values)
    stdev = statistics.stdev(component_values)

    # Coefficient of variation
    cv = stdev / max(median, 0.01)

    if cv < 0.3:
        return 'High'      # Stable across configurations
    elif cv < 0.6:
        return 'Medium'    # Some variation
    else:
        return 'Low'       # Highly sensitive to model choice
```

---

## Steppe Certainty Protocol

For unstable components (e.g., Steppe ancestry), apply rigorous testing:

### Phase 0: Standardize Inputs
- Create common SNP sets for fair comparison
- Fix population ontologies

### Phase 1: Build Identifiable Outgroup Set
- Add West Eurasian structure (WHG, CHG, EHG)
- Prevents Steppe from absorbing Iran/Farmer drift

### Phase 2: Necessity Testing
```python
def test_necessity(target, sources, right):
    """Test if each source is necessary."""
    full_result = run_qpadm(target, sources, right)

    for i, source in enumerate(sources):
        reduced_sources = sources[:i] + sources[i+1:]
        reduced_result = run_qpadm(target, reduced_sources, right)

        # Source necessary if:
        # - Full model passes (p ≥ 0.05)
        # - Reduced model fails (p < 0.05)
        necessary = (full_result.p >= 0.05 and reduced_result.p < 0.05)
        print(f"{source}: {'NECESSARY' if necessary else 'NOT REQUIRED'}")
```

### Phase 3: Outgroup Rotation
- Sample 100-1000 reasonable Right set combinations
- Compute median, IQR, range for each component
- Report uncertainty intervals

---

## Positive Control Validation

Test known populations to validate model setup:

```python
controls = ['GIH', 'ITU', 'PJL']  # 1000 Genomes South Asian populations

for pop in controls:
    result = run_qpadm(pop, sources, right)
    print(f"{pop}: {result.coeffs} p={result.p}")

# Expected: Reference populations may show infeasible coefficients
# (more complex ancestry than simple 3-way model)
```

---

## Output Parsing (Python)

```python
import re
from pathlib import Path
from typing import Dict, List, Optional

def parse_qpadm_output(filepath: Path) -> Optional[Dict]:
    """Parse qpAdm output file."""
    content = filepath.read_text()

    result = {
        'left': [],
        'right': [],
        'p_value': None,
        'coeffs': [],
        'std_errors': [],
    }

    # Parse p-value from summ line
    summ_match = re.search(r'summ:\s+\S+\s+\d+\s+([\d.e+-]+)', content)
    if summ_match:
        result['p_value'] = float(summ_match.group(1))

    # Parse coefficients
    coef_match = re.search(r'best coefficients:\s+(.*)', content)
    if coef_match:
        result['coeffs'] = [float(x) for x in coef_match.group(1).split()]

    # Parse standard errors
    se_match = re.search(r'std\. errors:\s+(.*)', content)
    if se_match:
        result['std_errors'] = [float(x) for x in se_match.group(1).split()]

    return result


def evaluate_model(result: Dict) -> List[str]:
    """Evaluate model quality."""
    flags = []

    # P-value check
    if result['p_value'] >= 0.01:
        flags.append('PASS_P_VALUE')
    else:
        flags.append('FAIL_P_VALUE')

    # Coefficient bounds
    all_valid = all(-0.02 <= c <= 1.02 for c in result['coeffs'])
    flags.append('PASS_COEF_BOUNDS' if all_valid else 'FAIL_COEF_BOUNDS')

    # Coefficient sum
    coef_sum = sum(result['coeffs'])
    flags.append('PASS_COEF_SUM' if abs(coef_sum - 1.0) <= 0.03 else 'WARN_COEF_SUM')

    # Overall
    if 'PASS_P_VALUE' in flags and 'PASS_COEF_BOUNDS' in flags:
        flags.append('PASS_BASIC')

    return flags
```

---

## Common Pitfalls & Solutions

### Inbreed Mode Error

**Error**: `fatalx: pop has sample size 1 and inbreed set`

**Solution**: Set `inbreed: NO` in parameter file

### Missing Population Labels

**Symptom**: .ind file shows "???"

**Solution**: Manually create .ind with correct labels:
```bash
echo "IND01 M TARGET" > target.ind
```

### Low SNP Overlap

**Symptom**: Only 10-20% SNP overlap with reference

**Cause**: Different array ascertainment (23andMe vs 1240K capture)

**Solutions**:
- Accept if >100K SNPs (generally sufficient)
- Consider HO panel (more array-compatible)
- Document limitation in results

### Negative Coefficients

**Symptom**: Model gives coefficients like 150%, -44%, -6%

**Causes**:
- Sources overlap in drift space
- Proxies don't represent components cleanly
- Insufficient outgroup structure

**Solutions**:
- Use more direct ancient samples
- Expand outgroup set
- Test simpler (2-way) models
- Check nested model tests

### Unstable Steppe Component

**Symptom**: Steppe changes 17% → 3% with different outgroups

**Solution**: Apply Steppe Certainty Protocol (see above)

---

## Interpretation Guidelines

### Language Constraints

**Do Say**:
- "Genetic affinity consistent with..."
- "Can be modeled as a mixture of..."
- "Proxy represents..."
- "53% ± 34% (95% CI: 0-87%)"

**Don't Say**:
- "Is X% from..."
- "Proves ancestry..."
- "Definitively shows..."

### Uncertainty Communication

Always report:
- Point estimate with standard error: "53% ± 6%"
- 95% CI: "coefficient ± 1.96 × SE"
- Range across configurations: "5-25% across model variants"
- Robustness grade: High/Medium/Low

### Required Caveats

1. Single-individual targets have larger standard errors than populations
2. All ancient sources are proxies; results reflect genetic affinities
3. Weakly identified components may not be strictly necessary (check nested tests)
4. Multiple plausible models may exist; results are model-dependent

---

## South Asian Model Example

### Three-Component Framework

| Component | Proxy Options | Expected Range |
|-----------|---------------|----------------|
| IVC-related | Rakhigarhi, Shahr-i-Sokhta | 40-70% |
| Steppe MLBA | Sintashta, Yamnaya | 5-25% |
| AASI | Great Andamanese, Irula | 20-50% |

### Recommended Outgroups

```
# Base diverse anchors
Mbuti.DG              # Sub-Saharan Africa
Han.DG                # East Asia
Papuan.DG             # Oceania
Karitiana.DG          # Americas

# Deep time anchors
Russia_UstIshim_IUP.DG
Russia_Kostenki14_UP.SG
Russia_MA1_UP.SG

# Regional structure (disambiguates West Eurasian)
Luxembourg_Mesolithic  # WHG
Georgia_Kotias         # CHG
Ethiopia_4500BP.SG     # Northeast Africa
```

---

## Environment Setup

```bash
# Create conda environment
conda create -n qpadm_env python=3.10
conda activate qpadm_env

# Install ADMIXTOOLS (via conda-forge)
conda install -c bioconda admixtools

# Or compile from source
git clone https://github.com/DReichLab/AdmixTools.git
cd AdmixTools/src
make all
```

---

## Workflow Summary

1. **Prepare Data**: QC → EIGENSTRAT → Merge with AADR
2. **Run qpWave**: Determine number of streams statistically
3. **Design Models**: Select sources and outgroups based on hypothesis
4. **Run qpAdm**: Test multiple model configurations
5. **Evaluate**: Check p-values, coefficients, nested tests
6. **Validate**: F4-statistics, positive controls
7. **Robustness**: Outgroup/proxy/panel rotation
8. **Interpret**: With uncertainty intervals and appropriate caveats
