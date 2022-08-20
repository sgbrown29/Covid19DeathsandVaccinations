# Covid-19 Deaths and Vaccinations Project

This is a data exploration project that examines the death and vaccination rates of the world during the Covid-19 Pandemic.  I will also soon create a Tableau dashboard to visualize this data, which can be accessed through my Tableau Public [here](https://public.tableau.com/app/profile/steven.brown8512).

### Data Tools Used
- Microsoft Excel (for initial data organizing)
- BigQuery SQL Workspace (for queries)

### Data Source
- [OurWorldinData.org Covid Deaths Dataset](https://ourworldindata.org/covid-deaths)

Because the main source data is updated daily, the coviddeaths and covidvaccinations files are too large to add to this repository, but the information from the queries is accessable.  

## SQL Walkthrough

In the coviddeaths table, the main columns being examined are `location`, `date`, `total_cases`, `new_cases`, `total_deaths`, and `population`. 

```SQL
SELECT location, date, total_cases, new_cases, total_deaths, population
FROM `portfolioproject-359316.covid19.coviddeaths`
  ORDER BY 1, 2
```
Results in repository: [coviddeathsessential.csv](https://github.com/sgbrown29/Covid19DeathsandVaccinations/blob/main/coviddeathsessential.csv)

