)# Covid 19 Data Analysis
![Python](https://img.shields.io/badge/python-3670A0?style=for-the-badge&logo=python&logoColor=ffdd54)
![Pandas](https://img.shields.io/badge/pandas-%23150458.svg?style=for-the-badge&logo=pandas&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-%23ffffff.svg?style=for-the-badge&logo=Matplotlib&logoColor=black)
![Power Bi](https://img.shields.io/badge/power_bi-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)

This is the project in which we analyze about the Covid-19 Pandemic Condition through out the world including total cases, deaths and vaccinations.
## Objective:
To understand the current situation occured to the Covid 19 Pandemic in the world. To find the insights from the data about the number of cases of covid occured at global level in each continent along with the number of deaths and total vaccinations happen. The purpose of this to develope the dynamic dashboard using Power BI software and tools to observe different regions and continents so that understand the trends of this pandemic and make counter plan according to the regions.
## Dashboard
### 1. Key Insights:
![Key Insights](https://github.com/Rohit-Moghe/Covid-19-Data-Analysis/blob/310c47136b1a5dbfbc998d55648b6f2beb7bd99c/Key%20Insights/Dashboard.jpg)
![Key Insights](https://user-images.githubusercontent.com/96460908/153005798-29994671-023b-499f-a5e1-6254460d3085.jpg)
### 2. Region Wise Data:
![Data](https://user-images.githubusercontent.com/96460908/153546284-ae5d414a-7554-437d-96bb-81b05757514c.jpg)
### 3. Covid Data for TOP 15 Countries:
![Covid](https://user-images.githubusercontent.com/96460908/153546378-7d2fcffa-f19b-469c-9cdc-a77c1c48675e.jpg)
### 4. Vaccination Data for Top 15 Countries:
![Vaccinations Data](https://user-images.githubusercontent.com/96460908/153546436-5a834b79-658a-418c-9486-cacb26ab5b5d.jpg)
## Database and Technology used:
Database: https://ourworldindata.org/covid-deaths
SQL: BigQuery
Data Visualization tools: Power-BI Power Editor with Dax Editor
## SQL for Data Analysis:
/*
	Covid 19 Data Exploration
	Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types
	Data Cleaning performed in MS Excel
	SQL Query Editor: BigQuery
	Dataset: https://ourworldindata.org/covid-deaths
	Data Visualization tools: Power-BI Power Editor with Dax Editor
*/


SELECT *
FROM covid_deaths.covid_death
ORDER BY 3, 4

-- Select Data we are going to use

SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM covid_deaths.covid_death
ORDER BY 1, 2

-- Looking at total case VS total deaths
-- Shows likelihood of dying if you get infected in your country

SELECT Location, date, total_cases, total_deaths, ((total_deaths/total_cases)*100) AS Death_Percentage
FROM covid_deaths.covid_death
WHERE Location = "India"  --Can be change with representative country
ORDER BY 1, 2

-- Looking Total Case VS Populaltion
-- Shows what percentage of population for covid

SELECT Location, date, total_cases, population, ((total_cases/population)*100) AS Covid_Percentage
FROM covid_deaths.covid_death
WHERE Location = "India"  --Can be change with representative country
ORDER BY 1, 2

-- Highest Infection Rate in Country

SELECT Location, population, MAX(total_cases) AS Highest_Infection_Count, MAX((total_cases/population)*100) AS Population_Infected 
FROM covid_deaths.covid_death
GROUP BY Location, population
ORDER BY Population_Infected DESC

-- Highest Death Count

SELECT Location, Population, MAX(total_deaths) AS Death_Count
FROM covid_deaths.covid_death
WHERE continent is not null 
GROUP BY Location, population
ORDER BY Death_Count DESC

-- Breaking it down by Continent

SELECT location, Population, MAX(total_deaths) AS Death_Count
FROM covid_deaths.covid_death
WHERE continent is null
GROUP BY location, population
ORDER BY Death_Count DESC

-- GLOBAL NUMBERS

SELECT date, SUM(new_cases) AS Total_Cases, SUM(new_deaths) AS Total_Deaths, ((SUM(new_deaths)/SUM(new_cases))*100) AS Death_Percantage
FROM covid_deaths.covid_death
WHERE Location = "India"  Can be change with representative country
WHERE continent is not null
GROUP BY date
ORDER BY 1, 2

-- LOOKING AT TOTAL POLLUTION VS TOTAL VACCINATION

SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(vac.new_vaccinations) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS Total_Rolling_Vaccinations
FROM covid_deaths.covid_death AS dea
JOIN covid_deaths.covid_vaccinations AS vac
    ON dea.location = vac.location
    AND dea.date = vac.date
WHERE dea.continent is not null
ORDER BY 2, 3

-- Using CTE to perform calculations

With PopvsVac
AS (
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated, 
       --(RollingPeopleVaccinated/population)*100
FROM covid_deaths.covid_death AS dea
JOIN covid_deaths.covid_vaccinations AS vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
)
Select *, (RollingPeopleVaccinated/Population)*100 AS vaccinated_percentage
From PopvsVac

-- TEMP TABLE

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

-- INSERT TEMP TABLE

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated,
       --(RollingPeopleVaccinated/population)*100
    FROM covid_deaths.covid_death AS dea
    JOIN covid_deaths.covid_vaccinations AS vac
	    On dea.location = vac.location
    	and dea.date = vac.date
WHERE dea.continent is not null
ORDER BY 2, 3

Select *, (RollingPeopleVaccinated/Population)*100 AS vaccinated_Percentage
From #PercentPopulationVaccinated

--Creating View to store data for later visualizations

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
       SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated,
       --(RollingPeopleVaccinated/population)*100
    FROM covid_deaths.covid_death AS dea
    JOIN covid_deaths.covid_vaccinations AS vac
	    On dea.location = vac.location
    	and dea.date = vac.date
where dea.continent is not null

## END RESULT:
An automated dashboard providing quick and latest Covid-19 insights in order to support data driven decision making.
