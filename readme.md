# Štatistika návštevnosti v budove Bubenská 1 #
### Diana Líšková ###

## Pôvod a štruktúra dát ##
Dáta som získala z knihy návštev, ktorá sa nachádza na recepcii budovy Bubenská 1, Praha, kde pracujem. Ich časové rozmedzie je 6 mesiacov - od februára 2022 do konca júla 2022. Budova slúži ako business centrum spájajúce niekoľko spoločností. Má viacero vchodov, a teda aj recepcii a recepcia, kde pracujem slúži konkrétne pre firmy Kantar a Wolt. 
Dáta sú tvorené riadkami obsahujúcimi údaje jednotlivých vstupov do budovy a to konkrétne dátum, deň v týždni, čas a firmu, ku ktorej sa návštevník hlásil.


```python
import numpy as np
import pandas as pd
import scipy.stats as stats
import matplotlib
import matplotlib.pyplot as plt
matplotlib.style.use('ggplot')
import math
import os
from sklearn import linear_model
from pandas.api.types import CategoricalDtype
```

## Načítanie a ukážka dát ##


```python
sample = pd.read_excel('/Users/dialiskova/Desktop/VS code/python/statistika/input/navstevnost.xlsx', parse_dates=[0])

sample_size = sample["datum"].count()
sample_indexes = sample.dtypes[sample.dtypes == "object"].index

sample.head(7)
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
      <th>datum</th>
      <th>den</th>
      <th>čas</th>
      <th>firma</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2022-07-28</td>
      <td>štv</td>
      <td>10:41:00</td>
      <td>kantar</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2022-07-28</td>
      <td>štv</td>
      <td>10:56:00</td>
      <td>kantar</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022-07-28</td>
      <td>štv</td>
      <td>13:27:00</td>
      <td>kantar</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2022-07-28</td>
      <td>štv</td>
      <td>17:49:00</td>
      <td>kantar</td>
    </tr>
    <tr>
      <th>4</th>
      <td>2022-07-27</td>
      <td>str</td>
      <td>12:18:00</td>
      <td>kantar</td>
    </tr>
    <tr>
      <th>5</th>
      <td>2022-07-27</td>
      <td>str</td>
      <td>13:56:00</td>
      <td>wolt</td>
    </tr>
    <tr>
      <th>6</th>
      <td>2022-07-27</td>
      <td>str</td>
      <td>14:21:00</td>
      <td>wolt</td>
    </tr>
  </tbody>
</table>
</div>



## Konfidenčné intervaly ##

Chceme zistiť, ktorý z dní v týždni má najväčšiu návštevnosť. Využijeme obe možnosti - ako prvé si vytvoríme crosstab a na jej základe histogram, kde budeme hľadný deň vidieť. 

Pre vizuálne potešenie oka si dni v týždni zoradíme pomocou knižnice pandas.api.types. 


```python
df = sample
cats = ["pon", "ut", "str", "štv", "pia"]
df['den'] = df['den'].astype('category', CategoricalDtype(categories=cats))
cat_type = CategoricalDtype(categories=cats, ordered=True)
df['den'] = df['den'].astype(cat_type)

days_by_business = pd.crosstab(index=df["den"], columns=df["firma"], rownames=["den"])
days_by_business
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
      <th>firma</th>
      <th>kantar</th>
      <th>wolt</th>
    </tr>
    <tr>
      <th>den</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>pon</th>
      <td>67</td>
      <td>29</td>
    </tr>
    <tr>
      <th>ut</th>
      <td>68</td>
      <td>36</td>
    </tr>
    <tr>
      <th>str</th>
      <td>79</td>
      <td>55</td>
    </tr>
    <tr>
      <th>štv</th>
      <td>51</td>
      <td>54</td>
    </tr>
    <tr>
      <th>pia</th>
      <td>34</td>
      <td>17</td>
    </tr>
  </tbody>
</table>
</div>



Vizualizujeme počet návštev na Bubenskej 1 v závislosti od dňa v týždni a firmy pomocou histogramu. 


```python
days_by_business.plot.bar(rot=0)
```




    <AxesSubplot:xlabel='den'>




    
[output_9_1](https://user-images.githubusercontent.com/110021631/186895678-e05b31ab-4354-4a41-8368-c3ca5781eb3c.png)

    


Za druhé: vytvoríme **konfidenčné intervaly** pre jednotlivé dni na základe ich frekvencie.


```python
days_by_business = pd.crosstab(index=sample["firma"], columns=sample["den"], margins=True)
# pre konfidenčné intervaly potrebujeme, aby boli v crosstab-e margins nastavené na True

for day in ["pon", "ut", "str", "štv", "pia"]:
    day_fqc = days_by_business.loc["All", day]/days_by_business.loc["All", "All"]
    z_critical = stats.norm.ppf(0.975)
    margin_of_error = z_critical * math.sqrt((day_fqc * (1-day_fqc))/sample_size)
    confidence_interval = (day_fqc - margin_of_error, day_fqc + margin_of_error)
    print(day, ":", confidence_interval, end="\n")
```

    pon : (0.16077545911793098, 0.23106127557594655)
    ut : (0.17604025299719978, 0.24844954292116755)
    str : (0.23400263734861507, 0.31293613816158905)
    štv : (0.1779545776419733, 0.25061685092945524)
    pia : (0.07704383464991696, 0.1311194306562055)


Na základe analýzy frekvencie jednotlivých dní v týždni z knihy návštev môžeme tvrdiť, že najviac návštevníkov prichádza v stredu, čo potvrdzuje aj histogram.

## Jednovýberový t-test ##
Jednovýberový t-test kontroluje, či sa výberový priemer líši od toho populačného. Otestujeme, či sa priemerný počet návštev za hodinu v stredu líši od priemeru počtu návštev za hodinu v obecnosti. 

**Nulová hypotéza:** vzorka dát zo stredy má rovnaké rozdelenie ako populačná. 

Pre lepšiu prehľadnosť som si napísala jednoduchú funkciu pre zaradenie časov do časových intervalov po 15tich minútach, s ktorými budem následne pracovať.


```python
def timeIntervals(inp):
    intervals = dict()

    for i in range(8, 18):
        if i < 10:
            h = "0"+str(i)
        else:
            h = str(i) 
        for j in ["00", "15", "30", "45"]:
            intervals[str(h+"."+j)] = 0
    intervals["18.00"] = 0

    for line in inp:
        h, m, s = line.split(":")
        m = int(m) 

        if m >= 0 and m < 15:
            time = h + ".00"
        elif m >= 15 and m < 30:
            time = h + ".15"
        elif m >= 30 and m < 45:
            time = h + ".30"
        elif m >= 45 and m < 60:
            time = h + ".45"
        
        intervals[time] +=1

    intervals = dict(sorted(intervals.items(), key=lambda item:item[0]))
    return list(intervals.values())

intervaly = []  # vytvorila som zoznam intervalov, pretože potrebujem všetky - aj tie, kde by bol počet návštev = 0
for i in range(8, 18):
    if i < 10:
        hod = "0"+str(i)
    else:
        hod = str(i) 
    for j in ["00", "15", "30", "45"]:
        intervaly.append(str(hod+"."+j))
intervaly.append("18.00")

frekvencie_dni = dict()
for day in ["pon", "ut", "str", "štv", "pia"]:
    dni = sample.loc[np.where(sample["den"] == day)]
    pocty = timeIntervals(dni["čas"].astype(str))
    frekvencie_dni[day] = pocty

intervals_by_days = pd.DataFrame(frekvencie_dni, index=intervaly) # zoznam použijem ako index
intervals_by_days
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
      <th>pon</th>
      <th>ut</th>
      <th>str</th>
      <th>štv</th>
      <th>pia</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>08.00</th>
      <td>0</td>
      <td>1</td>
      <td>8</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>08.15</th>
      <td>3</td>
      <td>2</td>
      <td>3</td>
      <td>8</td>
      <td>0</td>
    </tr>
    <tr>
      <th>08.30</th>
      <td>1</td>
      <td>1</td>
      <td>10</td>
      <td>11</td>
      <td>4</td>
    </tr>
    <tr>
      <th>08.45</th>
      <td>20</td>
      <td>10</td>
      <td>22</td>
      <td>10</td>
      <td>12</td>
    </tr>
    <tr>
      <th>09.00</th>
      <td>7</td>
      <td>11</td>
      <td>10</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <th>09.15</th>
      <td>6</td>
      <td>5</td>
      <td>2</td>
      <td>6</td>
      <td>4</td>
    </tr>
    <tr>
      <th>09.30</th>
      <td>5</td>
      <td>2</td>
      <td>5</td>
      <td>8</td>
      <td>1</td>
    </tr>
    <tr>
      <th>09.45</th>
      <td>7</td>
      <td>3</td>
      <td>8</td>
      <td>6</td>
      <td>3</td>
    </tr>
    <tr>
      <th>10.00</th>
      <td>0</td>
      <td>4</td>
      <td>2</td>
      <td>6</td>
      <td>2</td>
    </tr>
    <tr>
      <th>10.15</th>
      <td>3</td>
      <td>5</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>10.30</th>
      <td>2</td>
      <td>1</td>
      <td>3</td>
      <td>4</td>
      <td>1</td>
    </tr>
    <tr>
      <th>10.45</th>
      <td>3</td>
      <td>7</td>
      <td>3</td>
      <td>3</td>
      <td>2</td>
    </tr>
    <tr>
      <th>11.00</th>
      <td>2</td>
      <td>2</td>
      <td>3</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11.15</th>
      <td>4</td>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
    </tr>
    <tr>
      <th>11.30</th>
      <td>2</td>
      <td>3</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>11.45</th>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>12.00</th>
      <td>1</td>
      <td>2</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12.15</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12.30</th>
      <td>1</td>
      <td>0</td>
      <td>3</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>12.45</th>
      <td>1</td>
      <td>4</td>
      <td>3</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13.00</th>
      <td>3</td>
      <td>0</td>
      <td>8</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13.15</th>
      <td>1</td>
      <td>4</td>
      <td>4</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>13.30</th>
      <td>0</td>
      <td>4</td>
      <td>2</td>
      <td>2</td>
      <td>1</td>
    </tr>
    <tr>
      <th>13.45</th>
      <td>3</td>
      <td>2</td>
      <td>6</td>
      <td>2</td>
      <td>4</td>
    </tr>
    <tr>
      <th>14.00</th>
      <td>0</td>
      <td>0</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>14.15</th>
      <td>0</td>
      <td>2</td>
      <td>4</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>14.30</th>
      <td>1</td>
      <td>3</td>
      <td>0</td>
      <td>5</td>
      <td>1</td>
    </tr>
    <tr>
      <th>14.45</th>
      <td>4</td>
      <td>5</td>
      <td>1</td>
      <td>1</td>
      <td>2</td>
    </tr>
    <tr>
      <th>15.00</th>
      <td>4</td>
      <td>3</td>
      <td>0</td>
      <td>0</td>
      <td>2</td>
    </tr>
    <tr>
      <th>15.15</th>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>15.30</th>
      <td>2</td>
      <td>4</td>
      <td>6</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>15.45</th>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>1</td>
      <td>1</td>
    </tr>
    <tr>
      <th>16.00</th>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>2</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16.15</th>
      <td>2</td>
      <td>2</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>16.30</th>
      <td>2</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>16.45</th>
      <td>3</td>
      <td>6</td>
      <td>3</td>
      <td>0</td>
      <td>1</td>
    </tr>
    <tr>
      <th>17.00</th>
      <td>0</td>
      <td>0</td>
      <td>2</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>17.15</th>
      <td>0</td>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>0</td>
    </tr>
    <tr>
      <th>17.30</th>
      <td>1</td>
      <td>1</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>17.45</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
    <tr>
      <th>18.00</th>
      <td>0</td>
      <td>0</td>
      <td>0</td>
      <td>1</td>
      <td>0</td>
    </tr>
  </tbody>
</table>
</div>



Inicializujeme premenné pre stredajšiu strednú hodnotu, počet návštev zapísaných v stredu, stupne volnosti a hladinu významnosti.


```python
streda_hour_mean = intervals_by_days["str"].mean()
streda_size = intervals_by_days["str"].count()
pop_mean = intervals_by_days.mean().sum()/5

degrees_of_freedom = streda_size - 1
significance_level = 0.05
```

Vykonáme t-test pri 95 percentnom konfidenčnom leveli. 


```python
stats.ttest_1samp(a=intervals_by_days["str"], # vzorkové dáta
                  popmean=pop_mean)           # populačná stredná hodnota
```




    Ttest_1sampResult(statistic=1.3716400877961026, pvalue=0.17782079492260977)



Výsledok testu zobrazuje, že testová štatistika t je rovná približne 1.37. Táto testová štatistika nám ukazuje, ako veľmi sa stredná hodnota vzorky zo stredy odchyľuje od nulovej hypotézy.
Môžeme pozorovať, že p-hodnota je 0,17 (17,78%) a teda je vyššia ako hladina významnosti 5%, **nulovú hypotézu sa nám preto nepodarilo zamietnuť**.

Nakoniec spočítame konfidenčný interval.


```python
sigma = intervals_by_days["str"].std()/math.sqrt(streda_size)      # štandardná odchylka vzorku / veľkosť vzorky
stats.t.interval(0.95,                         # konfidenčný level
                 df=degrees_of_freedom,        # stupne volnosti
                 loc=streda_hour_mean,         # stredná hodnota vzorky
                 scale=sigma)                  # odhad št. odchylky
```




    (1.9745110334240492, 4.562074332429609)



Môžeme si všimnúť, že doň spadá aj populačná stredná hodnota (2.39), čo slúži ako uistenie, že nulovú hypotézu naozaj nemôžeme zamietnuť.

## Lineárna regresia ##

Lineárna regresia je technika predikčného modelovania na predpovedanie odozvy na základe jednej alebo viacerých premenných. Model je navrhnutý tak, aby vyhovoval čiare, ktorá minimalizuje štvorcové rozdiely (nazývané aj chyby alebo zvyšky).

V nšej lineárnej regresii budeme pozorovať, ako sa zmení celkový počet návštev za týždeň vzhľadom k návštevnosti v stredu. Budeme používať scikit-learn knižnicu a jej funkciu pre lineárnu regresiu.


```python
intervals_by_days["total"] = intervals_by_days.sum(axis=1)         # pridáme stĺpec so sumou pre každý čas 

regression_model = linear_model.LinearRegression()
regression_model.fit(X = pd.DataFrame(intervals_by_days["str"]), 
                     y = intervals_by_days["total"])
print(regression_model.intercept_)
print(regression_model.coef_)
```

    2.9906365681933664
    [2.7416709]


Výstup zobrazuje priesečník modelu a koeficienty použité na vytvorenie najvhodnejšej čiary. Model zodpovedá čiare: **total = 37,2851 - 5,3445 * str**


```python
regression_model.score(X = pd.DataFrame(intervals_by_days["str"]), 
                       y = intervals_by_days["total"])
```




    0.7942934641130857



Výstupom funkciu score je hodnota, ktorá sa pohybuje od 0 do 1 a popisuje podiel rozptylu vzhľadom k odozve (premennej total). V tomto prípade môžeme vidieť, že stredajšia návštevnosť **tvorí zhruba 79% rozptylu v celkovej návštevnosti.**

Následne vytvoríme predpoveď - model s regresívnou líniou. 


```python
prediction = regression_model.predict(X = pd.DataFrame(intervals_by_days["str"]))  # predpovedané hodnoty

intervals_by_days.plot(kind="scatter",
                       x="str",
                       y="total",
                       figsize=(9,9),
                       color="black")

plt.plot(intervals_by_days["str"],      
         prediction,                   
         color="red");
```

![output_28_0](https://user-images.githubusercontent.com/110021631/186895769-e9ed59e3-f7c1-406f-a804-4849afaa9c2f.png)

    

    

