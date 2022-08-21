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
In the coviddeaths table, the main columns being examined are `location`, `date`, `total_cases`, `new_cases`, `total_deaths`, and `population`. `location` contains a list of every country that has recorded covid data, as well as continents and income level.

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
The first query examines the cases and deaths per country, and creates a % chance of death based on the total case count.  This shows your daily chances of dying should you become infected for each country.  Because `location` has entries for each continent and income level, such as "Africa, North America, Low Income, etc.", we use the WHERE clause `continent IS NOT NULL`, which removes these locations and only includes country names in the final result. `ORDER BY 1,2` allows the results to be arranged alphabetically by location and chronologically by date.

```SQL
SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS death_percentage
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE continent IS NOT NULL
  ORDER BY 1,2
 ```
 >Results in repository: [DailyDeathPercent.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/DailyDeathPercent.csv)


### Query 2
This examines daily infection levels per country, and creates a percentage of the population that became infected.

```SQL
SELECT location, date, total_cases, population, (total_cases/population)*100 AS infection_percentage
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE continent IS NOT NULL
  ORDER BY 1, 2
 ```
 >Results in repository: [DailyInfectionPercentage.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/DailyInfectionPercentage.csv)

### Query 3
Query 3 examines the countries with the highest infection rates as a percent of the population.

```SQL
SELECT location, population, MAX(total_cases) AS highest_infection_count, 
  MAX((total_cases/population))*100 AS percent_population_infected
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE continent IS NOT NULL
  GROUP BY location, population
  ORDER BY percent_population_infected DESC
  
 ```
 >Results in repository: [CountryInfectionRate.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/CountryInfectionRate.csv)

### Query 4
Query 4 examines countries with the highest death counts as a total sum by taking the `MAX` of `total_deaths`, which is the most recent entry.

```SQL
SELECT location, MAX(total_deaths) AS total_death_count
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE continent IS NOT NULL
  GROUP BY location
  ORDER BY total_death_count DESC 
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
  
>Results in repository: [IncomeDeathCount.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/IncomeDeathCount.csv)

Results also below:

| location | total_death_count |
|---------|-------|
Upper middle income	|2560751|
High income	|2524434|
Lower middle income	|1322311|
Low income	|43475|

### Query 7
For Query 7, we used a `JOIN` clause to join both the `coviddeaths` and `covidvaccinations` tables to look at the number of new daily doses per country.  Note that `vac.new_vaccinations` was renamed `new_doses`. This is because `vac.new_vaccinations` tracks the number of doses given in that country in a single day, regardless of first, second, or booster dose.  This number DOES NOT mean the number of new people receiving their first dose. The `PARTITION` function creates a rolling count of the cumulative doses given for every country, and resets when `location` changes.  

```SQL
SELECT dea.continent, dea.location, dea.date, dea.population, 
  vac.new_vaccinations AS new_doses, SUM(new_vaccinations) 
  OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS rolling_total_doses
FROM `portfolioproject-359316.covid19.coviddeaths` dea
JOIN`portfolioproject-359316.covid19.covidvaccinations` vac
  ON dea.location = vac.location 
  AND dea.date = vac.date
WHERE
  dea.continent IS NOT NULL
  ORDER BY 2, 3
```

>Results in repository: [RollingTotalDoses.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/RollingTotalDoses.csv)

### Query 8
Query 8 examines the total number of doses given for every country.  These totals are obviously more than the total population per country.

```SQL
Select location, population, MAX(rolling_total_doses) AS total_doses_given
From `portfolioproject-359316.covid19.RollingTotalDoses`
  GROUP BY location, population
  ORDER BY total_doses_given desc
```

>Results in repository: [CountryDosesGiven.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/CountryDosesGiven.csv)

### Query 9
Query 9 examines the daily number of people with a single dose, and people who are fully vaccinated with respect to a country's population.  This query uses a similar `JOIN` structure as Query 7, but instead of `PARTITION`, we are creating single dose and fully vaccinated population percentages.  

```SQL
SELECT dea.continent, dea.location, dea.date, dea.population, vac.people_vaccinated as people_single_dose, vac.people_fully_vaccinated, (vac.people_vaccinated/dea.population)*100 AS percent_single_dose, (vac.people_fully_vaccinated/dea.population)*100 AS percent_fully_vaccinated
FROM `portfolioproject-359316.covid19.coviddeaths` dea
JOIN`portfolioproject-359316.covid19.covidvaccinations` vac
  ON dea.location = vac.location 
  AND dea.date = vac.date
WHERE
  dea.continent IS NOT NULL
  ORDER BY 2, 3
```
>Results in repository: [PercentPopulationVaccinated.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/PercentPopulationVaccinated.csv)

### Query 10
Finally, Query 10 examines the countries with the highest full-vaccination rates.  

```SQL
SELECT location, population, MAX(percent_single_dose) AS percent_single_dose, MAX(percent_fully_vaccinated) AS percent_fully_vaccinated
FROM `portfolioproject-359316.covid19.PercentPopulationVaccinated`
WHERE continent IS NOT NULL
  GROUP BY location, population
  ORDER BY percent_fully_vaccinated DESC
```
>Results in repository: [CountryVaccinationRates.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/CountryVaccinationRates.csv)

