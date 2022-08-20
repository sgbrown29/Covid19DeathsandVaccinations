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
>Results in repository: [coviddeathsessential.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/coviddeathsessential.csv)

### covidvaccinations Table

In the covidvaccinations table, the main columns being examined are `location`, `date`, `total_tests`, `new_tests`, and `population`.

```SQL
SELECT location, date, total_tests, new_tests, population
FROM `portfolioproject-359316.covid19.covidvaccinations`
  ORDER BY 1, 2
```
>Results in repository: [covidvaccinationsessential.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/covidvaccinationsessential.csv)

### Query 1
The first query examines the cases and deaths per country, and creates a % chance of death based on the total case count.  This shows your daily chances of dying should you become infected for each country.  

```SQL
SELECT location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS DeathPercentage
FROM `portfolioproject-359316.covid19.coviddeaths`
WHERE continent IS NOT NULL
  ORDER BY 1,2
 ```
Because `location` has entries for each continent and income level, such as "Africa, North America, Low Income, etc.", we use the WHERE clause `continent IS NOT NULL`, which removes these locations and only includes country names in the final result. `ORDER BY 1,2` allows the results to be arranged by alphabetically by location and chronologically by date.

### Query 2


