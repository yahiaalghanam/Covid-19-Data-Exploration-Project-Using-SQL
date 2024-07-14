
# COVID-19 Data Analysis Project

## Overview

This project aims to analyze COVID-19 data, focusing on cases, deaths, and vaccinations. The operations include examining total cases versus deaths, infection rates, and vaccination rates across various locations and continents.

## Table of Contents

- [Data Selection](#data-selection)
- [Total Cases vs Total Deaths](#total-cases-vs-total-deaths)
- [Percentage of Population Infected](#percentage-of-population-infected)
- [Countries with Highest Infection Rate](#countries-with-highest-infection-rate)
- [Countries with Highest Death Count per Population](#countries-with-highest-death-count-per-population)
- [Analysis by Continent](#analysis-by-continent)
- [Global Numbers](#global-numbers)
- [Vaccinations vs Population](#vaccinations-vs-population)
- [Number of People Vaccinated in Every Country](#number-of-people-vaccinated-in-every-country)
- [Temporary Table and View Creation](#temporary-table-and-view-creation)

## Data Selection

```sql
SELECT location, date, total_cases, new_cases, total_deaths, population 
FROM covid_death
ORDER BY 1, 2;
```

This query selects essential columns from the `covid_death` table, providing a comprehensive dataset for analysis.

## Total Cases vs Total Deaths

```sql
SELECT location, date, total_cases, new_cases, total_deaths, 
       (total_deaths / total_cases) * 100 AS total_death_percentage 
FROM covid_death
WHERE location LIKE '%Egypt%'
ORDER BY 1, 2;
```

This query calculates the percentage of total deaths compared to total cases, specifically for Egypt.

## Percentage of Population Infected

```sql
SELECT location, date, total_cases, population, 
       (total_cases / population) * 100 AS total_death_percentage_infected 
FROM covid_death
ORDER BY 1, 2;
```

This query shows what percentage of the population in various locations has been infected by COVID-19.

## Countries with Highest Infection Rate

```sql
SELECT location, population, 
       MAX(total_cases / population) AS total_death_percentage_infected 
FROM covid_death
GROUP BY location, population
ORDER BY 3 DESC;
```

This query identifies countries with the highest infection rates compared to their populations.

## Countries with Highest Death Count per Population

```sql
SELECT location, MAX(CAST(total_deaths AS int)) AS total_death_count
FROM covid_death
WHERE continent IS NOT NULL 
GROUP BY location, population
ORDER BY total_death_count DESC;
```

This query shows countries with the highest death counts per population.

## Analysis by Continent

### Breakdown by Continent

```sql
SELECT location, MAX(CAST(total_deaths AS int)) AS total_death_count
FROM covid_death
WHERE continent IS NULL 
GROUP BY location
ORDER BY total_death_count DESC;
```

This query breaks down the data by continent, showing the death counts.

### Continent with Highest Death Count

```sql
SELECT MAX(continent), MAX(CAST(total_deaths AS int)) AS total_death_count
FROM covid_death
WHERE continent IS NOT NULL 
ORDER BY total_death_count DESC;
```

This query identifies the continent with the highest death count.

## Global Numbers

### Deaths Percentage Around the World

```sql
SELECT date, SUM(new_cases) AS SumOfNewCases, 
       SUM(CAST(new_deaths AS int)) AS SumOfNewDeaths,
       (SUM(CAST(new_deaths AS int)) / SUM(new_cases)) * 100 AS DeathPercentage 
FROM covid_death
WHERE continent IS NOT NULL 
GROUP BY date;
```

This query calculates the death percentage around the world.

## Vaccinations vs Population

```sql
SELECT dea.location, dea.continent, dea.date, population, vac.new_vaccinations,
       SUM(CAST(vac.new_vaccinations AS int)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS RolledPeopleVaccinated 
FROM covid_death dea 
JOIN covid_vaccinations vac ON dea.date = vac.date AND dea.location = vac.location;
```

This query looks for total vaccinations versus population by location.

## Number of People Vaccinated in Every Country

```sql
WITH popVSvac AS (
    SELECT dea.location, dea.continent, dea.date, population, vac.new_vaccinations,
           SUM(CAST(vac.new_vaccinations AS int)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS RolledPeopleVaccinated 
    FROM covid_death dea 
    JOIN covid_vaccinations vac ON dea.date = vac.date AND dea.location = vac.location 
)
SELECT *, (RolledPeopleVaccinated / population) * 100 AS VaccinationPercentage
FROM popVSvac;
```

This query calculates the number of people vaccinated in each country.

## Temporary Table and View Creation

### Create a Temporary Table

```sql
CREATE TABLE #PercentageOfVaccinations (
    continent NVARCHAR(255),
    location NVARCHAR(255),
    date DATETIME,
    population NUMERIC,
    new_vaccination NUMERIC,
    RolledPeopleVaccinated NUMERIC
);

INSERT INTO #PercentageOfVaccinations
SELECT dea.location, dea.continent, dea.date, population, vac.new_vaccinations,
       SUM(CAST(vac.new_vaccinations AS int)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS RolledPeopleVaccinated 
FROM covid_death dea 
JOIN covid_vaccinations vac ON dea.date = vac.date AND dea.location = vac.location;

SELECT *, (RolledPeopleVaccinated / population) * 100 AS VaccinationPercentage
FROM #PercentageOfVaccinations;
```

### Create a View for Data Visualization

```sql
CREATE VIEW PercentageOfVaccination AS
SELECT dea.location, dea.continent, dea.date, population, vac.new_vaccinations,
       SUM(CAST(vac.new_vaccinations AS int)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS RolledPeopleVaccinated 
FROM covid_death dea 
JOIN covid_vaccinations vac ON dea.date = vac.date AND dea.location = vac.location;
```

These queries create a temporary table and a view to store and visualize the data later.

