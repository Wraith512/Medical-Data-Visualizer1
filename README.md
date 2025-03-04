# Medical-Data-Visualizer1
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt

# Demographic Data Analyzer
def demographic_data_analyzer():
    df = pd.read_csv("adult.data.csv")
    
    # Count race representation
    race_count = df['race'].value_counts()
    
    # Average age of men
    average_age_men = df[df['sex'] == 'Male']['age'].mean()
    
    # Percentage with Bachelor's degree
    percentage_bachelors = (df['education'] == 'Bachelors').mean() * 100
    
    # Higher education
    higher_edu = df['education'].isin(['Bachelors', 'Masters', 'Doctorate'])
    higher_edu_rich = (df[higher_edu]['salary'] == '>50K').mean() * 100
    lower_edu_rich = (df[~higher_edu]['salary'] == '>50K').mean() * 100
    
    # Min work hours & high earners in min work hours
    min_hours = df['hours-per-week'].min()
    rich_percentage = (df[df['hours-per-week'] == min_hours]['salary'] == '>50K').mean() * 100
    
    # Country with highest % of >50K earners
    country_stats = df[df['salary'] == '>50K']['native-country'].value_counts() / df['native-country'].value_counts() * 100
    highest_earning_country = country_stats.idxmax()
    highest_earning_country_percentage = country_stats.max()
    
    # Most common occupation in India for >50K earners
    top_IN_occupation = df[(df['native-country'] == 'India') & (df['salary'] == '>50K')]['occupation'].mode()[0]
    
    return {
        'race_count': race_count,
        'average_age_men': average_age_men,
        'percentage_bachelors': percentage_bachelors,
        'higher_education_rich': higher_edu_rich,
        'lower_education_rich': lower_edu_rich,
        'min_work_hours': min_hours,
        'rich_percentage': rich_percentage,
        'highest_earning_country': highest_earning_country,
        'highest_earning_country_percentage': highest_earning_country_percentage,
        'top_IN_occupation': top_IN_occupation
    }

# Medical Data Visualizer
def medical_data_visualizer():
    df = pd.read_csv("medical_examination.csv")
    
    # Add overweight column
    df['overweight'] = (df['weight'] / (df['height'] / 100) ** 2) > 25
    df['overweight'] = df['overweight'].astype(int)
    
    # Normalize cholesterol & glucose
    df[['cholesterol', 'gluc']] = (df[['cholesterol', 'gluc']] > 1).astype(int)
    
    # Categorical plot
    df_cat = pd.melt(df, id_vars=['cardio'], value_vars=['cholesterol', 'gluc', 'smoke', 'alco', 'active', 'overweight'])
    df_cat = df_cat.groupby(['cardio', 'variable', 'value']).size().reset_index(name='total')
    cat_plot = sns.catplot(x='variable', y='total', hue='value', col='cardio', data=df_cat, kind='bar')
    cat_plot.savefig('catplot.png')
    
    # Heatmap
    df_heat = df[(df['ap_lo'] <= df['ap_hi']) &
                 (df['height'].between(df['height'].quantile(0.025), df['height'].quantile(0.975))) &
                 (df['weight'].between(df['weight'].quantile(0.025), df['weight'].quantile(0.975)))]
    
    corr = df_heat.corr()
    mask = np.triu(np.ones_like(corr, dtype=bool))
    
    fig, ax = plt.subplots(figsize=(12, 8))
    sns.heatmap(corr, mask=mask, annot=True, fmt='.1f', cmap='coolwarm', ax=ax)
    fig.savefig('heatmap.png')

demographic_data_analyzer()
medical_data_visualizer()
