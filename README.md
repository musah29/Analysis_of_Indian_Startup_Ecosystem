# Analysis_of_Indian_Startup_Ecosystem
Uncovering Trends and Opportunities in the Indian Startup Ecosystem.

Introduction

In this project, Python is used to analyze and visualize industry data and identify key trends and opportunities in the Indian startup scene. The analysis will cover funding trends, the geographic distribution of the start-ups, funding sources, and the industrial sector in which the start-ups operate. The insights gained from this project will help potential investors stay ahead of the curve and identify promising investment opportunities. 
This post outlines the steps taken in analyzing the data and the insights gained. I will share code snippets where relevant and visualizations to aid in understanding the results obtained.

The data

The datasets covered the years 2018–2021 in a CSV file format. I set out to answer five questions that will be relevant to any would-be investor interested in venturing into the Indian startup ecosystem or potential startups interested in trying their luck. The information included in the dataset includes:

Company/Brand,
Amount($),
Stage,
Investors,
Industry,
Location, 
Funding year.

Data understanding

The first step is to study the data to understand what is available and what is not. This provided two key insights:

What is the quality of the data available?

What insights can be gleaned from data after cleaning?

Hypothesis testing

The next step I wanted to understand was whether the location of a startup had an impact on its ability to attract funding.

Hypothesis: The location of a start-up has an impact on the amount of funding it is able to secure.

Null hypothesis: The location of a start-up has no significant influence on the amount of funding it raises.

Alternate hypothesis: The location of a start-up significantly influences the amount of funding it raises.

# get the unique values in the 'headquarter' column
headquarters = df_startup['HeadQuarter'].unique()

# create a list to store the data for each group
data = []

# loop over the unique values and get the data for each group
for hq in headquarters:
    data.append(df_startup.loc[df_startup['HeadQuarter'] == hq, 'Amount($)'])

# perform the ANOVA test
f_statistic, p_value = f_oneway(*data)

# print the results
print('F-statistic:', f_statistic)
print('p-value:', p_value)
if p_value < 0.05:
    print('There is a significant difference between the groups.')
else:
    print('There is no significant difference between the groups.')
Output


Data cleaning

I started by inspecting each data set separately to identify any problems that might be present. Some of the challenges with the datasets are as follows:

The [Location] column of the 2018 dataset contained the city, state, and country names of the startups concerned. This was stripped down leaving the city name only which was relevant for the analysis.
The names of some of the columns in the 2018 dataset were changed to make them consistent with those of 2019,2020 and 2021.
The [Amount] column in all datasets had to be changed from an object datatype into a float for ease of analysis.
The [Amount] column had Indian Rupees, US dollars, and figures with no designated currency symbol. This was corrected by stripping away the currency symbols and converting all Indian Rupee entries to US dollar equivalents.
#2018 Exchange rate Rupee to a Dollar.
exchange_rate = 68.14

def rupee_to_dollar(amount):
    if isinstance(amount, str) and amount.startswith('₹'):
        amount = float(amount.replace(',', '')[1:]) * exchange_rate
        return f'${amount:.2f}'
    else:
        return amount

# remove rupee sign and comma, and convert to dollar equivalent
df_startup['Amount($)'] = df_startup['Amount($)'].apply(lambda x: rupee_to_dollar(x))

The Stage column some of the stages are spelled slightly differently, these need to be harmonised to a single spelling.

The [Sector] column, had a mismatch of names, the same sectors were spelled differently. Related sectors were then grouped for ease of analysis.

The [Year] column was an object datatype. This was converted to a year to make it useful for analysis.


f_startup['Year']=pd.to_datetime(df_startup.Year)
#Convert the datetime column to year
df_startup['Year'] = df_startup['Year'].dt.year

After cleaning and harmonizing all four datasets, I merged them to create a new DataFrame:

df_startup=pd.concat([su_18,su_19,su_20,su_21],ignore_index=True)

pd.set_option('display.max_rows', None)
pd.set_option('display.max_columns', None)


The five (5) questions I sought to answer:

1. What is the funding trend in the Indian startup ecosystem?
2018 was a really good year for startups in India, as they attracted their highest investment to date. The 25th Oct 2018 edition of the Economic Times online magazine indicated a 108% increase in startup funding in India. After 2018, the flow of investment slumped drastically, further analysis is required to understand what happened from 2019 to 2021.

funding_by_year = df_startup.groupby('Year')['Amount($)'].sum().apply(lambda x: '${:,.2f}'.format(x))
df_funding_by_year = pd.DataFrame({'Year': funding_by_year.index, 'Amount($)': funding_by_year.values})
df_funding_by_year['Amount($)'] = df_funding_by_year['Amount($)'].apply(lambda x: float(x.replace(',', '').replace('$', '')))
df_funding_by_year

sns.set_style(style='whitegrid')
plt.figure(figsize=(7, 3))
plt.title('Total Funding Raised')
sns.barplot(x='Year', y='Amount($)', palette='dark', data=df_funding_by_year, hue='Year')
plt.show()

2. Which sectors have received the most funding year on year?

There are a plethora of sectors in the startup ecosystem, I want to understand which industries are receiving the most funding in a given year.

funding_by_year_sector = df_startup.groupby(['Year','Sector'])['Amount($)'].sum()

funding_by_year_sector = funding_by_year_sector.groupby('Year').idxmax()

for year, sector in funding_by_year_sector.items():
   print(f"{year}: {sector[1]}")

3. Who are the top investors and what Sectors do they typically invest in?

I want to understand who are the top players on the scene. I also seek to understand the sectors they invest in, this will help potential investors decide which sectors to either invest in or not.

funding_by_investor = df_investor.groupby(['Investor','Sector'])['Amount($)'].sum().reset_index()
sorted_funding_by_investor = funding_by_investor.sort_values(by='Amount($)', ascending=False)
top_investors = sorted_funding_by_investor.head(5)

top_investors

plt.figure(figsize=(5,4))
plt.title('Top 5 Investors by Funding Amount')
sns.set_style(style='whitegrid')
top_investors['Amount($)'] = pd.to_numeric(top_investors['Amount($)'])
sns.barplot(x='Amount($)',y='Investor', palette='dark', data=top_investors, orient='h')
plt.show()

4. Where are the start-ups located and in what industries?

This question seeks to understand where the startups are located and whether there is a clustering effect among the startups. That is startups in similar or same industries set up close to one another to take advantage of skills or other kinds of resources required for that specific industry.

startup_location = df_startup.groupby(['HeadQuarter','Sector'])['Company/Brand'].count().reset_index(name='Number_of_firms')
startup_location = startup_location.sort_values(by='Number_of_firms', ascending=False).head(10)

startup_location

plt.figure(figsize=(7,4))
plt.title('Where are the startups located?')
sns.set_style(style='whitegrid')
sns.barplot(x='Number_of_firms',y='HeadQuarter', palette='dark', data=startup_location, orient='h')
plt.show()

Bengaluru has the highest number of startups in India by headline aggregate. Further drill down into the data indicates clustering effect-where firms in similar or the same industries sit close to one another. This pattern is noted in the concentration of FinTechs, EdTech's, Financial Services, Healthcare, and Information Technology startups in the city. This is consistent with the image of Bengaluru as the ‘Silicon Valley of India’.

5. Which location has attracted the most funding?

funds_by_location = df_startup.groupby(['HeadQuarter'])['Amount($)'].sum().reset_index(name='Total_Raised')
funds_by_location = funds_by_location.sort_values(by='Total_Raised', ascending=False).head(5)
funds_by_location['Total_Raised'] = funds_by_location['Total_Raised'].apply(lambda x: '${:,.2f}'.format(x))

funds_by_location

plt.figure(figsize=(7,4))
plt.title('Where did the money go?')
sns.set_style(style='whitegrid')
funds_by_location['Total_Raised'] = pd.to_numeric(funds_by_location['Total_Raised'].astype(str).str.replace('$', '').str.replace(',', ''))
sns.barplot(x='Total_Raised',y='HeadQuarter', palette='dark', data=funds_by_location, orient='h')
plt.show()

The top five destinations for startup investment are Mumbai, Haryana, Delhi, Bengaluru, and Jaipur. Mumbai is the largest Indian city by population and received the highest amount of investment. The data has to be further drilled down to identify the sectors the startups that received the investment belong to.










