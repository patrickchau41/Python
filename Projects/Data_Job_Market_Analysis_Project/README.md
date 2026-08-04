# The Overview
Welcome to my analysis of the data job market, focusing on data analyst roles. As a data analyst myself, this project was created out in order to learn and  understand the data job market more effectively. This project dives into top-paying and in-demand skills to help find optimal and balanced job opportunities for data analysts. 

The data provides a foundation for my analysis, containing detailed information on job titles, salaris, locations, and essential skills. Through a series of Python scripts, I explore key questions such as the most in-demand skills, salary trends, and the intersection of demand and salary in data analytics. 
# The Questions
Below are the questions I want to answer in my project:
1. What are the most demanded skills for the top 3 most popular data roles?
2. How are in-demand skills trending for Data Analysts?
3. How well do jobs and skills pay for Data Analysts?
4. What are the optimal skills for data analysts to learn? (High Demand AND High Paying)
# Tools I Used
For my deep dive into the data analyst job market, I imported several key tools:
- Python: The backbone of my analysis, allowing me to analyze the data and find critical insights. I also used the following Python libraries:
    - Pandas Library: To perform my analysis. 
    - Matplotlib Library: To visualize my data. 
    - Seaborn Library: To create more advanced and sophisticated visualizations.
- Jupyter Notebook: Used to run my Python scripts, including my analysis and commentary notes. 
- Visual Studio Code: Where I execute my Python scripts.
- Git & GitHub: Essential for version control and sharing my Python code and analysis, ensuring collaboration.
# Data Preparation and Cleanup
This section outlines the data cleaning steps taken to prepare the data for analysis, ensuring accuracy, readability, and usability.
## Import & Cleaning Data
I start by importing necessary libraries and loading the dataset, followed by initial data cleaning tasks to ensure 100% data accuracy.
# The Analysis

## 1. What are the most demanded skills for the top 3 most popular data roles?

I filtered out those positions by the top 3 most popular, and got the top 5 skills for these top 3 titles. This query highlights the most popular job titles and their top skills, showcasing which skills I should pay attention to depending on the role that I am targeting.

View my noteook with detailed steps here:
[Skill_Demand.ipynb](Projects/Project2/Skill_Demand.ipynb)

### Visualize Data

``` python
fig, ax = plt.subplots(len(job_titles), 1)


for i, job_title in enumerate(job_titles):
    df_plot = df_skills_perc[df_skills_perc['job_title_short'] == job_title].head(5)[::-1]
    sns.barplot(data=df_plot, x='skill_percent', y='job_skills', ax=ax[i], hue='skill_count', palette='dark:b_r')

plt.show()
```


### Results

![Visualization of Top Skills for Data Nerds](Projects/Project2/images/skill_demand_all_dataroles.png)


### Insights

- Python is a versatile skill, highly demanded across all three roles, but most prominently for its peaks for both Data Scientists (72%) and Data Engineers (65%).
- SQL is the most requested skill for Data Analysts and Data Scientists, with it in over half the job postings for both roles. For Data Engineers, Python is the most sought-after skill, appearing in 68% of job postings.
- Data Engineers require more specialized technical skills (AWS, Azure, Spark) compared to Data Analysts and Data Scientists who are expected to be proficient in more general analytical tools (Excel, Tableau).


## 2. How are in-demand skills trending for Data Analysts?

### Visualize Data

``` python

from matplotlib.ticker import PercentFormatter

df_plot = df_DA_US_percent.iloc[:, :5]
sns.lineplot(data=df_plot, dashes=False, legend='full', palette = 'tab10')

plt.gca().yaxis.set_major_formatter(PercentFormatter(decimals=0))

plt.show()

```


### Results

![Trending Top Skills for Data Analysts in the US](Projects/Project2/images/skill_trend.png)


### Insights

- SQL remains as the most consistently demanded skill throughout the year of 2023, although there is a slight, gradual decrease in likelihood demand.
- Excel experienced a significant increase in demand starting around September, surpassing both Python and Tableau by the end of the year. 
- Both Python and Tableau show relatively stable demand throughout the year with some flunctuations here and there, but remain essential skills for data analysts. Power BI, while less demanded compared to the others, shows a slight upward trend towards the year's end, suggesting a potential increase in demand.



## 3. How well do jobs and skills pay for Data Analysts?

### Salary Analysis for Data Nerds

#### Visualize Data

``` python
sns.boxplot(data=df_US_top6, x='salary_year_avg', y = 'job_title_short', order = job_order)

ticks_x = plt.FuncFormatter(lambda y, pos: f'${int(y/1000)}K')
plt.gca().xaxis.set_major_formatter(ticks_x)
plt.show()

``` 


#### Results

![Salary Distributions in the US](Projects/Project2/images/top6_boxplot.png)


#### Insights

- There's a significant variation in salary ranges across different job titles. Senior Data Scientist positions tend to have the highest salary potential, with up to $600K, indicating the high value placed on advanced data skills and experience in the industry.

- Senior Data Engineer and Senior Data Scientist roles show a considerable number of outliers on the higher end of the salary spectrum, suggesting that exceptional skills or circumstances can lead t o high pay in these roles. In contrast, Data Analyst roles demonstrate more consistency in salary, with fewer outiers.

- The median salaries increase with the seniority and specialization of the roles. Senior roles (Senior Data Scientist, Senior Data Engineer) not only have higher median salaries but also larger differences in typical salaries, reflecting greater variance in compensation as responsibilities increase.

### Highest Paid & Most Demanded Skills for Data Analysts

#### Visualize Data

``` python

fig, ax = plt.subplots(2,1)

# Top 10 Highest Paid Skills for Data Analysts
sns.barplot(data=df_DA_top_pay, x='median', y=df_DA_top_pay.index, hue='median', ax=ax[0], palette='dark:b_r)

#Top 10 Most In-Demand Skills for Data Analysts
sns.barplot(data=df_DA_top_pay, x='median', y=df_DA_top_pay.index, hue='median', ax=ax[1], palette='light:b_r)

plt.show()

```


In-Demand Skills for Data Analysts in the US:

![The Highest Paid & Most In-Demand Skills for Data Analysts in the US](Projects/Project2/images/top10barplot.png)


#### Insights:

- The top graph shows specialized technical skills like `dplyr`, `Bitbucket`, and `Github` are associated with higher salaries, some reaching up to $200K, suggesting that advanced technical proficiency can increase earning potential.

- The bottom graph highlights that foundational skills like `Excel`, `PowerPoint`, and `SQL` are the most in-demand, even though they may not offer the highest salaries. This demonstrates the importance of these core skills for employability in data analyst roles.

- There is a clear distinction between the skills that are highest paid and those that are most in-demand. Data Analysts aiming to maximize their career potential should consider developing a diverse skill set that includes both high-paying specialized skills and widely demanded foundational skills.


## 4. What is the most optimal skill to learn for Data Analysts?

#### Visualize Data

``` python
from adjustText import adjust_text
import matplotlib.pyplot as plt

plt.scatter(df_DA_skills_high_demand['skill_percent'], df_DA_skills_high_demand['median_salary'])
plt.show()

```

#### Results

![Most Optimal Skills for Data Analysts in the US](Projects/Project2/images/OptimalSkills.png)
*A scatter plot visualizing the most optimal skills (high paying & high demand) for data analysts in the US.*

#### Insights:
- The scatter plot shows that most of the `programming` skills (colored blue) tend to cluster at higher salary levels compared to other categories, indicating that programming expertise might offer greater salary benefits witin the data analytics field.

- Analyst tools (colored orange), including Tableau and Power BI, are prevalent in job postings and offer competitive salaries, showing that job visualization and data analysis software are crucial for current data roles. This category not only has good salaries but is also versatile across different types of data tasks.

- The database skills (colored green), such as Oracle and SQL Server, are associated with some of the highest salaries among data analyst tools. This indicates a significant demand and valuation for data management and manipulation expertise in the industry.