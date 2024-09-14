# SQL-Database-Exploration
# COVID-19 Data Exploration

## Project Overview

This project explores global COVID-19 data, focusing on key metrics such as infection rates, death percentages, and vaccination progress across different countries and continents. The analysis is done using SQL, highlighting various data manipulation techniques.

## Skills Demonstrated

- **Joins**
- **CTEs (Common Table Expressions)**
- **Temporary Tables**
- **Window Functions**
- **Aggregate Functions**
- **Views**
- **Data Type Conversions**

## Datasets

1. **CovidDeaths$** – Contains statistics such as total cases, new cases, deaths, and population by country.
2. **CovidVaccination$** – Contains information about COVID-19 vaccination efforts, including new vaccinations administered.

---

## SQL Queries

```sql
-- Initial Exploration
SELECT * 
FROM PortfolioProject..CovidDeaths$ 
WHERE continent IS NOT NULL 
ORDER BY 3, 4;

-- Select Data for Exploration
SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM PortfolioProject..CovidDeaths$
WHERE continent IS NOT NULL 
ORDER BY 1, 2;

-- Total Cases vs Total Deaths
SELECT Location, date, total_cases, total_deaths, 
       (total_deaths / total_cases) * 100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths$
WHERE location LIKE '%states%'
  AND continent IS NOT NULL
ORDER BY 1, 2;

-- Total Cases vs Population
SELECT Location, date, Population, total_cases, 
       (total_cases / population) * 100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths$
ORDER BY 1, 2;

-- Highest Infection Rate
SELECT Location, Population, MAX(total_cases) AS HighestInfectionCount,  
       MAX((total_cases / population)) * 100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths$
GROUP BY Location, Population
ORDER BY PercentPopulationInfected DESC;

-- Highest Death Count per Population
SELECT Location, MAX(CAST(Total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths$
WHERE continent IS NOT NULL 
GROUP BY Location
ORDER BY TotalDeathCount DESC;

-- Continental Breakdown
SELECT continent, MAX(CAST(Total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths$
WHERE continent IS NOT NULL 
GROUP BY continent
ORDER BY TotalDeathCount DESC;

-- Global Statistics
SELECT SUM(new_cases) AS total_cases, 
       SUM(CAST(new_deaths AS int)) AS total_deaths, 
       SUM(CAST(new_deaths AS int)) / SUM(New_Cases) * 100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths$
WHERE continent IS NOT NULL
ORDER BY 1, 2;

-- Total Population vs Vaccinations
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location 
       ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths$ dea
JOIN PortfolioProject..CovidVaccination$ vac
ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
ORDER BY 2, 3;

-- Using CTE to Perform Calculation
WITH PopvsVac AS (
    SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
           SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location 
           ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
    FROM PortfolioProject..CovidDeaths$ dea
    JOIN PortfolioProject..CovidVaccination$ vac
    ON dea.location = vac.location AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL
)
SELECT *, (RollingPeopleVaccinated / Population) * 100 AS PercentPopulationVaccinated
FROM PopvsVac;

-- Using Temporary Table for Vaccination Rates
DROP TABLE IF EXISTS #PercentPopulationVaccinated;

CREATE TABLE #PercentPopulationVaccinated (
    Continent nvarchar(255),
    Location nvarchar(255),
    Date datetime,
    Population numeric,
    New_vaccinations numeric,
    RollingPeopleVaccinated numeric
);

INSERT INTO #PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location 
       ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths$ dea
JOIN PortfolioProject..CovidVaccination$ vac
ON dea.location = vac.location AND dea.date = vac.date;

SELECT *, (RollingPeopleVaccinated / Population) * 100 AS PercentPopulationVaccinated
FROM #PercentPopulationVaccinated;

-- Creating a View for Vaccination Data
CREATE VIEW PercentPopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location 
       ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths$ dea
JOIN PortfolioProject..CovidVaccination$ vac
ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;
