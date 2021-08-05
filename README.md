# COVID_Death-Vaccination_SQL




CREATE DATABASE PortfolioProject;
USE PortfolioProject;

SELECT * 
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 3,4


--SELECT * 
--FROM PortfolioProject.CovidVaccination
--ORDER BY 3,4

--Select data that we are going to be using


SELECT Location, date , total_cases, new_cases, total_deaths, population 
FROM PortfolioProject..COVIDDEATHS
WHERE continent IS NOT NULL
ORDER BY location;


-- Looking at Total Cases vs Total Deaths
-- And Shows liklihood of dying if you contract COVID in your country
SELECT Location, date , total_cases, total_deaths, (total_deaths/total_cases)*100 AS DeathPercentage
FROM PortfolioProject..COVIDDEATHS
WHERE continent IS NOT NULL
ORDER BY 1,2;

SELECT Location, date , total_cases, total_deaths, (total_deaths/total_cases)*100 AS DeathPercentage
FROM PortfolioProject..COVIDDEATHS
WHERE Location = 'India' AND continent IS NOT NULL
ORDER BY 1,2;



--Looking at Total cases vs Population
--Shows what percentage of population got covid
SELECT Location, date ,  population,total_cases, (total_cases/population)*100 AS PercentPopulationInfected
FROM PortfolioProject..COVIDDEATHS 
WHERE Location = 'India' AND continent IS NOT NULL
ORDER BY 1,2;

SELECT Location, date ,  population,total_cases, (total_cases/population)*100 AS PercentPopulationInfected
FROM PortfolioProject..COVIDDEATHS
WHERE continent IS NOT NULL
ORDER BY 1,2;


--Looking at Countries with Highest Infection Rate Compared to Population

SELECT Location, population, Max(total_cases) AS HighestInfected, MAX((total_cases/population))*100 AS PercentPopulationInfected
FROM PortfolioProject..COVIDDEATHS
--WHERE Location = 'India'
GROUP BY Location, population
ORDER BY PercentPopulationInfected desc;



-- Showing Countries  with Highest Death Count per Population

SELECT Location, MAX(CAST(total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..COVIDDEATHS
WHERE  continent IS NOT NULL  --AND Location = 'India'
GROUP BY Location
ORDER BY TotalDeathCount desc;



--Let's Break Things Down By  Continent
--Showing Continent with the highest death count per population
SELECT location, MAX(CAST(total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..COVIDDEATHS
WHERE  continent IS NULL AND LOCATION = 'ASIA'
GROUP BY location
ORDER BY TotalDeathCount desc;


SELECT continent, MAX(CAST(total_deaths AS int)) AS TotalDeathCount
FROM PortfolioProject..COVIDDEATHS
WHERE  continent IS NULL AND LOCATION = 'ASIA'
GROUP BY continent
ORDER BY TotalDeathCount desc;


--Global Numbers
 
 --Total Deaths and New Cases by Date
SELECT date , SUM(new_cases) AS total_cases, SUM(CAST(new_deaths AS int)) AS total_deaths, 
(SUM(CAST(new_deaths AS int))/SUM(new_cases))*100 AS DeathPercentage
FROM PortfolioProject..COVIDDEATHS
WHERE continent IS NOT NULL
GROUP BY date
ORDER BY 1,2;


--Total Deaths and New Cases till 2nd Aug 2021 
SELECT SUM(new_cases) AS total_cases, SUM(CAST(new_deaths AS int)) AS total_deaths, 
(SUM(CAST(new_deaths AS int))/SUM(new_cases))*100 AS DeathPercentage
FROM PortfolioProject..COVIDDEATHS
WHERE continent IS NOT NULL
--GROUP BY date
ORDER BY 1,2;




--Looking at Total population vs Vaccinations

SELECT D1.continent, D1.location, D1.date, D1.population, V1.new_vaccinations,
SUM(CONVERT(int, V1.new_vaccinations)) OVER (PARTITION BY D1.location ORDER BY D1.location, D1.date) AS TotalVaccinatedPeopleByCountry
FROM CovidDeaths D1
JOIN CovidVaccination V1 
ON D1.location = V1.location
AND D1.date = V1.date
WHERE D1.continent IS NOT NULL --AND D1.location = 'India'
--ORDER BY 2,3

-- USE CTE

WITH PopvsVac (continent,location,date,population,new_vaccinations,TotalVaccinatedPeopleByCountry)
as
(
SELECT D1.continent, D1.location, D1.date, D1.population, V1.new_vaccinations,
SUM(CONVERT(int, V1.new_vaccinations)) OVER (PARTITION BY D1.location ORDER BY D1.location, D1.date) AS TotalVaccinatedPeopleByCountry
FROM CovidDeaths D1
JOIN CovidVaccination V1 
ON D1.location = V1.location
AND D1.date = V1.date
WHERE D1.continent IS NOT NULL --AND D1.location = 'India'
)
SELECT *, (TotalVaccinatedPeopleByCountry/population)*100 as PercentageofPeopleVaccinated  -------------- How Many Percentage of people got Vaccinated
FROM PopvsVac




--Temp Table
DROP TABLE IF Exists #PercentPopulationVaccinated
CREATE TABLE #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
TotalVaccinatedPeopleByCountry numeric
)

INSERT INTO #PercentPopulationVaccinated

SELECT D1.continent, D1.location, CAST(D1.date AS INT), D1.population, V1.new_vaccinations,
SUM(CONVERT(int, V1.new_vaccinations)) OVER (PARTITION BY D1.location ORDER BY D1.location, D1.date) AS TotalVaccinatedPeopleByCountry
FROM PortfolioProject..CovidDeaths D1
JOIN PortfolioProject..CovidVaccination V1 
ON D1.location = V1.location
AND D1.date = V1.date
WHERE D1.continent IS NOT NULL --AND D1.location = 'India'

 


	---Creating view to store  data for later Visualizations 

	Create VIEW PercentPopulationVaccinated AS SELECT D1.continent, D1.location,D1.date, D1.population, V1.new_vaccinations,
      SUM(CONVERT(int, V1.new_vaccinations)) OVER (PARTITION BY D1.location ORDER BY D1.location, D1.date) AS TotalVaccinatedPeopleByCountry
       FROM PortfolioProject..CovidDeaths D1
      JOIN PortfolioProject..CovidVaccination V1 
      ON D1.location = V1.location
     AND D1.date = V1.date
      WHERE D1.continent IS NOT NULL 

	  SELECT * FROM PercentPopulationVaccinated
