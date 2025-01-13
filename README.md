**COVID-19 Data Exploration Project**
This project involves analyzing global COVID-19 data, focusing on cases, deaths, and vaccinations. The analysis is performed using SQL, utilizing various advanced techniques to gain insights into the pandemic's impact.

**Skills and Tools Used**
SQL Concepts: Joins, Common Table Expressions (CTEs), Temporary Tables, Views, and Window Functions.
Functions Applied: Aggregate Functions, Data Type Conversions, and Calculations.
SQL Server Management Studio (SSMS) for writing and executing queries.
Project Description
The project explores the following:

Global Trends: COVID-19 cases and deaths across different countries and continents.
Death Rates: Percentage of deaths relative to total cases by location.
Infection Rates: Percentage of populations infected with COVID-19.
Vaccination Rollouts: Analysis of cumulative vaccinations over time, as well as the percentage of populations vaccinated.
Key Insights: Identification of countries with the highest infection and death rates.
**Dataset Description:**

The analysis is based on two datasets:

**CovidDeaths:** Contains information on:
Total cases
New cases
Total deaths
Population by country
**CovidVaccinations:** Contains information on:
New vaccinations administered by country and date

Both datasets are structured and stored in a SQL database named PortfolioProject.

**Project Goals**
To understand the spread and severity of COVID-19 across regions.
To identify vaccination trends and their correlation with population sizes.
To calculate and visualize key metrics such as infection rates and death percentages.

**Analysis Breakdown:**
1. Death Rates vs. Total Cases
Query to calculate the percentage of total cases that resulted in deaths:

SELECT Location, Date, Total_Cases, Total_Deaths,
       (Total_Deaths / Total_Cases) * 100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE Continent IS NOT NULL
ORDER BY Location, Date;

2. Infection Rates vs. Population
Query to calculate the percentage of a country's population infected with COVID-19:

SELECT Location, Date, Population, Total_Cases,
       (Total_Cases / Population) * 100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
WHERE Continent IS NOT NULL
ORDER BY Location, Date;

3. Vaccination Trends
Query to calculate the cumulative number of people vaccinated per country:

SELECT dea.Continent, dea.Location, dea.Date, dea.Population, vac.New_Vaccinations,
       SUM(CONVERT(INT, vac.New_Vaccinations)) 
       OVER (PARTITION BY dea.Location ORDER BY dea.Date) AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
  ON dea.Location = vac.Location AND dea.Date = vac.Date
WHERE dea.Continent IS NOT NULL;

4. Insights by Continents
Query to find the continent with the highest death counts:

SELECT Continent, MAX(CAST(Total_Deaths AS INT)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE Continent IS NOT NULL
GROUP BY Continent
ORDER BY TotalDeathCount DESC;

**Key SQL Features Used**

1. Common Table Expressions (CTEs)
Simplified queries by breaking down complex calculations.
Example: Using a CTE to calculate vaccination rates.
2. Window Functions
Calculated cumulative vaccination counts for rolling totals.
Partitioned data by country to group calculations.
3. Temporary Tables
Used to store intermediate results for better performance and modularity.
4. Views
Created reusable views to simplify data visualization workflows.

**Results and Insights**
Infection and Death Trends: Countries with the highest infection and death rates were identified.
Vaccination Progress: Rolling totals showed how vaccination efforts progressed over time.
Global Metrics:
Death percentage relative to total cases.
Infection percentage relative to population.

**All the Codes Used**
Select *
From PortfolioProject..CovidDeaths
Where continent is not null 
order by 3,4


-- Select Data that we are going to be starting with

Select Location, date, total_cases, new_cases, total_deaths, population
From PortfolioProject..CovidDeaths
Where continent is not null 
order by 1,2


-- Total Cases vs Total Deaths
-- Shows likelihood of dying if you contract covid in your country

Select Location, date, total_cases,total_deaths, (total_deaths/total_cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
Where location like '%States%'
and continent is not null 
order by 1,2


-- Total Cases vs Population
-- Shows what percentage of population infected with Covid

Select Location, date, Population, total_cases,  (total_cases/population)*100 as PercentPopulationInfected
From PortfolioProject..CovidDeaths
order by 1,2


-- Countries with Highest Infection Rate compared to Population

Select Location, Population, MAX(total_cases) as HighestInfectionCount,  Max((total_cases/population))*100 as PercentPopulationInfected
From PortfolioProject..CovidDeaths
Group by Location, Population
order by PercentPopulationInfected desc


-- Countries with Highest Death Count per Population

Select Location, MAX(cast(Total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
Where continent is not null 
Group by Location
order by TotalDeathCount desc



-- BREAKING THINGS DOWN BY CONTINENT

-- Showing contintents with the highest death count per population

Select continent, MAX(cast(Total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
Where continent is not null 
Group by continent
order by TotalDeathCount desc



-- GLOBAL NUMBERS

Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From PortfolioProject..CovidDeaths
--Where location like '%states%'
where continent is not null 
--Group By date
order by 1,2



-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3


-- Using CTE to perform Calculation on Partition By in previous query

With PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
--order by 2,3
)
Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac



-- Using Temp Table to perform Calculation on Partition By in previous query

DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated




-- Creating View to store data for later visualizations

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 


