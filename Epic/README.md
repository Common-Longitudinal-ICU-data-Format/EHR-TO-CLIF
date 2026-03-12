# Epic → CLIF ETL Guide

This guide documents pathways for transforming Epic EHR data into CLIF format.

## 📚 Key Epic Resources

### Public Resources (No Login Required)

| Resource | URL | Description |
|----------|-----|-------------|
| **EHI Export Schema** | https://open.epic.com/EHITables/GetTable/_index.htm | Complete schema documentation for Epic EHI tables (Feb 2026) |
| **Clarity Data Handbook** | https://datahandbook.epic.com/ClarityDictionary/Details | Clarity table documentation (some tables differ from EHI) |

### Epic UserWeb (Epic Login Required)

| Resource | URL | Description |
|----------|-----|-------------|
| **CLIF SQL Queries** | https://userweb.epic.com/Thread/136603/ | Pre-built SQL queries for Epic Caboodle/Clarity |

## 🗂️ Epic to CLIF Table Mappings

The following sections map Epic EHI/Clarity tables to their corresponding CLIF tables.

---

### Patient Demographics (`patient`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `patient_id` | PATIENT | PAT_ID | Primary patient identifier |
| `race_category` | PATIENT_RACE | PATIENT_RACE_C → ZC_PATIENT_RACE | Map to mCIDE race categories |
| `ethnicity_category` | PATIENT | ETHNIC_GROUP_C → ZC_ETHNIC_GROUP | Map to mCIDE ethnicity categories |
| `sex_category` | PATIENT | SEX_C → ZC_SEX | M/F/Other |
| `date_of_birth` | PATIENT | BIRTH_DATE | |

**Key Tables:**
- `PATIENT` - Core patient demographics
- `PATIENT_RACE` - Patient race (multiple allowed)
- `PATIENT_ETHNICITY` - Patient ethnicity
- `PATIENT_3` - Additional patient info

---

### Hospitalization (`hospitalization`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `hospitalization_id` | PAT_ENC_HSP | PAT_ENC_CSN_ID | Contact Serial Number |
| `patient_id` | PAT_ENC_HSP | PAT_ID | |
| `admission_dttm` | PAT_ENC_HSP | HOSP_ADMSN_TIME | |
| `discharge_dttm` | PAT_ENC_HSP | HOSP_DISCH_TIME | |
| `age_at_admission` | Calculated | BIRTH_DATE vs HOSP_ADMSN_TIME | Calculate from patient DOB |
| `admission_type_category` | PAT_ENC_HSP | ADT_PAT_CLASS_C | Map to mCIDE categories |
| `discharge_disposition_category` | PAT_ENC_HSP | DISCH_DISP_C | Map to mCIDE categories |

**Key Tables:**
- `PAT_ENC_HSP` - Hospital encounters (primary source)
- `PAT_ENC` - All patient encounters
- `HSP_ACCOUNT` - Hospital account information
- `PAT_ENC_2` - Additional encounter info

---

### ADT Events (`adt`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `hospitalization_id` | CLARITY_ADT | PAT_ENC_CSN_ID | |
| `in_dttm` | CLARITY_ADT | EFFECTIVE_TIME | Event start |
| `out_dttm` | CLARITY_ADT | NEXT_OUT_EVENT_ID → EFFECTIVE_TIME | Link to next event |
| `location_category` | CLARITY_ADT | ADT_DEPARTMENT_ID → CLARITY_DEP | Map dept to mCIDE location_category |
| `location_type` | CLARITY_ADT | ROOM_ID, BED_ID | ICU vs Floor vs ED |

**Key Tables:**
- `CLARITY_ADT` - ADT event records
- `ADT_ORDER_INFORMATION` - ADT orders
- `CLARITY_DEP` - Department master
- `CLARITY_LOC` - Location master
- `CLARITY_BED` - Bed master

**Tips:**
- Use `EVENT_TYPE_C` to filter: 1=Admission, 2=Transfer, 3=Discharge
- Link to `CLARITY_DEP` to get department names and determine ICU status
- `ADT_PATIENT_STAT_C` indicates patient status

---

### Vitals (`vitals`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `hospitalization_id` | IP_FLWSHT_MEAS | INPATIENT_DATA_ID → IP_DATA_STORE.PAT_ENC_CSN_ID | |
| `recorded_dttm` | IP_FLWSHT_MEAS | RECORDED_TIME | |
| `vital_category` | IP_FLWSHT_MEAS | FLO_MEAS_ID → IP_FLO_GP_DATA | Map flowsheet row to mCIDE |
| `vital_value` | IP_FLWSHT_MEAS | MEAS_VALUE | |

**Key Tables:**
- `IP_FLWSHT_MEAS` - Inpatient flowsheet measurements (primary)
- `IP_FLWSHT_REC` - Flowsheet records
- `IP_FLO_GP_DATA` - Flowsheet row definitions
- `IP_DATA_STORE` - Links flowsheets to encounters

**Common Flowsheet Rows (varies by site):**
| Vital | Common FLO_MEAS_ID | Description |
|-------|-------------------|-------------|
| Heart Rate | 5 | Pulse |
| SpO2 | 10 | Oxygen saturation |
| Temperature | 6 | Body temperature |
| Respiratory Rate | 9 | Breathing rate |
| SBP | 3 | Systolic BP |
| DBP | 4 | Diastolic BP |
| MAP | 11 | Mean arterial pressure |
| Weight | 14 | Patient weight |
| Height | 301070 | Patient height |

⚠️ **Note:** Flowsheet row IDs are site-specific. Check your site's `IP_FLO_GP_DATA` mapping.

---

### Labs (`labs`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `hospitalization_id` | ORDER_PROC | PAT_ENC_CSN_ID | |
| `lab_order_dttm` | ORDER_PROC | ORDER_TIME | |
| `lab_result_dttm` | ORDER_RESULTS | RESULT_TIME | |
| `lab_category` | ORDER_RESULTS | COMPONENT_ID → CLARITY_COMPONENT | Map to mCIDE lab categories |
| `lab_value` | ORDER_RESULTS | ORD_VALUE | |
| `reference_unit` | ORDER_RESULTS | REFERENCE_UNIT | |

**Key Tables:**
- `ORDER_PROC` - Lab orders
- `ORDER_RESULTS` - Lab results (link via ORDER_PROC_ID)
- `ORDER_RES_COMP_CMT` - Result comments
- `CLARITY_COMPONENT` - Component definitions
- `LAB_RESULT_CM` - Additional result info

**Example Lab Mappings:**

| mCIDE Category | Common Component Names |
|----------------|----------------------|
| `creatinine` | CREATININE, CREAT |
| `sodium` | SODIUM, NA |
| `potassium` | POTASSIUM, K |
| `hemoglobin` | HEMOGLOBIN, HGB |
| `lactate` | LACTATE, LACTIC ACID |
| `pao2` | PO2 ARTERIAL, PAO2 |
| `paco2` | PCO2 ARTERIAL, PACO2 |
| `ph_arterial` | PH ARTERIAL |

---

### Medication Orders (`medication_orders`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `hospitalization_id` | ORDER_MED | PAT_ENC_CSN_ID | |
| `order_id` | ORDER_MED | ORDER_MED_ID | |
| `med_order_dttm` | ORDER_MED | ORDER_INST | Order instant |
| `med_name` | ORDER_MED | MEDICATION_ID → CLARITY_MEDICATION.NAME | |
| `med_category` | ORDER_MED | MEDICATION_ID → mapping | Map to mCIDE categories |
| `med_dose` | ORDER_MED | HV_DISCRETE_DOSE | |
| `med_dose_unit` | ORDER_MED | HV_DOSE_UNIT_C | |
| `med_route` | ORDER_MED | MED_ROUTE_C → ZC_MED_ROUTE | |

**Key Tables:**
- `ORDER_MED` - Medication orders
- `ORDER_MED_2`, `ORDER_MED_3`, `ORDER_MED_4` - Additional order info
- `CLARITY_MEDICATION` - Medication master
- `RX_MED_ONE` - Pharmacy medication info
- `MAR_ADMIN_INFO` - Administration records

---

### Medication Administration - Continuous (`medication_admin_continuous`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `hospitalization_id` | MAR_ADMIN_INFO | PAT_ENC_CSN_ID | Via ORDER_MED link |
| `admin_dttm` | MAR_ADMIN_INFO | TAKEN_TIME | |
| `med_name` | MAR_ADMIN_INFO | ORDER_MED_ID → CLARITY_MEDICATION.NAME | |
| `med_category` | Derived | Map medication to mCIDE | |
| `med_dose` | MAR_ADMIN_INFO | MAR_INF_RATE | Infusion rate |
| `med_dose_unit` | MAR_ADMIN_INFO | MAR_INF_RATE_UNIT_C | Rate unit |

**Key Tables:**
- `MAR_ADMIN_INFO` - MAR records (filter for continuous infusions)
- `IP_MAR_LINE_INFO` - Line-level MAR info
- `MAR_ADMIN_ADDL_INFO` - Additional admin info

**Tips:**
- Filter for continuous infusions using `MAR_ACTION_C` or medication type
- Look for `MAR_INF_RATE` being populated
- Common continuous meds: vasopressors, sedatives, insulin drips

---

### Medication Administration - Intermittent (`medication_admin_intermittent`)

Same source tables as continuous, but filter for:
- PRN medications
- Scheduled intermittent doses
- `MAR_INF_RATE` is NULL or not applicable

---

### Respiratory Support (`respiratory_support`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `hospitalization_id` | IP_FLWSHT_MEAS | Via IP_DATA_STORE | |
| `recorded_dttm` | IP_FLWSHT_MEAS | RECORDED_TIME | |
| `device_category` | IP_FLWSHT_MEAS | FLO_MEAS_ID → mapping | Site-specific flowsheet rows |
| `mode_category` | IP_FLWSHT_MEAS | MEAS_VALUE (mode row) | |
| `fio2` | IP_FLWSHT_MEAS | MEAS_VALUE (FiO2 row) | |
| `peep` | IP_FLWSHT_MEAS | MEAS_VALUE (PEEP row) | |
| `tidal_volume_set` | IP_FLWSHT_MEAS | MEAS_VALUE (TV row) | |

**Common Respiratory Flowsheet Rows:**

| Parameter | Description |
|-----------|-------------|
| R OXYGEN FLOW RATE | O2 flow rate (L/min) |
| R OXYGEN DEVICE | Oxygen delivery device |
| R FIO2 | Fraction inspired O2 |
| R PEEP/CPAP | PEEP setting |
| R SET RESP RATE | Set respiratory rate |
| R MODE OF VENTILATION | Vent mode |
| R TIDAL VOLUME (SET) | Set tidal volume |
| R TIDAL VOLUME (MEAS) | Measured tidal volume |

---

### CRRT Therapy (`crrt_therapy`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `hospitalization_id` | IP_FLWSHT_MEAS | Via IP_DATA_STORE | |
| `recorded_dttm` | IP_FLWSHT_MEAS | RECORDED_TIME | |
| `crrt_mode_category` | IP_FLWSHT_MEAS | Mode flowsheet row | CVVH, CVVHD, CVVHDF, etc. |

**Also check:**
- Procedure orders for dialysis initiation
- Dialysis-specific flowsheet templates

---

### Microbiology Cultures (`microbiology_culture`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `hospitalization_id` | ORDER_PROC | PAT_ENC_CSN_ID | |
| `collect_dttm` | ORDER_PROC | SPECIMN_TAKEN_TIME | |
| `result_dttm` | ORDER_RESULTS | RESULT_TIME | |
| `organism_category` | MICRO_ORGANISM | ORGANISM_ID | Map to mCIDE |
| `fluid_category` | ORDER_PROC | SPECIMEN_TYPE_C | Map to mCIDE |

**Key Tables:**
- `ORDER_PROC` - Culture orders
- `ORDER_RESULTS` - Culture results
- `MICRO_ORGANISM` - Organism results
- `SUSCEPTIBILITY` - Antibiotic susceptibilities
- `SENS_ORGANISM_SYN` - Organism synonyms

---

### Sensitivity (`sensitivity`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `antibiotic` | SUSCEPTIBILITY | ANTIBIOTIC_C | |
| `susceptibility_category` | SUSCEPTIBILITY | SUSCEPT_C | S/I/R |
| `mic_value` | SUSCEPTIBILITY | MIC_VALUE | |

---

### Position (`position`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `position_category` | IP_FLWSHT_MEAS | Position flowsheet row | Prone, supine, etc. |

**Common Position Flowsheet Rows:**
- "Position" or "Patient Position"
- Look for values: Prone, Supine, Lateral, Sitting

---

### Code Status (`code_status`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `code_status_category` | ADVANCE_DIRECTIVE, CODE_STATUS flowsheet | | |

**Key Tables:**
- Flowsheet rows documenting code status
- `ADVANCE_DIRECTIVE` - Advance directive records
- Custom documentation in notes

---

### Hospital Diagnosis (`hospital_diagnosis`)

| CLIF Field | Epic EHI Table(s) | Epic Column(s) | Notes |
|------------|-------------------|----------------|-------|
| `hospitalization_id` | HSP_ACCT_DX_LIST | PAT_ENC_CSN_ID | |
| `diagnosis_code` | HSP_ACCT_DX_LIST | DX_ID → EDG_CURRENT_ICD10 | ICD-10 code |
| `diagnosis_name` | EDG_CURRENT_ICD10 | DX_NAME | |
| `diagnosis_type` | HSP_ACCT_DX_LIST | DX_ED_YN, PRINCIPAL_DX_YN | Primary/secondary |

**Key Tables:**
- `HSP_ACCT_DX_LIST` - Hospital account diagnoses
- `PAT_ENC_DX` - Encounter diagnoses
- `EDG_CURRENT_ICD10` - ICD-10 diagnosis master
- `CLARITY_EDG` - Diagnosis master

---

## 🔧 Implementation Tips

### 1. Start with EHI Tables

The [EHI Export Schema](https://open.epic.com/EHITables/GetTable/_index.htm) is publicly accessible and provides:
- Table descriptions
- Primary key information
- Column details with data types

### 2. Site-Specific Flowsheet Mapping

Flowsheet row IDs (`FLO_MEAS_ID`) are **site-specific**. You'll need to:

1. Query `IP_FLO_GP_DATA` to find your site's row IDs
2. Create a mapping CSV: `flowsheet_row_id` → `clif_category`
3. Document common row names for reference

### 3. Medication Categorization

Epic medications need mapping to mCIDE categories. Approaches:

1. **By therapeutic class:** Use RX_MED_ONE.THERA_CLASS_C
2. **By medication name:** Pattern matching on CLARITY_MEDICATION.NAME
3. **By Generic name:** Use SIMPLE_GENERIC_C

### 4. Validate Against CLIF Schema

After extraction, validate using:
- [clifpy](https://github.com/Common-Longitudinal-ICU-data-Format/clifpy) validation
- [CLIF Lighthouse](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-Lighthouse) QC dashboard

---

## 📁 Files in This Directory

| File | Description |
|------|-------------|
| `README.md` | This guide |
| `epic_ehi_clif_mapping.csv` | Detailed field-level mappings |
| `common_flowsheet_rows.csv` | Example flowsheet row mappings |

---

## 🤝 Contributing

Found a better mapping? Have queries that worked for your site?

1. Add your site-specific SQL or mapping files
2. Document any gotchas you encountered
3. Submit a PR!

---

## 📚 Additional Resources

- [CLIF Data Dictionary](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF)
- [CLIF-MIMIC](https://github.com/Common-Longitudinal-ICU-data-Format/CLIF-MIMIC) - Reference ETL implementation
- [clifpy Documentation](https://github.com/Common-Longitudinal-ICU-data-Format/clifpy)
- [Epic UserWeb CLIF Thread](https://userweb.epic.com/Thread/136603/) (Epic login required)
