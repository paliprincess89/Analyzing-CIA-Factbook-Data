# Analyzing CIA Factbook Data Using SQL

### In this guided project, we will be analyzing data from the CIA World Factbook, which contains demographics and statistics about all countries on Earth.

Some of the data provided in this factbook is as follows:

* population — the global population.
* population_growth — the annual population growth rate, as a percentage.
* area — the total land and water area.

## Connecting the database file


```python
%%capture
%load_ext sql
%sql sqlite:///factbook.db
```




    'Connected: None@factbook.db'




```sql
%%sql
SELECT *
  FROM sqlite_master
 WHERE type='table';
```

    Done.





<table>
    <tr>
        <th>type</th>
        <th>name</th>
        <th>tbl_name</th>
        <th>rootpage</th>
        <th>sql</th>
    </tr>
    <tr>
        <td>table</td>
        <td>sqlite_sequence</td>
        <td>sqlite_sequence</td>
        <td>3</td>
        <td>CREATE TABLE sqlite_sequence(name,seq)</td>
    </tr>
    <tr>
        <td>table</td>
        <td>facts</td>
        <td>facts</td>
        <td>47</td>
        <td>CREATE TABLE &quot;facts&quot; (&quot;id&quot; INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL, &quot;code&quot; varchar(255) NOT NULL, &quot;name&quot; varchar(255) NOT NULL, &quot;area&quot; integer, &quot;area_land&quot; integer, &quot;area_water&quot; integer, &quot;population&quot; integer, &quot;population_growth&quot; float, &quot;birth_rate&quot; float, &quot;death_rate&quot; float, &quot;migration_rate&quot; float)</td>
    </tr>
</table>



### First 5 rows of the 'facts' table in the database


```sql
%%sql
SELECT *
  FROM facts
 LIMIT 5;
```

    Done.





<table>
    <tr>
        <th>id</th>
        <th>code</th>
        <th>name</th>
        <th>area</th>
        <th>area_land</th>
        <th>area_water</th>
        <th>population</th>
        <th>population_growth</th>
        <th>birth_rate</th>
        <th>death_rate</th>
        <th>migration_rate</th>
    </tr>
    <tr>
        <td>1</td>
        <td>af</td>
        <td>Afghanistan</td>
        <td>652230</td>
        <td>652230</td>
        <td>0</td>
        <td>32564342</td>
        <td>2.32</td>
        <td>38.57</td>
        <td>13.89</td>
        <td>1.51</td>
    </tr>
    <tr>
        <td>2</td>
        <td>al</td>
        <td>Albania</td>
        <td>28748</td>
        <td>27398</td>
        <td>1350</td>
        <td>3029278</td>
        <td>0.3</td>
        <td>12.92</td>
        <td>6.58</td>
        <td>3.3</td>
    </tr>
    <tr>
        <td>3</td>
        <td>ag</td>
        <td>Algeria</td>
        <td>2381741</td>
        <td>2381741</td>
        <td>0</td>
        <td>39542166</td>
        <td>1.84</td>
        <td>23.67</td>
        <td>4.31</td>
        <td>0.92</td>
    </tr>
    <tr>
        <td>4</td>
        <td>an</td>
        <td>Andorra</td>
        <td>468</td>
        <td>468</td>
        <td>0</td>
        <td>85580</td>
        <td>0.12</td>
        <td>8.13</td>
        <td>6.96</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>5</td>
        <td>ao</td>
        <td>Angola</td>
        <td>1246700</td>
        <td>1246700</td>
        <td>0</td>
        <td>19625353</td>
        <td>2.78</td>
        <td>38.78</td>
        <td>11.49</td>
        <td>0.46</td>
    </tr>
</table>



### Here are the descriptions for some of the columns:

* name — the name of the country.
* area— the country's total area (both land and water).
* area_land — the country's land area in square kilometers.
* area_water — the country's waterarea in square kilometers.
* population — the country's population.
* population_growth— the country's population growth as a percentage.
* birth_rate — the country's birth rate, or the number of births per year per 1,000 people.
* death_rate — the country's death rate, or the number of death per year per 1,000 people.

### Calculating some summary statistics and looking for any outlier countries


```sql
%%sql
SELECT MIN(population) AS min_pop, 
       MAX(population) AS max_pop, 
       MIN(population_growth) AS min_pop_growth, 
       MAX(population_growth) AS max_pop_growth
  FROM facts;
    
```

    Done.





<table>
    <tr>
        <th>min_pop</th>
        <th>max_pop</th>
        <th>min_pop_growth</th>
        <th>max_pop_growth</th>
    </tr>
    <tr>
        <td>0</td>
        <td>7256490011</td>
        <td>0.0</td>
        <td>4.02</td>
    </tr>
</table>




```sql
%%sql
SELECT name, population
  FROM facts
 WHERE population IN (SELECT MIN(population) FROM facts);
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>population</th>
    </tr>
    <tr>
        <td>Antarctica</td>
        <td>0</td>
    </tr>
</table>




```sql
%%sql
SELECT name, population
  FROM facts
 WHERE population IN (SELECT MAX(population) FROM facts);
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>population</th>
    </tr>
    <tr>
        <td>World</td>
        <td>7256490011</td>
    </tr>
</table>



<b>We can see that the data includes a row for the whole World called 'World' and that Antarctica has a population of 0 which are both skewing our country population data. We'll rerun the same stats without these rows below.


```sql
%%sql
SELECT MIN(population) AS min_pop, 
       MAX(population) AS max_pop, 
       MIN(population_growth) AS min_pop_growth, 
       MAX(population_growth) AS max_pop_growth
  FROM facts
 WHERE name <> 'World' 
   AND name <> 'Antarctica';
```

    Done.





<table>
    <tr>
        <th>min_pop</th>
        <th>max_pop</th>
        <th>min_pop_growth</th>
        <th>max_pop_growth</th>
    </tr>
    <tr>
        <td>48</td>
        <td>1367485388</td>
        <td>0.0</td>
        <td>4.02</td>
    </tr>
</table>




```sql
%%sql
SELECT name, population
  FROM facts
 WHERE population IN (SELECT MIN(population) FROM facts
                       WHERE name <> 'World'
                         AND name <> 'Antarctica');
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>population</th>
    </tr>
    <tr>
        <td>Pitcairn Islands</td>
        <td>48</td>
    </tr>
</table>



From the above query, we can see that the British territory of Pitcairn Islands has the lowest population of any territory with 48 people, but it is technically not a country.


```sql
%%sql
SELECT name, population
  FROM facts
 WHERE population IN (SELECT MIN(population) FROM facts
                       WHERE name <> 'World'
                         AND name <> 'Antarctica'
                         AND name NOT LIKE '% Islands'
                         AND name NOT LIKE '% City)');
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>population</th>
    </tr>
    <tr>
        <td>Niue</td>
        <td>1190</td>
    </tr>
</table>



In order to exclude Island territories and cities, we excluded them in our SQL query filter and returned Niue as the Country with the lowest population: 1190.


```sql
%%sql
SELECT ROUND(AVG(population), 2) AS avg_pop, 
       ROUND(AVG(area), 2) AS avg_area
  FROM facts;
```

    Done.





<table>
    <tr>
        <th>avg_pop</th>
        <th>avg_area</th>
    </tr>
    <tr>
        <td>62094928.32</td>
        <td>555093.55</td>
    </tr>
</table>




```sql
%%sql
SELECT *
  FROM facts
 WHERE population > (SELECT ROUND(AVG(population), 2) 
                       FROM facts
                      WHERE name <> 'World'
                        AND name <> 'Antarctica')
   AND area < (SELECT ROUND(AVG(area), 2)
                 FROM facts
                WHERE name <> 'World'
                  AND name <> 'Antarctica')
 ORDER BY population DESC;
```

    Done.





<table>
    <tr>
        <th>id</th>
        <th>code</th>
        <th>name</th>
        <th>area</th>
        <th>area_land</th>
        <th>area_water</th>
        <th>population</th>
        <th>population_growth</th>
        <th>birth_rate</th>
        <th>death_rate</th>
        <th>migration_rate</th>
    </tr>
    <tr>
        <td>14</td>
        <td>bg</td>
        <td>Bangladesh</td>
        <td>148460</td>
        <td>130170</td>
        <td>18290</td>
        <td>168957745</td>
        <td>1.6</td>
        <td>21.14</td>
        <td>5.61</td>
        <td>0.46</td>
    </tr>
    <tr>
        <td>85</td>
        <td>ja</td>
        <td>Japan</td>
        <td>377915</td>
        <td>364485</td>
        <td>13430</td>
        <td>126919659</td>
        <td>0.16</td>
        <td>7.93</td>
        <td>9.51</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>138</td>
        <td>rp</td>
        <td>Philippines</td>
        <td>300000</td>
        <td>298170</td>
        <td>1830</td>
        <td>100998376</td>
        <td>1.61</td>
        <td>24.27</td>
        <td>6.11</td>
        <td>2.09</td>
    </tr>
    <tr>
        <td>192</td>
        <td>vm</td>
        <td>Vietnam</td>
        <td>331210</td>
        <td>310070</td>
        <td>21140</td>
        <td>94348835</td>
        <td>0.97</td>
        <td>15.96</td>
        <td>5.93</td>
        <td>0.3</td>
    </tr>
    <tr>
        <td>65</td>
        <td>gm</td>
        <td>Germany</td>
        <td>357022</td>
        <td>348672</td>
        <td>8350</td>
        <td>80854408</td>
        <td>0.17</td>
        <td>8.47</td>
        <td>11.42</td>
        <td>1.24</td>
    </tr>
    <tr>
        <td>173</td>
        <td>th</td>
        <td>Thailand</td>
        <td>513120</td>
        <td>510890</td>
        <td>2230</td>
        <td>67976405</td>
        <td>0.34</td>
        <td>11.19</td>
        <td>7.8</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>185</td>
        <td>uk</td>
        <td>United Kingdom</td>
        <td>243610</td>
        <td>241930</td>
        <td>1680</td>
        <td>64088222</td>
        <td>0.54</td>
        <td>12.17</td>
        <td>9.35</td>
        <td>2.54</td>
    </tr>
    <tr>
        <td>83</td>
        <td>it</td>
        <td>Italy</td>
        <td>301340</td>
        <td>294140</td>
        <td>7200</td>
        <td>61855120</td>
        <td>0.27</td>
        <td>8.74</td>
        <td>10.19</td>
        <td>4.1</td>
    </tr>
    <tr>
        <td>91</td>
        <td>ks</td>
        <td>Korea, South</td>
        <td>99720</td>
        <td>96920</td>
        <td>2800</td>
        <td>49115196</td>
        <td>0.14</td>
        <td>8.19</td>
        <td>6.75</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>163</td>
        <td>sp</td>
        <td>Spain</td>
        <td>505370</td>
        <td>498980</td>
        <td>6390</td>
        <td>48146134</td>
        <td>0.89</td>
        <td>9.64</td>
        <td>9.04</td>
        <td>8.31</td>
    </tr>
    <tr>
        <td>139</td>
        <td>pl</td>
        <td>Poland</td>
        <td>312685</td>
        <td>304255</td>
        <td>8430</td>
        <td>38562189</td>
        <td>0.09</td>
        <td>9.74</td>
        <td>10.19</td>
        <td>0.46</td>
    </tr>
    <tr>
        <td>182</td>
        <td>ug</td>
        <td>Uganda</td>
        <td>241038</td>
        <td>197100</td>
        <td>43938</td>
        <td>37101745</td>
        <td>3.24</td>
        <td>43.79</td>
        <td>10.69</td>
        <td>0.74</td>
    </tr>
    <tr>
        <td>80</td>
        <td>iz</td>
        <td>Iraq</td>
        <td>438317</td>
        <td>437367</td>
        <td>950</td>
        <td>37056169</td>
        <td>2.93</td>
        <td>31.45</td>
        <td>3.77</td>
        <td>1.62</td>
    </tr>
    <tr>
        <td>120</td>
        <td>mo</td>
        <td>Morocco</td>
        <td>446550</td>
        <td>446300</td>
        <td>250</td>
        <td>33322699</td>
        <td>1.0</td>
        <td>18.2</td>
        <td>4.81</td>
        <td>3.36</td>
    </tr>
</table>



### Above we can see there are 14 countries that have dense populations in below average areas.

### We will answer the following questions below:
* Which country has the most people? Which country has the highest growth rate?
* Which countries have the highest ratios of water to land? Which countries have more water than land?
* Which countries will add the most people to their populations next year?
* Which countries have a higher death rate than birth rate?
* Which countries have the highest population/area ratio, and how does it compare to list we found in the previous screen?

<b/>Which country has the most people?


```sql
%%sql
SELECT *
  FROM facts
 Where name <> 'World'
   AND name <> 'Antarctica'
   AND name NOT LIKE '% Islands'
   AND name NOT LIKE '% City)'
 ORDER BY population DESC
 LIMIT 1;
```

    Done.





<table>
    <tr>
        <th>id</th>
        <th>code</th>
        <th>name</th>
        <th>area</th>
        <th>area_land</th>
        <th>area_water</th>
        <th>population</th>
        <th>population_growth</th>
        <th>birth_rate</th>
        <th>death_rate</th>
        <th>migration_rate</th>
    </tr>
    <tr>
        <td>37</td>
        <td>ch</td>
        <td>China</td>
        <td>9596960</td>
        <td>9326410</td>
        <td>270550</td>
        <td>1367485388</td>
        <td>0.45</td>
        <td>12.49</td>
        <td>7.53</td>
        <td>0.44</td>
    </tr>
</table>



<b>Which country has the highest growth rate?


```sql
%%sql
SELECT *
  FROM facts
 WHERE name <> 'World'
   AND name <> 'Antarctica'
 ORDER BY population_growth DESC
 LIMIT 1;
```

    Done.





<table>
    <tr>
        <th>id</th>
        <th>code</th>
        <th>name</th>
        <th>area</th>
        <th>area_land</th>
        <th>area_water</th>
        <th>population</th>
        <th>population_growth</th>
        <th>birth_rate</th>
        <th>death_rate</th>
        <th>migration_rate</th>
    </tr>
    <tr>
        <td>162</td>
        <td>od</td>
        <td>South Sudan</td>
        <td>644329</td>
        <td>None</td>
        <td>None</td>
        <td>12042910</td>
        <td>4.02</td>
        <td>36.91</td>
        <td>8.18</td>
        <td>11.47</td>
    </tr>
</table>



South Sudan has the highest growth rate at 4.02%.

Let's see which country has the lowest growth rate below.

<b>Which country has the lowest growth rate?


```sql
%%sql
SELECT *
  FROM facts
 WHERE name <> 'World'
   AND name <> 'Antarctica'
   AND name NOT LIKE '% Islands'
   AND name NOT LIKE '% City)'
 ORDER BY population_growth
 LIMIT 1;
```

    Done.





<table>
    <tr>
        <th>id</th>
        <th>code</th>
        <th>name</th>
        <th>area</th>
        <th>area_land</th>
        <th>area_water</th>
        <th>population</th>
        <th>population_growth</th>
        <th>birth_rate</th>
        <th>death_rate</th>
        <th>migration_rate</th>
    </tr>
    <tr>
        <td>92</td>
        <td>kv</td>
        <td>Kosovo</td>
        <td>10887</td>
        <td>10887</td>
        <td>0</td>
        <td>1870981</td>
        <td>None</td>
        <td>None</td>
        <td>None</td>
        <td>None</td>
    </tr>
</table>



From the above query, Kosovo has the lowest growth rate but that seems to be because it is missing data. Let's run this again fitering out countries with 'None' values.


```sql
%%sql
SELECT *
  FROM facts
 WHERE name <> 'World'
   AND name <> 'Antarctica'
   AND name NOT LIKE '% Islands'
   AND name NOT LIKE '% City)'
   AND population_growth IS NOT NULL
 ORDER BY population_growth
 LIMIT 1;
```

    Done.





<table>
    <tr>
        <th>id</th>
        <th>code</th>
        <th>name</th>
        <th>area</th>
        <th>area_land</th>
        <th>area_water</th>
        <th>population</th>
        <th>population_growth</th>
        <th>birth_rate</th>
        <th>death_rate</th>
        <th>migration_rate</th>
    </tr>
    <tr>
        <td>207</td>
        <td>gl</td>
        <td>Greenland</td>
        <td>2166086</td>
        <td>2166086</td>
        <td>None</td>
        <td>57733</td>
        <td>0.0</td>
        <td>14.48</td>
        <td>8.49</td>
        <td>5.98</td>
    </tr>
</table>



Now we can see that Greenland has the lowest population growth which is actually 0.0 and not NULL. After a quick Google search on Greenland, we found that it is a territory of Denmark, so again not technically an autonomous country. Let's exclude it from the query and see if we can find the lowest growth rate for autonomous countries.


```sql
%%sql
SELECT *
  FROM facts
 WHERE name <> 'World'
   AND name <> 'Antarctica'
   AND name <> 'Greenland' 
   AND name NOT LIKE '% Islands'
   AND name NOT LIKE '% City)'
   AND population_growth IS NOT NULL
 ORDER BY population_growth
 LIMIT 1;
```

    Done.





<table>
    <tr>
        <th>id</th>
        <th>code</th>
        <th>name</th>
        <th>area</th>
        <th>area_land</th>
        <th>area_water</th>
        <th>population</th>
        <th>population_growth</th>
        <th>birth_rate</th>
        <th>death_rate</th>
        <th>migration_rate</th>
    </tr>
    <tr>
        <td>67</td>
        <td>gr</td>
        <td>Greece</td>
        <td>131957</td>
        <td>130647</td>
        <td>1310</td>
        <td>10775643</td>
        <td>0.01</td>
        <td>8.66</td>
        <td>11.09</td>
        <td>2.32</td>
    </tr>
</table>



Finally we've found the country with the lowest population growth rate, which is 0.01: <b>Greece.

<b>Which countries have the highest ratios of water to land?</b>


```sql
%%sql
SELECT *, ROUND((area_water / area_land)*100, 2) AS water_land_ratio
  FROM facts
 WHERE name <> 'World'
   AND name <> 'Antarctica'
   AND name <> 'Greenland' 
   AND name NOT LIKE '% Islands'
   AND name NOT LIKE '% City)'
   AND name NOT LIKE '% territory'
   AND area_land IS NOT NULL
   AND area_water IS NOT NULL
 ORDER BY water_land_ratio DESC
 LIMIT 20;
```

    Done.





<table>
    <tr>
        <th>id</th>
        <th>code</th>
        <th>name</th>
        <th>area</th>
        <th>area_land</th>
        <th>area_water</th>
        <th>population</th>
        <th>population_growth</th>
        <th>birth_rate</th>
        <th>death_rate</th>
        <th>migration_rate</th>
        <th>water_land_ratio</th>
    </tr>
    <tr>
        <td>1</td>
        <td>af</td>
        <td>Afghanistan</td>
        <td>652230</td>
        <td>652230</td>
        <td>0</td>
        <td>32564342</td>
        <td>2.32</td>
        <td>38.57</td>
        <td>13.89</td>
        <td>1.51</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>2</td>
        <td>al</td>
        <td>Albania</td>
        <td>28748</td>
        <td>27398</td>
        <td>1350</td>
        <td>3029278</td>
        <td>0.3</td>
        <td>12.92</td>
        <td>6.58</td>
        <td>3.3</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>3</td>
        <td>ag</td>
        <td>Algeria</td>
        <td>2381741</td>
        <td>2381741</td>
        <td>0</td>
        <td>39542166</td>
        <td>1.84</td>
        <td>23.67</td>
        <td>4.31</td>
        <td>0.92</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>4</td>
        <td>an</td>
        <td>Andorra</td>
        <td>468</td>
        <td>468</td>
        <td>0</td>
        <td>85580</td>
        <td>0.12</td>
        <td>8.13</td>
        <td>6.96</td>
        <td>0.0</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>5</td>
        <td>ao</td>
        <td>Angola</td>
        <td>1246700</td>
        <td>1246700</td>
        <td>0</td>
        <td>19625353</td>
        <td>2.78</td>
        <td>38.78</td>
        <td>11.49</td>
        <td>0.46</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>6</td>
        <td>ac</td>
        <td>Antigua and Barbuda</td>
        <td>442</td>
        <td>442</td>
        <td>0</td>
        <td>92436</td>
        <td>1.24</td>
        <td>15.85</td>
        <td>5.69</td>
        <td>2.21</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>7</td>
        <td>ar</td>
        <td>Argentina</td>
        <td>2780400</td>
        <td>2736690</td>
        <td>43710</td>
        <td>43431886</td>
        <td>0.93</td>
        <td>16.64</td>
        <td>7.33</td>
        <td>0.0</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>8</td>
        <td>am</td>
        <td>Armenia</td>
        <td>29743</td>
        <td>28203</td>
        <td>1540</td>
        <td>3056382</td>
        <td>0.15</td>
        <td>13.61</td>
        <td>9.34</td>
        <td>5.8</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>9</td>
        <td>as</td>
        <td>Australia</td>
        <td>7741220</td>
        <td>7682300</td>
        <td>58920</td>
        <td>22751014</td>
        <td>1.07</td>
        <td>12.15</td>
        <td>7.14</td>
        <td>5.65</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>10</td>
        <td>au</td>
        <td>Austria</td>
        <td>83871</td>
        <td>82445</td>
        <td>1426</td>
        <td>8665550</td>
        <td>0.55</td>
        <td>9.41</td>
        <td>9.42</td>
        <td>5.56</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>11</td>
        <td>aj</td>
        <td>Azerbaijan</td>
        <td>86600</td>
        <td>82629</td>
        <td>3971</td>
        <td>9780780</td>
        <td>0.96</td>
        <td>16.64</td>
        <td>7.07</td>
        <td>0.0</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>12</td>
        <td>bf</td>
        <td>Bahamas, The</td>
        <td>13880</td>
        <td>10010</td>
        <td>3870</td>
        <td>324597</td>
        <td>0.85</td>
        <td>15.5</td>
        <td>7.05</td>
        <td>0.0</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>13</td>
        <td>ba</td>
        <td>Bahrain</td>
        <td>760</td>
        <td>760</td>
        <td>0</td>
        <td>1346613</td>
        <td>2.41</td>
        <td>13.66</td>
        <td>2.69</td>
        <td>13.09</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>14</td>
        <td>bg</td>
        <td>Bangladesh</td>
        <td>148460</td>
        <td>130170</td>
        <td>18290</td>
        <td>168957745</td>
        <td>1.6</td>
        <td>21.14</td>
        <td>5.61</td>
        <td>0.46</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>15</td>
        <td>bb</td>
        <td>Barbados</td>
        <td>430</td>
        <td>430</td>
        <td>0</td>
        <td>290604</td>
        <td>0.31</td>
        <td>11.87</td>
        <td>8.44</td>
        <td>0.3</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>16</td>
        <td>bo</td>
        <td>Belarus</td>
        <td>207600</td>
        <td>202900</td>
        <td>4700</td>
        <td>9589689</td>
        <td>0.2</td>
        <td>10.7</td>
        <td>13.36</td>
        <td>0.7</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>17</td>
        <td>be</td>
        <td>Belgium</td>
        <td>30528</td>
        <td>30278</td>
        <td>250</td>
        <td>11323973</td>
        <td>0.76</td>
        <td>11.41</td>
        <td>9.63</td>
        <td>5.87</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>18</td>
        <td>bh</td>
        <td>Belize</td>
        <td>22966</td>
        <td>22806</td>
        <td>160</td>
        <td>347369</td>
        <td>1.87</td>
        <td>24.68</td>
        <td>5.97</td>
        <td>0.0</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>19</td>
        <td>bn</td>
        <td>Benin</td>
        <td>112622</td>
        <td>110622</td>
        <td>2000</td>
        <td>10448647</td>
        <td>2.78</td>
        <td>36.02</td>
        <td>8.21</td>
        <td>0.0</td>
        <td>0.0</td>
    </tr>
    <tr>
        <td>20</td>
        <td>bt</td>
        <td>Bhutan</td>
        <td>38394</td>
        <td>38394</td>
        <td>0</td>
        <td>741919</td>
        <td>1.11</td>
        <td>17.78</td>
        <td>6.69</td>
        <td>0.0</td>
        <td>0.0</td>
    </tr>
</table>



<b>Which countries have more water than land?

Looking at the results of our query above, it seems that there is something wrong with the data that is giving nearly every country a 0.0 ratio of water-to-land. We can see here -> https://en.wikipedia.org/wiki/Talk%3AList_of_countries_by_percentage_of_water_area that we're not the only ones to notice this data is incorrect. Due to this, we will leave these findings out of our summary.

<b>Which countries will add the most people to their populations next year?


```sql
%%sql
SELECT name, ROUND(birth_rate - death_rate, 2) AS birth_ratio
  FROM facts
 WHERE name <> 'World'
   AND name <> 'Antarctica'
   AND name <> 'Greenland' 
   AND name NOT LIKE '% Islands'
   AND name NOT LIKE '% City)'
   AND name NOT LIKE '% territory'
 ORDER BY birth_ratio DESC
 LIMIT 3;
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>birth_ratio</th>
    </tr>
    <tr>
        <td>Malawi</td>
        <td>33.15</td>
    </tr>
    <tr>
        <td>Uganda</td>
        <td>33.1</td>
    </tr>
    <tr>
        <td>Niger</td>
        <td>33.03</td>
    </tr>
</table>



From the above query, we can see that the top 3 countries which will add the most people to their population next year according to birth ratio are:

* <b>Malawi
* Uganda
* Niger </b>

All of which happen to be countries in the African Continent (where it's very hot climate)

<b>Which countries have a higher death rate than birth rate?


```sql
%%sql
SELECT name, ROUND(death_rate - birth_rate, 2) AS death_ratio
  FROM facts
 WHERE name <> 'World'
   AND name <> 'Antarctica'
   AND name <> 'Greenland' 
   AND name NOT LIKE '% Islands'
   AND name NOT LIKE '% City)'
   AND name NOT LIKE '% territory'
 ORDER BY death_ratio DESC
 LIMIT 3;
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>death_ratio</th>
    </tr>
    <tr>
        <td>Bulgaria</td>
        <td>5.52</td>
    </tr>
    <tr>
        <td>Serbia</td>
        <td>4.58</td>
    </tr>
    <tr>
        <td>Latvia</td>
        <td>4.31</td>
    </tr>
</table>



In the above query, we can see the top 3 countries which have higher death to birth ratios:

* <b>Bulgaria
* Serbia
* Latvia </b>

All 3 countries happen to be in Eastern Europe (where it's extremely cold)

<b>Which countries have the highest population/area ratio, and how does it compare to list we found in the previous screen?


```sql
%%sql
SELECT name, ROUND(population / area, 2) AS pop_area_ratio
  FROM facts
 WHERE name <> 'World'
   AND name <> 'Antarctica'
   AND name <> 'Greenland' 
   AND name NOT LIKE '% Islands'
   AND name NOT LIKE '% City)'
   AND name NOT LIKE '% territory'
 ORDER BY pop_area_ratio DESC
 LIMIT 3;
```

    Done.





<table>
    <tr>
        <th>name</th>
        <th>pop_area_ratio</th>
    </tr>
    <tr>
        <td>Macau</td>
        <td>21168.0</td>
    </tr>
    <tr>
        <td>Monaco</td>
        <td>15267.0</td>
    </tr>
    <tr>
        <td>Singapore</td>
        <td>8141.0</td>
    </tr>
</table>



In the above query, we can see that the top 3 countries with the highest population/area ratio are:

* <b>Macau
* Monaco
* Singapore </b>

Interestingly, these 3 countries are well developed and known to be some of the richest/most expensive countries and are all relatively small.

## Summary of statistics found in this analysis:

### In the course of this analysis of CIA Factbook data using SQL, we discovered the following statistics:

* The lowest population of any country on Earth is 0, and that country (technically a Continent) is Antartica
* Controlling for Antartica, we can see that 
* The highest population of any country on Earth is 1,367,485,388, and that country is China
* The lowest growth rate is 0.01: Greece
* The highest growth rate is 4.02: South Sudan
* The top 7 countries with above average populations AND below average total area are:
 * Bangladesh
 * Japan
 * Philippines
 * Vietnam
 * Germany
 * Thailand
 * United Kingdom
* The water and land ratios are incorrect in the data so we did not find any conclusions involving that data
* The top 3 countries with the highest birth/death ratios:
 * Malawi
 * Uganda
 * Niger
* The top 3 countries with the highest death/birth ratios:
 * Bulgaria
 * Serbia
 * Latvia
* The top 3 countries with the highest population/area ratios:
 * Macau
 * Monaco
 * Singapore



```python

```
