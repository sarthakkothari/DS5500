# Loading script and helper functions

```
df = pd.read_csv('../data2/Sdf16_1a.txt', sep='\t')
df.head()
language_df = pd.read_csv('../data2/rla-achievement-lea-sy2016-17.csv')
language_df = language_df[language_df['ALL_RLA00PCTPROF_1617'].str.contains('PS') == False]
language_df.head()
membership = pd.read_csv('../data2/ccd_lea_052_1516_w_1a_011717.csv')
membership.head()

disable = pd.read_csv('../data2/ccd_lea_002089_1516_w_1a_011717.csv')
disable.head()

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

budget = np.sum(df['TFEDREV'].values)
cut = int(budget * 0.15)
cutting_df = pd.merge(df[['LEAID', 'TFEDREV']], language_df[['LEAID', 'ParsedScores']], on='LEAID')
cutting_df = cutting_df[cutting_df['TFEDREV'] > 0]
top = cutting_df[cutting_df['ParsedScores'] > \
                   (np.mean(cutting_df['ParsedScores']) - np.std(cutting_df['ParsedScores']))]
top['Rate'] = (top['TFEDREV'] * 0.25) + (top['ParsedScores'] * 0.75)
total = np.sum(top['Rate'])
top['Normalized'] = top['Rate'] / total
top['BudgetCut'] = round(top['Normalized'] * cut, 2)
top['NewBudget'] = top['TFEDREV'] - top['BudgetCut']

def autolabel(rects):
    """
    Attach a text label above each bar displaying its height
    """
    for rect in rects:
        height = rect.get_height()
        ax.text(rect.get_x() + rect.get_width()/2., height-0.1,
                height,
                ha='center', va='bottom')
```

# Problem 1

#### For the districts you selected for budget cuts in HW 3 Problem 4, calculate and visualize the proportion of each district’s total funding that will be lost. Which districts will be affected by your budget cuts the most?

```
top['BudgetCutProp'] = top['BudgetCut'] / top['TFEDREV']
plt.hist(top['BudgetCutProp'])
plt.title('Distribution of Budget Cuts')
plt.xlabel('Proportion of Budget Cut')
plt.ylabel('Counts')
plt.savefig('Q1.png')
plt.show()
```

![Q1-A](Q1.png)
![Q1-B](Q1-B.png)

# Problem 2

#### A common problem with purely data-driven solutions is that they can inadvertently perpetuate hidden pre-existing biases in the data, and further disadvantage groups that are already disadvantaged. Calculate the proportion of enrolled students by race for each district, then visualize the distributions of these for districts that received budget cuts versus districts that did not receive budget cuts. Comment on whether the the distributions appear to be the same or different. Did your selection include any hidden biases, or manage to avoid them?

```
budget_cuts = pd.merge(df, top, on='LEAID', how='left')
budget_cuts['BudgetCutProp'] = budget_cuts['BudgetCutProp'].apply(lambda x: 0 if pd.isna(x) else x)

q2 = pd.merge(budget_cuts, membership, on='LEAID')
q2.head()

q2_nocut = q2[q2['BudgetCutProp'] == 0]
q2_cut = q2[q2['BudgetCutProp'] > 0]

q2_nocut = q2_nocut[['V33', 'AM', 'AS','HI', 'BL', 'WH', 'HP', 'TR', 'TOTAL']].sum()
q2_nocut['BLP'] = q2_nocut['BL'] / q2_nocut['TOTAL']
q2_nocut['AMP'] = q2_nocut['AM'] / q2_nocut['TOTAL']
q2_nocut['HPP'] = q2_nocut['HP'] / q2_nocut['TOTAL']
q2_nocut['ASP'] = q2_nocut['AS'] / q2_nocut['TOTAL']
q2_nocut['WHP'] = q2_nocut['WH'] / q2_nocut['TOTAL']
q2_nocut['HIP'] = q2_nocut['HI'] / q2_nocut['TOTAL']
q2_nocut['TRP'] = q2_nocut['TR'] / q2_nocut['TOTAL']

q2_cut = q2_cut[['V33', 'AM', 'AS','HI', 'BL', 'WH', 'HP', 'TR', 'TOTAL']].sum()
q2_cut['BLP'] = q2_cut['BL'] / q2_cut['TOTAL']
q2_cut['AMP'] = q2_cut['AM'] / q2_cut['TOTAL']
q2_cut['HPP'] = q2_cut['HP'] / q2_cut['TOTAL']
q2_cut['ASP'] = q2_cut['AS'] / q2_cut['TOTAL']
q2_cut['WHP'] = q2_cut['WH'] / q2_cut['TOTAL']
q2_cut['HIP'] = q2_cut['HI'] / q2_cut['TOTAL']
q2_cut['TRP'] = q2_cut['TR'] / q2_cut['TOTAL']

plt.figure(figsize=(10,5))
plt.subplot(1, 2, 1)
plt.bar(q2_nocut.index[9:], q2_nocut[9:])
plt.title('No Budget Cut')
plt.xlabel('Race')
plt.ylabel('Race Proportion')


plt.subplot(1, 2, 2)
plt.bar(q2_cut.index[9:], q2_cut[9:], color = 'orange')

plt.title('Budget Cut')
plt.xlabel('Race')
plt.ylabel('Race Proportion')

plt.savefig('Q2.png')
plt.show()
```

![Q2](Q2.png)
My method for selection for budget cuts does include a bias, BLP is significantly less affected by the budget cuts and WHP is affected significantly more by budget cut.

# Problem 3

#### Calculate the proportion of enrolled students by disability status (students with an IEP under IDEA) for each district, then visualize the distributions of these proportions for districts that received budget cuts versus districts that did not receive budget cuts. Comment on whether the the distributions appear to be the same or different. Did your selection include any hidden biases, or manage to avoid them?

```
q3 = pd.merge(budget_cuts, disable, on='LEAID')
q3.head()

q3_nocut = q3[q3['BudgetCutProp'] == 0]
q3_cut = q3[q3['BudgetCutProp'] > 0]


q3_nocut = q3_nocut[['V33', 'SPECED']]
q3_cut = q3_cut[['V33', 'SPECED']]


q3_nocut['NOSPECED'] = q3_nocut['V33'] - q3_nocut['SPECED']
q3_cut['NOSPECED'] = q3_cut['V33'] - q3_cut['SPECED']

q3_nocut = q3_nocut.sum()
q3_cut = q3_cut.sum()

q3_nocut['NOSPECEDR'] = q3_nocut['NOSPECED'] / q3_nocut['V33']
q3_nocut['SPECEDR'] = q3_nocut['SPECED'] / q3_nocut['V33']

q3_cut['NOSPECEDR'] = q3_cut['NOSPECED'] / q3_cut['V33']
q3_cut['SPECEDR'] = q3_cut['SPECED'] / q3_cut['V33']

fig, ax = plt.subplots(figsize=(10,5))
x = plt.bar(['No Cut' , 'Cut'], [q3_nocut.values[3], q3_cut.values[3]])
y = plt.bar(['No Cut' , 'Cut'], [q3_nocut.values[4], q3_cut.values[4]])


autolabel(x)
autolabel(y)
plt.legend([x,y], ['Not Disabled', 'Disabled'])
plt.savefig('Q3.png')
plt.show()
```

![Q3](Q3.png)

There are almost negligible bias that my selection method includes when it comes to student with disabilities

# Problem 4

#### Choose and critique one of your fellow classmates’ selection of schools for budget cuts in HW 3 Problem 4 and Problem 5. What was the justification of their selection? Discuss any advantages or disadvantages of their approach.

I choose to critique Omair Shafi Ahmed's apporach. Omari uses simple apporach to reduce 20% of the highest funded school till the overall goal is achieved. This apporach seems pretty straightforward and seems like the correct thing to do, it might lead to various pitfall's like, since the expenditure cost is not taken into the district already has high cost of operation hence the high budget or the high funded schools need those funds to provide necessary facilities for certain students

# Problem 5

#### Summarize and comment on what you learned from one the special topics lectures (MapReduce + Hadoop, Visualization, Causal Inference, or the Industry Panel) of your choice.

I'd like to summarize the MapReduce + Hadoop special topics lecture. That lecture gave us a complete insight on the MapReduce enviornment on hadoop, its history and how it has lead to the developments in the field of Big Data. The lecture gave us an idea of how parallelism is important in the Data industry and how it has evolved over time
