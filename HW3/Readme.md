# Loading script

```
df = pd.read_csv('../data2/Sdf16_1a.txt', sep='\t')
language_df = pd.read_csv('../data2/rla-achievement-lea-sy2016-17.csv')
language_df = language_df[language_df['ALL_RLA00PCTPROF_1617'].str.contains('PS') == False]
```

# Problem 1

#### Rank and visualize the states that take in the most federal funding (revenue). Which states spend the most federal funding per student?

```
states = df[['STNAME', 'TFEDREV', 'TOTALEXP', 'V33']]
states = states.groupby(['STNAME']).sum().reset_index().sort_values(by='TFEDREV')
states['TotalPerStudent'] = states['TOTALEXP'] / states['V33']
plt.figure(figsize=(10,12))
plt.barh(states['STNAME'], states['TFEDREV'], align='center', alpha=0.7)
y_pos = np.arange(len(states))
plt.yticks(y_pos, states['STNAME'])
plt.xlabel('Funding (in Billions)')
plt.title('Funding per states')
plt.savefig('Q1.png')
plt.show()
print('Top 10 states with highest spending per students are')
states.sort_values(by='TotalPerStudent', ascending=False).head(10).reset_index()[['STNAME', 'TotalPerStudent']]
```

![Q1-A](Q1-A.png)
![Q1-B](Q1-B.png)

# Problem 2

#### Visualize the relationship between school districts’ total revenue and expenditures. Which states have the most debt per student?

```
districts = df[['LEAID', 'NAME','TOTALREV', 'TOTALEXP', '_41F', '_66V', 'V33']]
districts = districts[districts['TOTALREV'] > 0]
districts = districts[districts['V33'] > 0]
districts['SpendRate'] = districts['TOTALEXP'] / districts['TOTALREV']
districts['TotalDebt'] = districts['_41F'] + districts['_66V']
districts['TotalDebtPerStudent'] = districts['TotalDebt'] / districts['V33']
plt.figure(figsize=(8,5))
plt.hist(districts['SpendRate'].values, bins=np.arange(0, 2, 0.1))
plt.xticks(np.arange(0, 2, 0.1))
plt.xlabel('Ratio of Expenditure to Revenue')
plt.ylabel('Number of Districts')
plt.title('Distribution of relationship between Total Revenue and Expenditure')
plt.savefig('Q2.png')
plt.show()
print('Top 10 states with most debt spending per students are')
districts.sort_values(by='TotalDebtPerStudent', ascending=False).head(10) \
                .reset_index()[['NAME', 'TotalDebtPerStudent']]
```

![Q2-A](Q2-A.png)
![Q2-B](Q2-B.png)

# Problem 3

#### The district-level performance metrics from EDFacts may be useful in your decision. However, to protect student privacy, the data in these datasets has been heavily “blurred” to prevent students from being identified. Therefore, most of the numeric metrics are presented as ranges in string format. In addition, censored and missing data must be imputed. Write and explain a function for processing a single column of “blurred” metrics into usable numeric values. Use it to process and then visualize the distribution of a performance metric of your choice.

```
def parseScore(sc):
    try:
         return int(sc)
    except:
        if '-' in sc:
            x = sc.split('-')
        elif 'LE' in sc or 'LT' in sc:
            x = [0, sc[2:]]
        else:
            x = [100, sc[2:]]
        return (int(x[0]) + int(x[1])) // 2
language_df['ParsedScores'] = language_df['ALL_RLA00PCTPROF_1617'].apply(parseScore)
plt.figure(figsize=(8,5))
plt.hist(language_df['ParsedScores'])
plt.xlabel('Percentage of students in the school that scored at or above proficient')
plt.ylabel('Number of Districts')
plt.title('Distribution of Perecentage of students who scored at or above proficient')
plt.savefig('Q3.png')
plt.show()
```

![Q3](Q3.png)

# Problem 4

#### You are tasked with cutting 15% of the U.S. federal budget currently being spent on funding school districts. How much money is this? Choose which school districts will have their funding cut and how this will be done. (You should produce a table of LEA IDs and the dollar amount by which their federal funding will be cut you do not need print the entire table.)

```
budget = np.sum(df['TFEDREV'].values)
cut = int(budget * 0.15)
print('Total Budget : ' + str(budget))
print('15% of Total Budget : ' + str(cut))
cutting_df = pd.merge(df[['LEAID', 'TFEDREV']], language_df[['LEAID', 'ParsedScores']], on='LEAID')
cutting_df = cutting_df[cutting_df['TFEDREV'] > 0]
print('Total Districts : ' + str(cutting_df.shape[0]))
top = cutting_df[cutting_df['ParsedScores'] > \
                   (np.mean(cutting_df['ParsedScores']) - np.std(cutting_df['ParsedScores']))]
print('Districts with percentage of proficent students 1 standard diviation away from mean : '  \
      + str(top.shape[0]))
top['Rate'] = top['TFEDREV'] * top['ParsedScores']
total = np.sum(top['Rate'])
top['Normalized'] = top['Rate'] / total
top['BudgetCut'] = round(top['Normalized'] * cut, 2)
top['NewBudget'] = top['TFEDREV'] - top['BudgetCut']
top[['LEAID', 'BudgetCut', 'NewBudget']].head(20)
```

Total Budget : 55602739138 <br/>
15% of Total Budget : 8340410870 <br/>
Total Districts : 7217 <br/>
Districts with percentage of proficent students greater than 1 standard diviation less than from mean : 5946 <br/>
![Q4](Q4.png)

# Problem 5

#### Provide a statement for your supervisor justifying your decisions on which school districts will lose funding.

Since the budget plays an important factor in functioning of school, penalising schools with lower score would not be the right thing to do. However, schools already doing well can have some budget cut and over all effect would not be as high.

Hence, we take all the schools who's percentage of proficient student fall above a threshold, in our case 1 standard deviation below the mean, and reduce the budget as a weighted result of total current budget and percentage of proficient student which makes sure schools with higher budget and higher score are penalised more than schools with lower budget and higher score. Similarly lower budget and lower scores are also penalised less.
