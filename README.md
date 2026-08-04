# Comparative Analysis of LLM-Generated Ferritic and Martensitic Steel Data

Four LLMs (Grok, Gemini, Claude, and ChatGPT) were given the same prompt to generate a synthetic dataset of Ferritic and Martensitic stainless steel properties. This project cleans each dataset, checks it against known metallurgical relationships, and compares the four models against each other and against real experimental literature data.

## Background

The goal was to build practical skills in materials informatics- specifically generating, cleaning, and analyzing materials data; while also testing something more
interesting: can an LLM generate synthetic materials data that actually behaves like real steel, or does it just produce numbers that look plausible in isolation?

## Prompt used

The same prompt was issued to all four models:

> "Generate a hypothetical dataset of 70 independent data points for Ferritic and  Martensitic Stainless Steels. Each point should include chemical composition (wt.% for elements like Fe, Cr, Mn, etc.), heat treatment details (e.g., annealing  temperature/time), mechanical testing temperature and relevant data, grain size (µm), phases present with volume fractions (%), yield strength (MPa), elastic modulus (GPa), ultimate tensile strength (MPa), hardness (value and scale), elongation (%), strain rate (s⁻¹). Introduce variability in compositions and properties based on realistic ranges for this alloy class. Include 15% missing values randomly, and add minor inconsistencies like occasional negative values or unit errors. Output as a CSV with headers."

This was designed to simulate real-world data problems on purpose- missing values, unit inconsistencies, and physical outliers- so the exercise wasn't just about generating numbers but about cleaning and critically evaluating them afterward.

## Contents

```
├── data/
│   ├── raw/       four unmodified LLM outputs (grok.csv, gemini.csv, claude.csv, chatgpt.csv)
│   └── cleaned/   cleaned versions, produced by running notebooks 1-4
├── notebooks/
│   ├── 01_grok_analysis.ipynb
│   ├── 02_gemini_analysis.ipynb
│   ├── 03_claude_analysis.ipynb
│   ├── 04_chatgpt_analysis.ipynb
│   └── 05_comparative_analysis.ipynb
└── requirements.txt
```

Each of the first four notebooks cleans one model's raw output and checks it against metallurgical trends on its own. The fifth notebook loads all four cleaned datasets and compares them side by side, plot by plot, and validates them against real literature data.

## Running it

Run notebooks 1 through 4 first, each one cleans its own dataset and saves the result into `data/cleaned/`. Then run notebook 5, which depends on all four cleaned files being present. Install the packages in `requirements.txt` first if needed.

## Cleaning methodology

Each dataset had its own quirks, so each was cleaned a little differently:

**Grok**- Column headers were standardized to match a common schema across all four datasets. Missing values were imputed with the column median to avoid skewing the
distribution. All numeric columns were strictly parsed as numbers, coercing any leftover text artifacts to missing before imputation.

**Gemini**- Physically impossible negative values (e.g. negative yield strength or temperature) were treated as missing rather than kept. Elastic modulus values given in the wrong order of magnitude (MPa instead of GPa) were corrected. Remaining gaps were filled with median imputation.

**ChatGPT**- Extreme outliers in mechanical properties were capped to realistic ranges before plotting, to avoid one bad point distorting the whole picture. Ferrite/martensite volume fractions were checked for consistency, and where one phase was missing it was calculated from the balance of the other. Headers were standardized and any remaining text characters in numeric fields were cleaned out.

**Claude**- A "Phase Austenite" column that was entirely zero was removed, since it doesn't apply to a Ferritic-Martensitic system. An explicit "Alloy Type" column was
missing from the raw data, so it was engineered from the first letter of the sample ID (e.g. F001 → Ferritic, M005 → Martensitic). Hardness values given on mixed Rockwell scales (HRC/HRB) were converted to a single Vickers (HV) scale so they could be compared directly against the other datasets.

## Analysis approach

Basic statistics (mean, median, dispersion) were computed first as a baseline, grouped by alloy type to check whether each model correctly distinguished Ferritic from
Martensitic properties. From there, a series of plots tested whether each model followed real metallurgical rules rather than just generating random numbers within a range:

- **Correlation matrix**- a first sanity check on whether properties moved together the way real steel properties should (e.g. hardness rising with carbon content).
- **Yield strength vs. grain size**- tests the Hall-Petch relationship (finer grains → higher strength).
- **Yield strength vs. elongation**- tests the standard strength-ductility trade-off.
- **Hardness vs. carbon content**- tests solid-solution strengthening.
- **UTS vs. hardness**- both measure resistance to deformation, so they should track linearly; this checks internal consistency within each dataset.
- **Yield strength vs. ferrite fraction**- strength should drop as the softer ferrite phase fraction increases.
- **Yield strength vs. heat treatment temperature**- tests whether the model captured the causal link between processing conditions and final mechanical properties.
- **Violin plots of strength by alloy type**- checks whether the value ranges are realistic and clearly separate the two alloy classes.

## Comparison with literature data

The four AI-generated datasets were also checked against real experimental data aggregated from 13 peer-reviewed papers on Ferritic and Martensitic stainless steels. The literature values ranged from 310 to 1265 MPa for yield strength, and 2.7% to 26% for elongation.

Claude and Gemini stayed closest to these ranges, correctly assigning higher strength to martensitic grades. Grok deviated the most, including elongation values above 40%, well outside the literature limits for this alloy class.

The literature data also showed a clear processing-property relationship: annealed samples were consistently softer and more ductile than quenched samples. Claude was the only model to reproduce this clearly, with higher heat treatment temperatures correlating with reduced strength. ChatGPT and Grok showed no clear relationship between heat treatment and final mechanical properties.

One more difference worth noting: the real literature dataset had large, uneven gaps (hardness alone was missing in roughly 58% of data points), while all four AI-generated datasets were much cleaner, containing only the ~15% random missing values that were explicitly requested. This made the synthetic data easier to work with, but it doesn't reflect how messy real experimental data actually tends to be.

## Key findings

- **Claude and Gemini** were the most physically consistent of the four: both reproduced the Hall-Petch relationship, the strength-ductility trade-off, and the expected effect of heat treatment on strength.
- **ChatGPT** generated numerically plausible values but didn't capture the underlying physical relationships- several plots showed scattered or step-like patterns instead of smooth physical trends.
- **Grok** performed the weakest overall, with a number of physically implausible points and values that fall outside real-world limits for this alloy class.

## Conclusion

LLMs are useful for quickly producing structured, schema-consistent synthetic data, but this comparison shows they can't yet be fully trusted to simulate accurate physical behavior without oversight. The need for extensive cleaning, and the fact that some models failed to replicate even basic processing-property trends, confirms that AI-generated data should always be validated against real experimental evidence before being used for any technical analysis.
