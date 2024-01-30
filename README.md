# 🏈    Using Python to calculate Fantasy Football player values (VORP)

[![](https://img.shields.io/badge/Python-3.8.1-FFD43B?style=for-the-badge&logo=python&logoColor=blue)](https://www.python.org/downloads/)
[![](https://img.shields.io/badge/Spyder%20IDE-838485?style=for-the-badge&logo=spyder%20ide&logoColor=maroon)](https://www.spyder-ide.org/)

## Intializing/Setting things up ##

Import and clean NFL 2023 Season Week 1-17 Half-PPR player stats downloaded from FantasyPros
```
import pandas as pd
import seaborn as sns
sns.set_style('whitegrid')

pd.set_option("display.max_columns",200)
pd.set_option("display.max_rows",1000)
pd.set_option('display.width', 1000)

###

qb_df = pd.read_csv("https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Player%20Stats%20and%20Data%20from%20FantasyPros/2023_NFL_Quarterback_Stats.csv")
rb_df = pd.read_csv("https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Player%20Stats%20and%20Data%20from%20FantasyPros/2023_NFL_Running_Back_Stats.csv")
wr_df = pd.read_csv("https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Player%20Stats%20and%20Data%20from%20FantasyPros/2023_NFL_Wide_Receiver_Stats.csv")
te_df = pd.read_csv("https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Player%20Stats%20and%20Data%20from%20FantasyPros/2023_NFL_Tight_End_Stats.csv")
k_df = pd.read_csv("https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Player%20Stats%20and%20Data%20from%20FantasyPros/2023_NFL_Kicker_Stats.csv")
dst_df = pd.read_csv("https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Player%20Stats%20and%20Data%20from%20FantasyPros/2023_NFL_DST_Stats.csv")

###

frames = [qb_df, rb_df, wr_df, te_df, k_df, dst_df]
final_df = pd.concat(frames)
points_column = final_df.pop('FPTS')
final_df.insert(2, 'FPTS', points_column)
final_df = final_df.sort_values('FPTS', ascending=False)
final_df.sort_values('FPTS', ascending=False, inplace=True)
final_df = final_df.loc[(final_df['G'] > 1)]
final_df.reset_index(drop=True, inplace=True)
```

## VORP ##
**VORP**, which stands for *value over replacement player*, is a metric that is used to determine how valuable individual players are
relative to one another. There are many different standards and ways of calculating VORP/VOR.  
In fantasy football, VORP can be used to rank players despite them being in different skill positions.   
VORP also takes into account positional roster limits that are set in fantasy leagues.
```
print(final_df.iloc[0:12,0:3])
```
![](https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Images/points_ranks.PNG)

Looking at total fantasy points scored, one would assume that QBs should be prioritized in a draft since they make up most of the highest ranked players

## Positional Scarcity and Roster Limits ##
However, based off the common roster setting: 1 QB / 2 RB / 2 WR / 1 TE / 1 Flex / 1 K / 1 DST ,  only 1 QB can be started at a time, while up to 3 RBs or 3 WRs may be started.  
The amount of players that can be started for a given position creates what is known as positional scarcity.  
This scarcity results in the inflation of RB and WR player values because the amount of "viable" players is limited. 

To calculate how large this increase in value is, multiple the number of players in that position that can
be started, with the amount of people within the fantasy league.  
For example, in a 12 team league, it would be 12(teams) x 1(QB) = 12 baseline  
The flex position can be filled with either a WR, RB, or TE. So it's baseline value of 12 is divided and added onto the WR/RB/TE baselines.

+ WR/RB: &nbsp; &nbsp; &nbsp; 12(teams) x 2(position) + 4(flex) = 28
+ QB/K/DST:&nbsp; 12 x 1 = 12
+ TE: &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; 12 x 1 + 4 = 16

## Replacement Players and Values
These baselines, plus one, are then used to find a position's "replacement player" and their respective fantasy points scored value.  
So the QB position, which has a baseline of 12, would have a replacement player who is represented by the 13th ranked QB.   
That would be C.J. Stroud with 260 points.
```
qb_vor_cutoff = qb_df[:13]
rb_vor_cutoff = rb_df[:29]
wr_vor_cutoff = wr_df[:29]
te_vor_cutoff = te_df[:17]
k_vor_cutoff = k_df[:13]
dst_vor_cutoff = dst_df[:13]


df_vor_cutoff = [rb_vor_cutoff, qb_vor_cutoff, wr_vor_cutoff, te_vor_cutoff, k_vor_cutoff, dst_vor_cutoff]
df_vor_cutoff = pd.concat(df_vor_cutoff)
df_vor_cutoff.sort_values(by='FPTS', ascending=False, inplace=True)

###

replacement_players = {
    'QB':'',
    'RB':'',
    'WR':'',
    'TE':'',
    'K': '',
    'DST': ''
    }

for _, row in df_vor_cutoff.iterrows():
    position = row['POS']
    player = row['Player']
    
    if position in replacement_players:                
        replacement_players[position] = player

print(replacement_players)
```


![](https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Images/replacement_players.PNG)


```
replacement_values = {}    
        
for position, player_name in replacement_players.items():
    player = final_df.loc[final_df['Player'] == player_name.strip()]        
    replacement_value = player['FPTS'].tolist()[0]
    replacement_values[position] = replacement_value

print(replacement_values)
```


![](https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Images/replacement_values.PNG)


## Total Points vs VOR

A formula is then used to subtract, based off of their position, the replacement players fantasy points from the points of every player to get their final VOR score.

```
final_df['VOR'] = final_df.apply(lambda row: row['FPTS'] - replacement_values[row['POS']], axis =1)

vor_column = final_df.pop('VOR')

final_df.insert(3, 'VOR', vor_column)

final_df.sort_values(by='VOR', ascending=False, inplace=True)
final_df.reset_index(drop=True, inplace=True)

print(final_df.iloc[0:12,0:4])
```
![](https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Images/vor_ranks.PNG)

As you can see, Josh Allen is only 4th in total VOR despite being ranked 1st in total fantasy points. Dak Prescott who was 5th in total points isn't in the top 12 of VOR.


## Expectations vs Reality

Patrick Mahomes was the fantasy consensus #1 QB in pre-season rankings. Josh Allen was the #1 QB in VOR.   
In the previous season, Miles Sanders was selected for the NFL Pro Bowl and started in the Super Bowl.   
Christian Mccaffrey was #1 in VOR while Austin Ekeler was consensus #2 RB in pre-season rankings.   
Justin Jefferson was rated the #1 overall fantasy draft pick. Tyreek Hill was the #2 WR in VOR and pre-season rankings.


```
import numpy as np
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker

###
final_df.index[final_df['Player'] == 'Josh Allen (BUF)']
final_df.index[final_df['Player'] == 'Miles Sanders (CAR)']
final_df.index[final_df['Player'] == 'Christian McCaffrey (SF)']
final_df.index[final_df['Player'] == 'Austin Ekeler (LAC)']
final_df.index[final_df['Player'] == 'Justin Jefferson (MIN)']
final_df.index[final_df['Player'] == 'Tyreek Hill (MIA)']

###

names = ['Mahomes','Allen','Sanders','Mccaffrey','Ekeler', 'Jefferson', 'Tyreek']
y_pos = np.arange(len(names))
player_vor = [final_df.loc[43,'VOR'],final_df.loc[3,'VOR'], final_df.loc[269,'VOR'], final_df.loc[0,'VOR'], final_df.loc[114,'VOR'], final_df.loc[161,'VOR'], final_df.loc[2,'VOR']]
colors = ["red" if i < 0 else "blue" for i in player_vor]

###

plt.bar(y_pos, player_vor, align='center', color = colors)

ax = plt.gca()
ax.set_xticklabels([])
ax.grid()

ax.yaxis.set_major_locator(ticker.MultipleLocator(25))
ax.spines["bottom"].set_position(("data", 0))
ax.spines["top"].set_visible(False)
ax.spines["right"].set_visible(False)


label_offset = 0.5
for players_names, (x_position, y_position) in zip(names, enumerate(player_vor)):
    if y_position > 0:
        label_y = -label_offset
    else:
        label_y = y_position - label_offset
    ax.text(x_position, label_y, players_names, ha="center", va="top", fontsize=8)

plt.ylabel('VOR')
plt.show()
```
![](https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Images/player_vor_graph.png)


## Kickers and Defense

Compared to their overall low rankings and late average draft position(ADP), Kicker and DST VOR seem quite high.   
That's because it doesn't take into account injury risk which further increases the inherent value of skill position players.
```
final_df.sort_values(by='VOR', ascending=False, inplace=True)

qb_missed = qb_df[:24]
rb_missed = rb_df[:24]
wr_missed = wr_df[:24]
te_missed = te_df[:24]
k_missed = k_df[:24]

###

missed_games = {
    'QB' : len(qb_missed.loc[(qb_missed['G'] < 16)]),
    'RB' : len(rb_missed.loc[(rb_missed['G'] < 16)]),
    'WR' : len(wr_missed.loc[(wr_missed['G'] < 16)]),
    'TE' : len(te_missed.loc[(te_missed['G'] < 16)]),
    'K'  : len(k_missed.loc[(k_missed['G'] < 16)])
    }

print(missed_games)

```
![](https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Images/missed_games.PNG)

Of the top 24 players by VOR in each position, **only 2 kickers** missed playing at least one game during the fantasy season(17 weeks).   
The other 4 positions each had 11 to 12 players that missed a minimum of one week.

Other reasons for this disconnect between ADP and VOR include:
+ The best kicker and defense have a lower potential VOR ceiling compared to the top players in other skill positions.
+ People feel that predicting the top kicker and defense for the whole season is harder than predicting for other positions
+ Many believe that K/DST points are extremely matchup dependent and are willing to replace them each week using the waiver wire

## Final Thoughts
Everyone drafts differently.

Draft rankings are not absolute; player values dynamically change as a draft progresses. They start at different draft positions, they might not follow the same expert lists, they have their own biases towards players, and they have to meet their own roster requirements.   
Someone who begins their first 3 draft picks as: RB/RB/RB, would prioritize a different position for their 4th pick.   
And If the highest ranked player left for your next pick is a TE, it wouldn't make much sense to get him if you already have 2 TEs.

Although VORP is a good metric for ranking fantasy players, when using pre-season points projections or post-season stats, it is not a direct 1:1 representation of what player draft rankings should be.   
