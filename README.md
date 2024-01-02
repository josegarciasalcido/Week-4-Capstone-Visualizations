# Week-4-Capstone-Visualizations
import sqlite3
import pandas as pd
import requests
import tempfile

# Download the SQLite database file
url = "https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DA0321EN-SkillsNetwork/LargeData/m4_survey_data.sqlite"
response = requests.get(url)
file_content = response.content

# Create a temporary file and write the content to it
with tempfile.NamedTemporaryFile(delete=False, suffix=".sqlite") as temp_file:
    temp_file.write(file_content)
    temp_file_path = temp_file.name

# Connect to the SQLite database
conn = sqlite3.connect(temp_file_path)
query = "SELECT * FROM master;" # Query to fetch data from the 'master' table
query2 = "SELECT count(DatabaseDesireNextYear) as Count,DatabaseDesireNextYear from DatabaseDesireNextYear group by DatabaseDesireNextYear order by count(DatabaseDesireNextYear) DESC LIMIT 5"
query3 = "SELECT * from DatabaseWorkedWith"
query4 = "SELECT * from LanguageDesireNextYear"
query5 = "SELECT * from LanguageWorkedWith"
query6 = "SELECT * from DevType"

# Create DataFrames by fetching data from the SQLite database
df = pd.read_sql_query(query, conn)
df2 = pd.read_sql_query(query2,conn)
df2.set_index('DatabaseDesireNextYear',inplace=True)
df3 = pd.read_sql_query(query3, conn)
df4 = pd.read_sql_query(query4, conn)
df5 = pd.read_sql_query(query5, conn)
df6 = pd.read_sql_query(query6, conn)


import matplotlib.pyplot as plt
import seaborn as sns
import matplotlib as mpl

# HISTOGRAM for Annual Compensation
plt.figure(figsize=(10, 6))
sns.histplot(df['ConvertedComp'], color='blue', bins=30)  # Salary converted to annual USD salaries using the exchange rate on 2019-02-01.
plt.title('Annual Salaries')
plt.xlabel('Annual Compensation $USD')
plt.ylabel('Frequency')
plt.show()

# Box Plot for Age
plt.figure(figsize=(10, 6))
sns.boxplot(x=df['Age'])
plt.title('Age')
plt.xlabel('Age (Years)')
plt.show()

# Scatter Plot for Age and WorkWeekHrs
df.plot(kind='scatter', x='Age', y='WorkWeekHrs', figsize=(10, 6), color='darkblue')
plt.title('Hours of work per week by age')
plt.xlabel('Age (Years)')
plt.ylabel('WorkWeekHrs')
plt.show()

# bubble plot of WorkWeekHrs and CodeRevHrs, use Age column as bubble size.
plt.figure(figsize=(12, 8))
cmap = mpl.cm.cool
scaling_factor = 3
scatter = plt.scatter(df['WorkWeekHrs'], df['CodeRevHrs'], s=scaling_factor*df['Age'], alpha=0.5, cmap=cmap)
plt.clim(0,100)
plt.colorbar(scatter, label='Age')
plt.title('Bubble Plot of WorkWeekHrs, CodeRevHrs, and Age')
plt.xlabel('WorkWeekHrs')
plt.ylabel('CodeRevHrs')
plt.grid(True)
plt.show()

# Create pie chart for top 5 databases
labels = df2.index  # Use the index as labels
plt.figure(figsize=(10, 6))
plt.pie(df2['Count'], labels=labels, autopct='%1.1f%%', startangle=90, shadow=True, pctdistance=0.85)
plt.title('Top 5 Database Desire Next Year')
plt.axis('equal')  # Equal aspect ratio ensures that pie is drawn as a circle
plt.legend(labels, loc='upper left')
plt.show()

# in the list of most popular languages respondents wish to learn, what is the rank of python?
language_counts = df4['LanguageDesireNextYear'].value_counts().reset_index()
language_counts.columns = ['Language', 'TotalOccurrences']
RankedLanguages = language_counts.sort_values(by='TotalOccurrences', ascending=False)
print(RankedLanguages)
# How many respondents indicated that they currently work with SQL?
print("# of respondents who work with SQL is: " + str(df5['LanguageWorkedWith'].str.contains('SQL', case=False).sum()))
# How many respondents indicated that they work on MySQL only?
print("# of respondents who work with MySQL is: " + str(df3.groupby('Respondent')['DatabaseWorkedWith'].apply(lambda x: 'MySQL' in x.values and len(x) == 1).sum()))

# stacked chart of median WorkWeekHrs and CodeRevHrs for the age group 30 to 35.
Selected_Ages = df[(df['Age'] >= 30) & (df['Age'] <= 35)]
pivot_df = Selected_Ages.pivot_table(index='Age', values=['WorkWeekHrs', 'CodeRevHrs'], aggfunc='median') # Pivot the DataFrame for stacking
pivot_df.plot(kind='bar', stacked=True, figsize=(10, 6))
plt.title('Median WorkWeekHrs and CodeRevHrs for Age Group 30 to 35')
plt.xlabel('Age')
plt.ylabel('Median Hours')
plt.show()

# Plot a line chart the median ConvertedComp for all ages from 45 to 60
Selected_Ages = df[(df['Age'] >= 45) & (df['Age'] <= 60)]
median_ConvertedComp = Selected_Ages.groupby('Age')['ConvertedComp'].median() # Calculate median ConvertedComp for each age
median_ConvertedComp.plot(kind='line', x='Age', y='ConvertedComp', figsize=(10, 6), color='darkblue', marker='o')
plt.title('Median ConvertedComp for Ages 45 to 60')
plt.xlabel('Age')
plt.ylabel('Median ConvertedComp')
plt.grid(True)
plt.show()

# Plot a line chart the median ConvertedComp for all ages from 25 to 30
Selected_Ages = df[(df['Age'] >= 25) & (df['Age'] <= 30)]
median_ConvertedComp = Selected_Ages.groupby('Age')['ConvertedComp'].median()
median_ConvertedComp.plot(kind='line', x='Age', y='ConvertedComp', figsize=(10, 6), color='darkblue', marker='o')
plt.title('Median ConvertedComp for Ages 25 to 30')
plt.xlabel('Age')
plt.ylabel('Median ConvertedComp')
plt.grid(True)
plt.show()

# Create a horizontal bar chart using column MainBranch
branch_counts = df['MainBranch'].value_counts() # Count the occurrences of each value in the "MainBranch" column
plt.subplots_adjust(left=0.395)  # Adjust the left margin
branch_counts.plot(kind='barh', color='blue', figsize=(6, 6))
plt.title('Distribution of MainBranch')
plt.xlabel('Count')
plt.ylabel('MainBranch')
plt.show()

# Find the type of developers and rank them according to frequency
TypesDevel = df6['DevType'].value_counts().reset_index()
TypesDevel.columns = ['TypeOfDeveloper', 'TotalOccurrences']
RankedDevelopers = TypesDevel.sort_values(by='TotalOccurrences', ascending=False)
print(RankedDevelopers)

conn.close() # Close the database connection
