
# Covid 19 Data Exploration

Skills used: Joins, CTEs, Temp Tables, Windows Functions, Aggregated Functions, Views, Converting Data Types

Select *
From PortfolioProject..CovidDeaths
Order by 3,4

## 1 Import Data

Select location, date, total_cases, new_cases, total_deaths, population
From PortfolioProject..CovidDeaths
Order by 1,2

## 2 Lets look at Total Cases vs Total Deaths 
--This shows the likelihood of dying if you contract covid in Nigeria

Select location, date, total_cases, total_deaths, (total_deaths/total_cases) * 100 as DeathPct 
From PortfolioProject..CovidDeaths
Where location = 'Nigeria'
Order by 1,2

## 3 Looking at Total Cases vs Population
--Shows the percentage of population with covid in Nigeria

Select location, date, population, total_cases, (total_cases/population) * 100 as CovidCasesPct 
From PortfolioProject..CovidDeaths
Where location = 'Nigeria'
Order by 1,2

## 4 Countries with highest infection rate compared to population

Select location, population, MAX(total_cases) as HighestInfectionCount, MAX((total_cases/population)) * 100 as PctPopulationInfected
From PortfolioProject..CovidDeaths
Group by location, population
Order by PctPopulationInfected desc

## 5 Shows Countries with highest death count per population

Select location,  MAX(cast(total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
Where continent is not null
Group by location, population
Order by TotalDeathCount desc


## 6 Showing Continents with highest death count per population

Select continent,  MAX(cast(total_deaths as int)) as TotalDeathCount
From PortfolioProject..CovidDeaths
Where continent is not null
Group by continent
Order by TotalDeathCount desc


# Global numbers 

Select date, sum(new_cases) as total_cases, sum(cast(new_deaths as int)) as total_deaths, sum(cast(new_deaths as int))/sum(new_cases)*100 as DeathPct 
From PortfolioProject..CovidDeaths
Where continent is not null
Group by date 
Order by 1,2

## 7 Lets look at Total Population vs Vaccinations
-- Showing the Percentage of Population that have received at least one Covid vaccine

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
Sum(cast(vac.new_vaccinations as int)) OVER (Partition by dea.location order by dea.location, dea.date) as RollingPeopleVaccinated
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
Where dea.continent is not null
Order by 2,3


## 8 Using CTE to perform calculation on Partition By in previous query 

With PopvsVac (Continent, location, Date, Population,new_vaccinations, RollingPeopleVaccinated)
as
(
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
Sum(cast(vac.new_vaccinations as int)) OVER (Partition by dea.location order by dea.location, dea.date) as RollingPeopleVaccinated
-- Sum(CONVERT(int, vac.new_vaccinations)) OVER (Partition by dea.location)
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
Where dea.continent is not null
--Order by 2,3
)
Select * , (RollingPeopleVaccinated/Population)*100 
From PopvsVac


## 9 Using Temp table to perform calculation on Partition By in previous query

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
From PortfolioProject..CovidDeaths dea
	Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 

	
Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated


## 10 Creating view to store data for later visualisations

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 

End


