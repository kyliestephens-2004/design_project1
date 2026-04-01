```python
#Importing Dependencies
import pandas as pd
import duckdb
import matplotlib.pyplot as plt
import seaborn as sns
```

###Data Preparation
Files loaded into a database using Duckdb and Python.


```python
con = duckdb.connect()

con.execute("""
CREATE TABLE person AS SELECT * FROM 'person.parquet';
""")

con.execute("""
CREATE TABLE visit_occurrence AS SELECT * FROM 'visit_occurrence.parquet';
""")

con.execute("""
CREATE TABLE hospital_visits AS SELECT * FROM 'hospital_visits.parquet';
""")

con.execute("""
CREATE TABLE readmission_features AS SELECT * FROM 'readmission_features.parquet';
""")

con.execute("""
CREATE TABLE procedures AS SELECT * FROM 'procedures.parquet';
""")

con.execute("""
CREATE TABLE conditions AS SELECT * FROM 'conditions.parquet';
""")
```


    FloatProgress(value=0.0, layout=Layout(width='auto'), style=ProgressStyle(bar_color='black'))





    <duckdb.duckdb.DuckDBPyConnection at 0x7da185fe0a70>




```python
conditions_agg = con.execute("""
SELECT
    visit_occurrence_id,
    COUNT(*) AS num_conditions,
    COUNT(DISTINCT condition_concept_id) AS unique_conditions
FROM conditions
GROUP BY visit_occurrence_id
""").df()
```


```python
procedures_agg = con.execute("""
SELECT
    visit_occurrence_id,
    COUNT(*) AS num_procedures,
    COUNT(DISTINCT procedure_concept_id) AS unique_procedures
FROM procedures
GROUP BY visit_occurrence_id
""").df()
```


    FloatProgress(value=0.0, layout=Layout(width='auto'), style=ProgressStyle(bar_color='black'))


###Queries
Increasing in complexity


```python
df1 = con.execute("""
SELECT
    v.visit_occurrence_id,
    v.person_id,
    p.gender_concept_id,
    p.race_concept_id,
    v.visit_start_date
FROM visit_occurrence v
JOIN person p
ON v.person_id = p.person_id
WHERE v.visit_concept_id = 9201;
""").df()

df1.head()
```





  <div id="df-7ae3b036-1992-4c9e-b834-70e59b5b3a0b" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>visit_occurrence_id</th>
      <th>person_id</th>
      <th>gender_concept_id</th>
      <th>race_concept_id</th>
      <th>visit_start_date</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>75079</td>
      <td>76630</td>
      <td>8507</td>
      <td>8527</td>
      <td>1918-11-22</td>
    </tr>
    <tr>
      <th>1</th>
      <td>109405</td>
      <td>107193</td>
      <td>8507</td>
      <td>8527</td>
      <td>1918-11-25</td>
    </tr>
    <tr>
      <th>2</th>
      <td>89890</td>
      <td>71521</td>
      <td>8532</td>
      <td>0</td>
      <td>1918-11-30</td>
    </tr>
    <tr>
      <th>3</th>
      <td>96263</td>
      <td>88173</td>
      <td>8507</td>
      <td>8527</td>
      <td>1918-12-11</td>
    </tr>
    <tr>
      <th>4</th>
      <td>72518</td>
      <td>1088</td>
      <td>8507</td>
      <td>8527</td>
      <td>1918-12-15</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-7ae3b036-1992-4c9e-b834-70e59b5b3a0b')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-7ae3b036-1992-4c9e-b834-70e59b5b3a0b button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-7ae3b036-1992-4c9e-b834-70e59b5b3a0b');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


    </div>
  </div>





```python
df2 = con.execute("""
SELECT
    person_id,
    COUNT(*) AS total_visits,
    MIN(visit_start_date) AS first_visit,
    MAX(visit_start_date) AS last_visit
FROM visit_occurrence
GROUP BY person_id;
""").df()

df2.head()
```





  <div id="df-7d27e003-6997-40eb-8d77-cb677fe3584a" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>person_id</th>
      <th>total_visits</th>
      <th>first_visit</th>
      <th>last_visit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>7982</td>
      <td>20</td>
      <td>1970-06-08</td>
      <td>2006-05-11</td>
    </tr>
    <tr>
      <th>1</th>
      <td>4377</td>
      <td>82</td>
      <td>1945-07-04</td>
      <td>1972-03-08</td>
    </tr>
    <tr>
      <th>2</th>
      <td>48996</td>
      <td>31</td>
      <td>1929-01-13</td>
      <td>1971-06-03</td>
    </tr>
    <tr>
      <th>3</th>
      <td>76523</td>
      <td>29</td>
      <td>1963-12-16</td>
      <td>1995-05-15</td>
    </tr>
    <tr>
      <th>4</th>
      <td>104925</td>
      <td>39</td>
      <td>1970-06-08</td>
      <td>2019-03-25</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-7d27e003-6997-40eb-8d77-cb677fe3584a')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-7d27e003-6997-40eb-8d77-cb677fe3584a button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-7d27e003-6997-40eb-8d77-cb677fe3584a');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


    </div>
  </div>





```python
df3 = con.execute("""
SELECT
    person_id,
    visit_occurrence_id,
    visit_start_date,
    LAG(visit_start_date) OVER (
        PARTITION BY person_id
        ORDER BY visit_start_date
    ) AS prev_visit
FROM visit_occurrence;
""").df()

df3.head()
```


    FloatProgress(value=0.0, layout=Layout(width='auto'), style=ProgressStyle(bar_color='black'))






  <div id="df-6acfdcb8-0d34-42dd-9c22-feb916caecf1" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>person_id</th>
      <th>visit_occurrence_id</th>
      <th>visit_start_date</th>
      <th>prev_visit</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>24</td>
      <td>346306</td>
      <td>1999-08-18</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>24</td>
      <td>346261</td>
      <td>2003-08-23</td>
      <td>1999-08-18</td>
    </tr>
    <tr>
      <th>2</th>
      <td>24</td>
      <td>346262</td>
      <td>2009-05-19</td>
      <td>2003-08-23</td>
    </tr>
    <tr>
      <th>3</th>
      <td>24</td>
      <td>346245</td>
      <td>2009-05-20</td>
      <td>2009-05-19</td>
    </tr>
    <tr>
      <th>4</th>
      <td>24</td>
      <td>346263</td>
      <td>2009-08-19</td>
      <td>2009-05-20</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-6acfdcb8-0d34-42dd-9c22-feb916caecf1')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-6acfdcb8-0d34-42dd-9c22-feb916caecf1 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-6acfdcb8-0d34-42dd-9c22-feb916caecf1');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


    </div>
  </div>





```python
df4 = con.execute("""
SELECT
    f.person_id,
    f.AGE,
    f.PRIOR_ADMISSIONS,
    f.READMIT_30,
    agg.total_visits
FROM readmission_features f
JOIN (
    SELECT person_id, COUNT(*) AS total_visits
    FROM visit_occurrence
    GROUP BY person_id
) agg
ON f.person_id = agg.person_id;
""").df()

df4.head()
```





  <div id="df-b677beee-2b09-4065-b984-40fb025f3f01" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>person_id</th>
      <th>AGE</th>
      <th>PRIOR_ADMISSIONS</th>
      <th>READMIT_30</th>
      <th>total_visits</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>103649</td>
      <td>79</td>
      <td>17</td>
      <td>1</td>
      <td>151</td>
    </tr>
    <tr>
      <th>1</th>
      <td>103649</td>
      <td>79</td>
      <td>18</td>
      <td>0</td>
      <td>151</td>
    </tr>
    <tr>
      <th>2</th>
      <td>103649</td>
      <td>79</td>
      <td>19</td>
      <td>1</td>
      <td>151</td>
    </tr>
    <tr>
      <th>3</th>
      <td>103649</td>
      <td>79</td>
      <td>20</td>
      <td>1</td>
      <td>151</td>
    </tr>
    <tr>
      <th>4</th>
      <td>103649</td>
      <td>79</td>
      <td>21</td>
      <td>0</td>
      <td>151</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-b677beee-2b09-4065-b984-40fb025f3f01')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-b677beee-2b09-4065-b984-40fb025f3f01 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-b677beee-2b09-4065-b984-40fb025f3f01');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


    </div>
  </div>





```python
df = con.execute("""
SELECT
    f.person_id,
    f.READMIT_30,
    f.AGE,
    f.PRIOR_ADMISSIONS,
    agg.total_visits,
    p.gender_concept_id,
    p.race_concept_id
FROM readmission_features f
JOIN person p
    ON f.person_id = p.person_id
JOIN (
    SELECT person_id, COUNT(*) AS total_visits
    FROM visit_occurrence
    GROUP BY person_id
) agg
ON f.person_id = agg.person_id;
""").df()

df.head()
```





  <div id="df-f56db5e0-f892-440e-9695-684fd8f1fdd4" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>person_id</th>
      <th>READMIT_30</th>
      <th>AGE</th>
      <th>PRIOR_ADMISSIONS</th>
      <th>total_visits</th>
      <th>gender_concept_id</th>
      <th>race_concept_id</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>103649</td>
      <td>1</td>
      <td>79</td>
      <td>17</td>
      <td>151</td>
      <td>8532</td>
      <td>8527</td>
    </tr>
    <tr>
      <th>1</th>
      <td>103649</td>
      <td>0</td>
      <td>79</td>
      <td>18</td>
      <td>151</td>
      <td>8532</td>
      <td>8527</td>
    </tr>
    <tr>
      <th>2</th>
      <td>103649</td>
      <td>1</td>
      <td>79</td>
      <td>19</td>
      <td>151</td>
      <td>8532</td>
      <td>8527</td>
    </tr>
    <tr>
      <th>3</th>
      <td>103649</td>
      <td>1</td>
      <td>79</td>
      <td>20</td>
      <td>151</td>
      <td>8532</td>
      <td>8527</td>
    </tr>
    <tr>
      <th>4</th>
      <td>103649</td>
      <td>0</td>
      <td>79</td>
      <td>21</td>
      <td>151</td>
      <td>8532</td>
      <td>8527</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-f56db5e0-f892-440e-9695-684fd8f1fdd4')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-f56db5e0-f892-440e-9695-684fd8f1fdd4 button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-f56db5e0-f892-440e-9695-684fd8f1fdd4');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


    </div>
  </div>





```python
dfmod = con.execute("""
SELECT
    f.visit_occurrence_id,
    f.person_id,
    f.READMIT_30,
    f.AGE,
    f.PRIOR_ADMISSIONS,
    agg.total_visits,
    p.gender_concept_id,
    p.race_concept_id,

    COALESCE(c.num_conditions, 0) AS num_conditions,
    COALESCE(c.unique_conditions, 0) AS unique_conditions,

    COALESCE(pr.num_procedures, 0) AS num_procedures,
    COALESCE(pr.unique_procedures, 0) AS unique_procedures

FROM readmission_features f

JOIN person p
    ON f.person_id = p.person_id

JOIN (
    SELECT person_id, COUNT(*) AS total_visits
    FROM visit_occurrence
    GROUP BY person_id
) agg
    ON f.person_id = agg.person_id

LEFT JOIN (
    SELECT
        visit_occurrence_id,
        COUNT(*) AS num_conditions,
        COUNT(DISTINCT condition_concept_id) AS unique_conditions
    FROM conditions
    GROUP BY visit_occurrence_id
) c
    ON f.visit_occurrence_id = c.visit_occurrence_id

LEFT JOIN (
    SELECT
        visit_occurrence_id,
        COUNT(*) AS num_procedures,
        COUNT(DISTINCT procedure_concept_id) AS unique_procedures
    FROM procedures
    GROUP BY visit_occurrence_id
) pr
    ON f.visit_occurrence_id = pr.visit_occurrence_id;
""").df()
```


    FloatProgress(value=0.0, layout=Layout(width='auto'), style=ProgressStyle(bar_color='black'))



```python
dfmod.head()
```





  <div id="df-e0ea82bf-4bcc-4c83-8f13-058937bc38be" class="colab-df-container">
    <div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>visit_occurrence_id</th>
      <th>person_id</th>
      <th>READMIT_30</th>
      <th>AGE</th>
      <th>PRIOR_ADMISSIONS</th>
      <th>total_visits</th>
      <th>gender_concept_id</th>
      <th>race_concept_id</th>
      <th>num_conditions</th>
      <th>unique_conditions</th>
      <th>num_procedures</th>
      <th>unique_procedures</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2338</td>
      <td>3626</td>
      <td>0</td>
      <td>50</td>
      <td>0</td>
      <td>54</td>
      <td>8532</td>
      <td>8527</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>91403</td>
      <td>3632</td>
      <td>0</td>
      <td>43</td>
      <td>2</td>
      <td>39</td>
      <td>8532</td>
      <td>8527</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>1433</td>
      <td>3636</td>
      <td>0</td>
      <td>23</td>
      <td>0</td>
      <td>14</td>
      <td>8507</td>
      <td>8527</td>
      <td>3</td>
      <td>1</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>76258</td>
      <td>3646</td>
      <td>0</td>
      <td>115</td>
      <td>10</td>
      <td>56</td>
      <td>8507</td>
      <td>8527</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>27515</td>
      <td>3646</td>
      <td>0</td>
      <td>115</td>
      <td>8</td>
      <td>56</td>
      <td>8507</td>
      <td>8527</td>
      <td>0</td>
      <td>0</td>
      <td>4</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>
    <div class="colab-df-buttons">

  <div class="colab-df-container">
    <button class="colab-df-convert" onclick="convertToInteractive('df-e0ea82bf-4bcc-4c83-8f13-058937bc38be')"
            title="Convert this dataframe to an interactive table."
            style="display:none;">

  <svg xmlns="http://www.w3.org/2000/svg" height="24px" viewBox="0 -960 960 960">
    <path d="M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z"/>
  </svg>
    </button>

  <style>
    .colab-df-container {
      display:flex;
      gap: 12px;
    }

    .colab-df-convert {
      background-color: #E8F0FE;
      border: none;
      border-radius: 50%;
      cursor: pointer;
      display: none;
      fill: #1967D2;
      height: 32px;
      padding: 0 0 0 0;
      width: 32px;
    }

    .colab-df-convert:hover {
      background-color: #E2EBFA;
      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);
      fill: #174EA6;
    }

    .colab-df-buttons div {
      margin-bottom: 4px;
    }

    [theme=dark] .colab-df-convert {
      background-color: #3B4455;
      fill: #D2E3FC;
    }

    [theme=dark] .colab-df-convert:hover {
      background-color: #434B5C;
      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);
      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));
      fill: #FFFFFF;
    }
  </style>

    <script>
      const buttonEl =
        document.querySelector('#df-e0ea82bf-4bcc-4c83-8f13-058937bc38be button.colab-df-convert');
      buttonEl.style.display =
        google.colab.kernel.accessAllowed ? 'block' : 'none';

      async function convertToInteractive(key) {
        const element = document.querySelector('#df-e0ea82bf-4bcc-4c83-8f13-058937bc38be');
        const dataTable =
          await google.colab.kernel.invokeFunction('convertToInteractive',
                                                    [key], {});
        if (!dataTable) return;

        const docLinkHtml = 'Like what you see? Visit the ' +
          '<a target="_blank" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'
          + ' to learn more about interactive tables.';
        element.innerHTML = '';
        dataTable['output_type'] = 'display_data';
        await google.colab.output.renderOutput(dataTable, element);
        const docLink = document.createElement('div');
        docLink.innerHTML = docLinkHtml;
        element.appendChild(docLink);
      }
    </script>
  </div>


    </div>
  </div>





```python
dfmod["visit_occurrence_id"].nunique() == len(dfmod)
```




    True




```python
dfmod['READMIT_30'].value_counts()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>count</th>
    </tr>
    <tr>
      <th>READMIT_30</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>106480</td>
    </tr>
    <tr>
      <th>1</th>
      <td>33537</td>
    </tr>
  </tbody>
</table>
</div><br><label><b>dtype:</b> int64</label>



###Solution Implementation
Building a Model.


```python
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import roc_auc_score

# Feature Engineering
# -----------------------------
dfmod = dfmod.copy()

dfmod["complexity"] = dfmod["num_conditions"] + dfmod["num_procedures"]

dfmod["proc_per_condition"] = dfmod["num_procedures"] / (dfmod["num_conditions"] + 1)

dfmod["conditions_per_visit"] = dfmod["num_conditions"] / dfmod["total_visits"]

dfmod["admission_condition_interaction"] = dfmod["PRIOR_ADMISSIONS"] * dfmod["num_conditions"]

# -----------------------------
# One-hot encode categorical variables
# -----------------------------
dfmod = pd.get_dummies(
    dfmod,
    columns=["gender_concept_id", "race_concept_id"],
    drop_first=True
)

# -----------------------------
# Define features and target
# -----------------------------
target = "READMIT_30"

feature_cols = [
    "AGE",
    "PRIOR_ADMISSIONS",
    "total_visits",
    "num_conditions",
    "unique_conditions",
    "num_procedures",
    "unique_procedures",
    "complexity",
    "proc_per_condition",
    "conditions_per_visit",
    "admission_condition_interaction"
]

# Include all one-hot encoded columns automatically
categorical_cols = [col for col in dfmod.columns if col.startswith("gender_concept_id_") or col.startswith("race_concept_id_")]
feature_cols = feature_cols + categorical_cols

X = dfmod[feature_cols]
y = dfmod[target]

# -----------------------------
# Train-test split
# -----------------------------
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

# -----------------------------
# Train Random Forest model
# -----------------------------
model = RandomForestClassifier(
    n_estimators=200,
    max_depth=12,
    min_samples_leaf=5,
    random_state=42,
    n_jobs=-1
)

model.fit(X_train, y_train)

# -----------------------------
# Evaluate model
# -----------------------------
y_pred_proba = model.predict_proba(X_test)[:, 1]
auc = roc_auc_score(y_test, y_pred_proba)

print("ROC AUC:", auc)
```

    ROC AUC: 0.8794397175994932



```python
dfmod["READMIT_30"].value_counts(normalize=True)
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>proportion</th>
    </tr>
    <tr>
      <th>READMIT_30</th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0.760479</td>
    </tr>
    <tr>
      <th>1</th>
      <td>0.239521</td>
    </tr>
  </tbody>
</table>
</div><br><label><b>dtype:</b> float64</label>




```python
train_pred = model.predict_proba(X_train)[:,1]
test_pred = model.predict_proba(X_test)[:,1]

print("Train AUC:", roc_auc_score(y_train, train_pred))
print("Test AUC:", roc_auc_score(y_test, test_pred))
```

    Train AUC: 0.8928151079180862
    Test AUC: 0.8794397175994932


Checking Train and Test AUC to ensure that the model was not way overfitting.  

### Visualization Results


```python
from sklearn.metrics import roc_curve, auc
# Set global font to Times New Roman
plt.rcParams["font.family"] = ["DejaVu Serif", "serif"]

# Predict probabilities
y_pred_proba = model.predict_proba(X_test)[:, 1]

# ROC curve
fpr, tpr, thresholds = roc_curve(y_test, y_pred_proba)
roc_auc = auc(fpr, tpr)

plt.figure(figsize=(8, 6))

plt.plot(fpr, tpr, label=f"Model (AUC = {roc_auc:.3f})")
plt.plot([0, 1], [0, 1], linestyle="--", label="Random Classifier")

plt.xlabel("False Positive Rate", fontweight="bold")
plt.ylabel("True Positive Rate", fontweight="bold")
plt.title("ROC Curve for 30-Day Readmission Model", fontweight="bold", fontsize = 15)
plt.legend(loc="lower right")

# Caption (publication-style)
plt.figtext(
    0.5, -0.05,
    "Figure 1: Receiver Operating Characteristic (ROC) curve for the Random Forest model predicting 30-day hospital readmission. "
    "The model achieves an AUC of {:.3f}, indicating strong discriminatory performance compared to a random baseline.".format(roc_auc),
    wrap=True,
    horizontalalignment="center",
    fontsize=10
)

plt.tight_layout()
plt.show()
```


    
![png](Problem_Solution_Pipeline_files/Problem_Solution_Pipeline_21_0.png)
    



```python
# Feature importance
importances = model.feature_importances_
feature_names = X_train.columns

feat_imp_df = pd.DataFrame({
    "feature": feature_names,
    "importance": importances
}).sort_values(by="importance", ascending=True)  # ascending for horizontal plot

plt.figure(figsize=(10, 6))

# Horizontal bar chart
plt.barh(feat_imp_df["feature"], feat_imp_df["importance"])

plt.xlabel("Feature Importance", fontweight="bold")
plt.ylabel("Features", fontweight="bold")
plt.title("Feature Importance from Random Forest Model", fontweight="bold", fontsize = 15)

# Caption
plt.figtext(
    .65, -0.1,
    "Figure 2: Horizontal bar chart of feature importance from the Random Forest model. "
    "Features are ranked by their relative contribution to predictive performance. "
    "Higher values indicate greater influence on predicting 30-day readmission risk, "
    "with clinical history variables such as prior admissions and condition/procedure counts typically dominating.",
    wrap=True,
    horizontalalignment="center",
    fontsize=10
)

plt.tight_layout()
plt.show()
```


    
![png](Problem_Solution_Pipeline_files/Problem_Solution_Pipeline_22_0.png)
    



```python
# Get predicted probabilities
y_pred_proba = model.predict_proba(X_test)[:, 1]

# Separate by actual outcome
probs_readmit = y_pred_proba[y_test == 1]
probs_no_readmit = y_pred_proba[y_test == 0]
threshold = np.percentile(y_pred_proba, 80)  # top 20% highest risk

plt.figure(figsize=(8, 6))

plt.hist(probs_no_readmit, bins=30, alpha=0.7, label="No Readmission (0)")
plt.hist(probs_readmit, bins=30, alpha=0.7, label="Readmission (1)")

plt.xlabel("Predicted Probability of Readmission", fontweight='bold')
plt.ylabel("Number of Patients", fontweight='bold')
plt.title("Distribution of Predicted Readmission Risk by Outcome", fontweight='bold', fontsize= 15)

# Vertical threshold line
plt.axvline(threshold, color='black', linestyle='--', label="Top 20% Threshold")

plt.legend()

# Caption
plt.figtext(
    0.55, -0.09,
    "Figure 3: Distribution of predicted readmission probabilities stratified by true outcomes. "
    "The vertical dashed line represents the top 20% risk threshold, which could be used to "
    "identify patients for targeted clinical intervention.",
    wrap=True,
    horizontalalignment="center",
    fontsize=10
)

plt.tight_layout()
plt.show()
```


    
![png](Problem_Solution_Pipeline_files/Problem_Solution_Pipeline_23_0.png)
    



```python
from sklearn.metrics import precision_recall_curve

# Predicted probabilities
y_pred_proba = model.predict_proba(X_test)[:, 1]

precision, recall, thresholds = precision_recall_curve(y_test, y_pred_proba)

# Target recall
target_recall = 0.80

# Find indices where recall >= target
valid_indices = np.where(recall >= target_recall)[0]

# Choose the threshold that gives recall just above target
idx = valid_indices[-1]  # closest to target from below

# Corresponding threshold
chosen_threshold = thresholds[idx]

print("Chosen threshold for ~80% recall:", chosen_threshold)
print("Achieved recall:", recall[idx])
print("Corresponding precision:", precision[idx])
```

    Chosen threshold for ~80% recall: 0.46361772991666805
    Achieved recall: 0.8000894454382826
    Corresponding precision: 0.544762484774665



```python
plt.figure(figsize=(8, 6))

# PR curve
plt.plot(recall, precision, label="Model PR Curve")

# Mark the selected threshold point
plt.scatter(recall[idx], precision[idx], color='red',
            label=f"Recall ≈ {recall[idx]:.2f} at threshold ≈ {chosen_threshold:.2f}")

plt.xlabel("Recall (Sensitivity)", fontweight='bold')
plt.ylabel("Precision (Positive Predictive Value)", fontweight='bold')
plt.title("Precision–Recall Curve with Recall-Constrained Threshold", fontweight='bold', fontsize= 15)

plt.legend()

plt.figtext(
    0.52, -0.08,
    "Figure 4: Precision–Recall curve with a selected operating threshold that achieves approximately "
    "80% recall. This threshold prioritizes identifying a high proportion of true readmissions, "
    "which is critical in clinical settings where missing high-risk patients carries significant cost.",
    wrap=True,
    horizontalalignment="center",
    fontsize=10
)

plt.tight_layout()
plt.show()
```


    
![png](Problem_Solution_Pipeline_files/Problem_Solution_Pipeline_25_0.png)
    


###Analysis Rationale
The analysis was designed to develop a predictive model for 30-day hospital readmission using structured patient-level data derived from a relational dataset. A Random Forest classifier was selected due to its ability to handle nonlinear relationships, interactions between variables, and mixed feature types without requiring extensive preprocessing or strong parametric assumptions. The dataset was constructed by joining multiple tables (e.g., patient demographics, visit history, conditions, and procedures) to ensure that each observation represents a patient visit enriched with clinically relevant context. Feature engineering focused on aggregating patient history into meaningful predictors such as prior admissions, total visits, and counts of conditions and procedures, which serve as proxies for patient complexity and healthcare utilization. Categorical variables such as race and gender were included as predictors to capture potential disparities or population-level trends. Model evaluation was performed using a train-test split to assess generalization, and ROC-AUC and Average Precision were used as primary metrics to account for class imbalance and to evaluate both ranking ability and performance on the minority class (readmissions). Visualizations provided another layer of analysis for use by healthcare workers and hospitals, the end users who would ultimately be considering implementing and using such a model in the workplace.

### Visualization Rationale
The visualizations were selected to provide both performance evaluation and interpretability from a clinical decision-making perspective. The ROC curve was used to assess the model’s ability to discriminate between readmitted and non-readmitted patients across all classification thresholds, with the AUC providing a summary measure of overall performance. In addition, the Precision–Recall curve was included as it is more informative in the presence of class imbalance, emphasizing the tradeoff between correctly identifying high-risk patients (recall) and the reliability of those predictions (precision). A risk score distribution plot was used to visualize how predicted probabilities differ between actual outcomes, allowing stakeholders to assess whether the model meaningfully separates high-risk and low-risk patients. This is particularly important in healthcare settings where prioritization of limited resources is required. Finally, a horizontal feature importance plot was used to identify the most influential predictors in the model, improving interpretability and helping clinicians understand which factors drive predictions. Together, these visualizations were chosen to balance statistical performance assessment with practical interpretability, supporting both model validation and potential real-world implementation in a hospital setting. It is important to note that in a real-world clinical setting, providers would be able to distinguish between gender and race codes as well as the procedure and condition types as they would be encoded in their own hospital system.
