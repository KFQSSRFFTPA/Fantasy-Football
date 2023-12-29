# Using Python to calculate NFL Fantasy Football player positional values

***Spyder IDE was used for this project***


## Intializing/Setting things up ##

Import and clean Week 1-16 Half PPR player stats downloaded from FantasyPros
```
import pandas as pd

import seaborn as sns
sns.set_style('whitegrid')

pd.set_option("display.max_columns",200)
pd.set_option("display.max_rows",1000)
pd.set_option('display.width', 1000)

qb_df = pd.read_csv("https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Player%20Stats%20and%20Data%20from%20FantasyPros/2023_NFL_Quarterback_Stats.csv")
rb_df = pd.read_csv("https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Player%20Stats%20and%20Data%20from%20FantasyPros/2023_NFL_Running_Back_Stats.csv")
wr_df = pd.read_csv("https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Player%20Stats%20and%20Data%20from%20FantasyPros/2023_NFL_Wide_Receiver_Stats.csv")
te_df = pd.read_csv("https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Player%20Stats%20and%20Data%20from%20FantasyPros/2023_NFL_Tight_End_Stats.csv")

frames = [qb_df, rb_df, wr_df, te_df]
final_df = pd.concat(frames)
points_column = final_df.pop('FPTS')
final_df.insert(2, 'FPTS', points_column)
final_df = final_df.sort_values('FPTS', ascending=False)
final_df.sort_values('FPTS', ascending=False, inplace=True)
final_df = final_df.loc[(final_df['FPTS'] > 1)]
final_df.reset_index(drop=True, inplace=True)
```

## VORP ##
**VORP**, which stands for *value over replacement player*, is a metric that is used to determine how valuable individual players are
relative to one another. There are many different standards and ways of calculating VORP.  
In fantasy football, VORP can be used to rank players despite them being in different skill positions.   
VORP also takes into account positional roster limits that are set in fantasy leagues.
```
print(final_df.iloc[0:12,0:3])
```
![](https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Images/total_points_ranking.PNG)

Looking at total fantasy points scored, one would assume that, because QBs make up most of the highest ranked players, they should be prioritized in a draft.

## Positional Scarcity and Roster Limits ##
However, based off the common roster setting: 1 QB / 2 RB / 2 WR / 1 TE / 1 Flex ,  only 1 QB can be started at a time, while up to 3 RBs or 3 WRs may be started.  
The amount of players that can be started for a given position creates what is known as positional scarcity.  
This scarcity results in the inflation of RB and WR player values.

To calculate how large this increase in value is, multiple the number of players in that position that can
be started, with the amount of people within the fantasy league.  
For example, in a 12 team league, it would be 12(teams) x 1(QB) = 12 baseline  
The flex position can be filled with either a WR, RB, or TE. So it's baseline value of 12 is divided and added onto the WR/RB/TE baselines.

+ WR/RB:&nbsp; 12(teams) x 2(position) + 4(flex) = 28
+ QB: &nbsp; &nbsp; &nbsp; &nbsp; 12 x 1 = 12
+ TE:  &nbsp; &nbsp; &nbsp; &nbsp;&nbsp; 12 x 1 + 4 = 16

## Replacement Players and Values
These baselines, plus one, are then used to find a position's "replacement player" and their respective point value.  
So the QB position, which has a baseline of 12, would have a replacement player that is represented by the 13th ranked QB.
```
rb_df_vor_cutoff = rb_df[:31]
qb_df_vor_cutoff = qb_df[:13]
wr_df_vor_cutoff = wr_df[:31]
te_df_vor_cutoff = te_df[:13]

df_vor_cutoff = [rb_df_vor_cutoff, qb_df_vor_cutoff, wr_df_vor_cutoff, te_df_vor_cutoff]
df_vor_cutoff = pd.concat(df_vor_cutoff)
df_vor_cutoff.sort_values(by='FPTS', ascending=False, inplace=True)
replacement_players = {
    'RB':'',
    'QB':'',
    'WR':'',
    'TE':''
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

![](https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Images/replacement_points.PNG)

## Draft Rankings vs Final Rankings

A formula is then used to subtract, based off of their position, the replacement players fantasy points from the points of every player to get their final VOR score.

```
final_df['VOR'] = final_df.apply(lambda row: row['FPTS'] - replacement_values[row['POS']], axis =1)

vor_column = final_df.pop('VOR')

final_df.insert(3, 'VOR', vor_column)

final_df.sort_values(by='VOR', ascending=False, inplace=True)
final_df.reset_index(drop=True, inplace=True)

print(final_df.iloc[0:12,0:4])
```
![](https://raw.githubusercontent.com/KFQSSRFFTPA/Fantasy-Football/main/Images/vor_ranking.PNG)

As you can see, Josh Allen is only ranked 5th in VORP despite being ranked 1st in total fantasy points.
