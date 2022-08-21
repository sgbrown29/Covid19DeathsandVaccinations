# Covid-19 Deaths and Vaccinations Project

This is a data exploration project that examines the death and vaccination rates of the world during the Covid-19 Pandemic.  I will also soon create a Tableau dashboard to visualize this data, which can be accessed through my Tableau Public [here](https://public.tableau.com/app/profile/steven.brown8512).

### Data Tools Used
- Microsoft Excel (for initial data organizing)
- BigQuery SQL Workspace (for queries)

### Data Source
- [OurWorldinData.org Covid Deaths Dataset](https://ourworldindata.org/covid-deaths)

Because the main source data is updated daily, the coviddeaths and covidvaccinations files are too large to add to this repository, but the information from the queries is accessable.  

## SQL Walkthrough
### coviddeaths Table
In the coviddeaths table, the main columns being examined are `location`, `date`, `total_cases`, `new_cases`, `total_deaths`, and `population`. `location` contains a list of every country with recorded covid data, as well as continents and income level.

```SQL
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM `portfolioproject-359316.covid19.coviddeaths`
  ORDER BY 1, 2
```
>Results in repository: [CovidDeathsEssential.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/CovidDeathsEssential.csv)

### covidvaccinations Table

In the covidvaccinations table, the main columns being examined are `location`, `date`, `total_tests`, `new_tests`, and `population`.

```SQL
SELECT location, date, total_tests, new_tests, population
FROM `portfolioproject-359316.covid19.covidvaccinations`
  ORDER BY 1, 2
```
>Results in repository: [CovidVaccinationsEssential.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/CovidVaccinationsEssential.csv)

### Query 1
The first query examines the cases and deaths per country, and creates a % chance of death based on the total case count.  This shows your daily chances of dying should you become infected for each country.  

```SQL
SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS DeathPercentage
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE continent IS NOT NULL
  ORDER BY 1,2
 ```
 >Results in repository: [DailyDeathPercent.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/DailyDeathPercent.csv)

Because `location` has entries for each continent and income level, such as "Africa, North America, Low Income, etc.", we use the WHERE clause `continent IS NOT NULL`, which removes these locations and only includes country names in the final result. `ORDER BY 1,2` allows the results to be arranged alphabetically by location and chronologically by date.

### Query 2
This examines daily infection levels per country, and creates a percentage of the population that became infected.

```SQL
SELECT location, date, total_cases, population, (total_cases/population)*100 AS InfectionPercentage
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE continent IS NOT NULL
  ORDER BY 1, 2
 ```
 >Results in repository: [DailyInfectionPercentage.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/DailyInfectionPercentage.csv)

### Query 3
Query 3 examines the countries with the highest infection rates as a percent of the population.

```SQL
SELECT location, population, MAX(total_cases) AS HighestInfectionCount, 
  MAX((total_cases/population))*100 AS PercentPopulationInfected
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE continent IS NOT NULL
  GROUP BY location, population
  ORDER BY PercentPopulationInfected DESC
 ```
 >Results in repository: [CountryInfectionRate.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/CountryInfectionRate.csv)

### Query 4
Query 4 examines countries with the highest death counts as a total sum by taking the `MAX` of `total_deaths`, which is the most recent entry.

```SQL
SELECT location, MAX(Total_deaths) AS TotalDeathCount
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE continent IS NOT NULL
  GROUP BY location
  ORDER BY TotalDeathCount DESC 
```
>Results in repository: [CountryDeathCount.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/CountryDeathCount.csv)

### Query 5
Query 5 examines continents with the highest death counts as a total sum, the same way as Query 4, but instead with the `WHERE` clause `continent IS NULL`.  The clauses `location NOT LIKE '%income%'` and `location NOT LIKE '%Union%'` are to ignore income level entries and the European Union entry. 

```SQL
SELECT location, MAX(total_deaths) AS total_death_count
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE location NOT LIKE '%income%' 
  AND location NOT LIKE '%Union%'
  AND continent IS NULL 
  GROUP BY location
  ORDER BY total_death_count DESC
```
>Results in repository: [ContinentDeathCount.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/ContinentDeathCount.csv)

Results also below:

| location | total_death_count |
|---------|-------|
|World|6451050|
|Europe|1900463|
North America|1489924|
Asia|1464328|
South America|1321971|
Africa|256429|
Oceania|17920|
International|15|

### Query 6
Query 6 examines the total death count based on income level around the world.  This is similar to Query 5, only we use the `WHERE` clause `location LIKE '%income%'`.

```SQL
SELECT location, MAX(total_deaths) AS total_death_count
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE location LIKE '%income%' 
  AND location NOT LIKE '%Union%'
  AND continent IS NULL 
  GROUP BY location
  ORDER BY total_death_count DESC
 ```
  
>Results in repository: [IncomeDeathCount](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/IncomeDeathCount.csv)

Results also below:

| location | total_death_count |
|---------|-------|
Upper middle income	|2560751|
High income	|2524434|
Lower middle income	|1322311|
Low income	|43475|
