---
marp: true
theme: default
paginate: true
style: |
  .columns {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    gap: 1rem;
  }
  section.title {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
  section.title h1 {
    font-size: 3.5rem;
    font-weight: 700;
    margin: 0;
  }
  .dataframe {
    width: 39%;
    height: auto;
    margin-left: 0;
    margin-right: auto;
    border-collapse: collapse;
    font-size: 0.6em;

  }
  .dataframe th, .dataframe td {
    padding: 0.3em;
    text-align: left;
  }

---

<!-- _class: title -->

# Examining IHC Algorithms

---

# Intial look at data
----------------------------------------------------------------------------------------------------------------------

## Dataset overview
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th>Metric</th>
      <th>Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Total Records</td>
      <td>20,000</td>
    </tr>
    <tr>
      <td>Number of Sites</td>
      <td>2</td>
    </tr>
    <tr>
      <td>ED Patients</td>
      <td>5,528 (27.6%)</td>
    </tr>
    <tr>
      <td>IN Patients</td>
      <td>14,472 (72.4%)</td>
    </tr>
    <tr>
      <td>Male Patients</td>
      <td>10,780 (53.9%)</td>
    </tr>
    <tr>
      <td>Female Patients</td>
      <td>9,220 (46.1%)</td>
    </tr>
    <tr>
      <td>Age Mean ± Std</td>
      <td>48.5 ± 17.9</td>
    </tr>
    <tr>
      <td>Age Range</td>
      <td>5 - 86</td>
    </tr>
    <tr>
      <td>Treatment Duration Mean ± Std (min)</td>
      <td>10.32 ± 2.45</td>
    </tr>
    <tr>
      <td>Treatment Duration Range (min)</td>
      <td>-0.27 - 20.27</td>
    </tr>
  </tbody>
</table>

---

## Treatment duration by patient class
ED treatment duration is shorter than IN on average, with a narrower distribution.
  
![](images/duration_by_class.png)  

----
##  Treatment duration by site and patient class

<div class="columns">
  <div>
  'healthy_vibes' process IN patients faster than 'best_doctors', while the opposite is true for ED patients. 
  </div>

  <div>

  ![](images/duration_per_site_class.png)

  </div>
</div>

----
## Gender distribution by patient class
Male patients are significantly overrepresented in the ED population compared to IN where the proportions are balanced.
![w:750](images/gender_by_class.png)

---
# Algo personalities

--------------------------------------------------------------------------------------------------------------------
## Speed comparison
![](images/algo_mean_run_time.png)


## Algo 1 - The Sharpshooter 🎯 
<!-- 🏹 -->
Super fast, high PPV, low sensitivity.

![](images/algo_compare_overall_highlight_algo1.png)

---
## Algo 2 - The Catch-all 😋
Moderate speed, high sensitivity, low PPV.
![](images/algo_compare_overall_highlight_algo2.png)

---

## Algo 3 - The Thorough one 🧙‍♂️
Slow, high performance across all metrics.
![](images/algo_compare_overall_highlight_algo3.png)

---

# Algos in the wild
----------------------------------------------------------------------------------------------------

## Patient class
<div class="columns">
  <div>

  - PPV is considerably lower in ED patients across all algorithms, likely owing to different CT administration practices (broad screening for rule-out in the ED versus targeted diagnostic monitoring in the IN).
  
  - Algo1 demonstrates the most robust PPV stability, showing only a 1.2x reduction in the ED setting, compared to 1.4x for Algo3 and a significant 2.5x drop for Algo2.
  </div>

  <div>

   ![](images/ppv_per_algo_by_class.png)

  </div>
</div>


---

## Age groups


- Algo2's specificity and PPV drops sharply with age, indicating a higher false positive rate in elderly patients likely due to age-related mimics.
- Alog3 maintains consistent performance across age groups, suggesting robustness against anatomical complexities in older patients.

![](images/metrics_by_age_side_by_side.png)

---

# Key recommendation 
Combine all three algorithms into a unified diagnostic engine that leverages their complementary strengths to create a robust, multi-stage detection pipeline. 

----------------------------------------------------------------------------------------------------

## Algo 1: Worklist Prioritization

Role: Flag positive cases to prioritize critical patient review.

Key Rationale:

- **High PPV Stability:** Demonstrated robust resistance to low-prevalence environments (1.2x PPV drop). High reliability minimizes workflow interruptions from false alarms.

- **Throughput Efficiency:** Fast processing enables real-time queue re-ordering, ensuring positive cases are identified promptly despite high case volumes.

- **Rule-In Capability:** While lower sensitivity limits use for diagnosis (rule-out), high specificity makes it effective for identifying clear positives (rule-in).

---

## Algo 2: Exclusionary Safety Net

Role: Screen low-risk patients to facilitate rapid discharge of negative cases.

Key Rationale:

- **High Sensitivity:** Maximizes detection of potential bleeds. In a "rule-out" workflow, this minimizes the risk of false negatives.

- **PPV Trade-off:** A significant PPV reduction in ED settings reflects a higher false positive rate. This is an acceptable trade-off for safety, as dismissing false positives is preferable to missing true pathology.

- **Operational Speed:** Intermediate processing speed is adequate for clearing the volume of routine negative scans.
---

## Algo 3: Expert Secondary Review

Role: Verify complex cases and resolve ambiguous findings.

Key Rationale:

- **Age-Invariant Specificity:** Maintains consistent performance across age groups, unlike other models affected by mimics (e.g., calcifications). Best suited for complex anatomy.

- **Depth over Speed:** Slower processing time limits utility for real-time triage but supports a rigorous secondary review where precision is prioritized over speed.


---
# The full workflow 

![center w:2500 center h:500](images/flow.svg)

<!-- Step 1: The CT scan is uploaded to the system.

Step 2: The Parallel Split Two algorithms run at the same time:

- Algo 1 (The Sprint): Looks for obvious, critical bleeds.

    - Result: If it finds one, the case goes straight to the Top of the Worklist (Priority).

- Algo 2 (The Sweep): Looks for any sign of trouble to ensure safety.
  - Result: If it finds nothing, the patient is marked for Fast-Track Discharge.

  - Result: If it finds something suspicious, it calls Algo 3.

Step 3: The Expert Review (Conditional) If Algo 2 flagged something suspicious:

  - Algo 3 (The Expert): Reviews the specific area of concern.

  - If it's a real bleed: Moves case to Top of the Worklist.

  - If it's a mimic (e.g., calcification): Moves case to the Standard Queue. -->

---

# Putting superAlgo to the test
----------------------------------------------------------------------------------------------------

## Better performance at faster speeds
SuperAlgo shows a slight improvement over Algo3 in both sensitivity and PPV, while running significantly faster.
<div class="columns">
  <div>

  ![](images/algo_compare_overall_highlight_superalgo.png)

  </div>

  <div>

   ![](images/algo_mean_run_time_w_super.png)

  </div>
</div>

## 

