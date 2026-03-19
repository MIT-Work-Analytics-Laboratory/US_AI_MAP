# May 2024 OEWS Estimates Dataset

Data description for the **Occupational Employment and Wage Statistics (OEWS)** Survey.
Produced by the Bureau of Labor Statistics (BLS), Department of Labor.

**Reference Links:**
* **Website:** [www.bls.gov/oes](https://www.bls.gov/oes)
* **Contact:** oewsinfo@bls.gov

---

## 1. Field Definitions

### Geography & Industry
| Field | Description |
| :--- | :--- |
| **area** | U.S. (99), state FIPS code, Metropolitan Statistical Area (MSA) code, or OEWS-specific nonmetropolitan area code. |
| **area_title** | Area name. |
| **area_type** | 1=U.S.; 2=State; 3=U.S. Territory; 4=MSA; 6=Nonmetropolitan Area. |
| **prim_state** | The primary state for the given area. "US" is used for national estimates. |
| **naics** | North American Industry Classification System (NAICS) code for the given industry. |
| **naics_title** | NAICS title for the given industry. |
| **i_group** | Industry level (Sector, 3-digit, 4-digit, 5-digit, or 6-digit). |
| **own_code** | Ownership type (1=Federal; 2=State; 3=Local; 5=Private; 123=All Gov; 57=Private Gambling; 58=Private/Local Gov Hospitals; etc). |

### Occupation & Classification
| Field | Description |
| :--- | :--- |
| **occ_code** | The 6-digit Standard Occupational Classification (SOC) code. |
| **occ_title** | SOC title or OEWS-specific title for the occupation. |
| **o_group** | SOC occupation level (Total, Major, Minor, Broad, or Detailed). |

### Employment Metrics
| Field | Description |
| :--- | :--- |
| **tot_emp** | Estimated total employment rounded to the nearest 10 (excludes self-employed). |
| **emp_prse** | Percent relative standard error (PRSE) for the employment estimate. Measures sampling error/precision. |
| **jobs_1000** | Number of jobs in the occupation per 1,000 jobs in the given area. |
| **loc_quotient** | Ratio of an occupation's share of local employment vs. its share of U.S. employment. |

### Wage Metrics (Hourly & Annual)
| Field | Description |
| :--- | :--- |
| **h_mean / a_mean** | Mean hourly / annual wage. |
| **mean_prse** | Percent relative standard error for the mean wage estimate. |
| **h_pct(10,25,50,75,90)** | Hourly wage percentiles (Median = 50th). |
| **a_pct(10,25,50,75,90)** | Annual wage percentiles (Median = 50th). |

---

## 2. Special Flags & Distribution
* **pct_total:** Percent of industry employment in the given occupation (National only).
* **pct_rpt:** Percent of establishments reporting the occupation (National only).
* **annual:** "TRUE" if only annual wages are released (e.g., teachers, pilots).
* **hourly:** "TRUE" if only hourly wages are released (e.g., actors, dancers).

---

## 3. Data Key & Special Symbols

| Symbol | Definition |
| :--- | :--- |
| `*` | Wage estimate is not available. |
| `**` | Employment estimate is not available. |
| `#` | Wage is high-cap: $\ge$ $115.00/hr or $\ge$ $239,200/yr. |
| `~` | Percent of establishments reporting the occupation is < 0.5%. |

---

### Usage Note
> [!IMPORTANT]
> Not all fields are available for every type of estimate. For example, `loc_quotient` and `jobs_1000` are only available for state and MSA estimates; they will be blank in national industry-level files.