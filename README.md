# EHR-TO-CLIF

## Overview

This repository is a practical guide to **CLIFing**—the process of transforming local clinical EHR data into a CLIF.

## 📚 Key Resources

| Resource | Description |
|----------|-------------|
| **[Epic UserWeb: CLIF SQL Queries](https://userweb.epic.com/Thread/136603/CLIF-A-Common-Longitudinal-ICU-data-Format-for-Federated-Cri/)** | SQL queries for Epic Caboodle/Clarity to extract CLIF tables (requires Epic UserWeb access) |
| **[CLIF-MIMIC](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-MIMIC)** | Reference ETL implementation converting MIMIC-IV → CLIF |
| **[CLIF Data Dictionary](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF)** | Official schema definitions and mCIDE vocabularies |
| **[CLIF 101](https://common-longitudinal-icu-data-format.github.io/CLIF-101/)** | Beginner's guide to working with CLIF data |
| **[clifpy](https://github.com/Common-Longitudinal-ICU-data-Format/clifpy)** | Python package for validation and analysis |
| **[CLIF Lighthouse](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Lighthouse)** | Interactive QC dashboard |

---

Different institutions take different paths to build their CLIF instances. There is **no one-size-fits-all solution**, as source EHR systems, data availability, and local constraints vary widely.

The goal of this repo is to document **how we built our CLIF pipeline**, starting from a local EHR and progressing step by step to a CLIF-ready dataset, and to share lessons learned along the way.
(we try to share as much as we can)

## What This Repository Covers

- The overall pathway from **EHR → CLIF**
- Design decisions and trade-offs we made
- Mapping strategies for diagnoses, procedures, labs, and ICU concepts
- Common challenges encountered during CLIF implementation
- Practical examples and scripts used in our pipeline

This is not intended to be a strict template, but rather a **reference and starting point** for other teams building their own CLIF instances.

## What This Repository Is Not

- A plug-and-play CLIF solution
- A definitive or “correct” way to build CLIF
- A replacement for official CLIF documentation

Instead, it reflects **one real-world implementation** and is meant to complement existing CLIF resources.

## Intended Audience

- Researchers and engineers working with ICU or clinical EHR data
- Institutions planning to adopt or pilot CLIF
- Teams looking for practical examples beyond high-level specifications

## How to Use This Repo

- Review the documented pipeline to understand the full CLIFing process
- Adapt relevant steps to your local data and infrastructure
- Use examples as guidance, not strict requirements

## Contributing Your Site's ETL

We encourage sites to share their ETL approaches! To contribute:

1. Create a folder with your site/institution name (e.g., `RUSH/`, `UCMC/`)
2. Add your data transformation scripts (censored of any PHI or proprietary details)
3. Include a brief README explaining your EHR source and approach
4. Submit a PR

The more examples we collect, the easier it becomes for new sites to onboard.

## Questions?

- Join the **#clif-code-ecosystem** channel in the CLIF Consortium Slack
- Check the [Epic UserWeb thread](https://userweb.epic.com/Thread/136603/) for Epic-specific discussions
- Review [CLIF-MIMIC](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-MIMIC) for a complete reference implementation
