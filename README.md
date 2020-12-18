# nbaunicorn

import streamlit as st 
from nba_api.stats.static import teams
from nba_api.stats.static import players
from nba_api.stats.endpoints import commonplayerinfo
from nba_api.stats.endpoints import shotchartdetail
from nba_api.stats.endpoints import playercareerstats
from nba_api.stats.endpoints import leaguestandings
from nba_api.stats.endpoints import playercareerstats
from csv import reader
import csv 
import json
import datetime
import numpy as np 
import pandas as pd 
import matplotlib.pyplot as plt 
import seaborn as sns 
from matplotlib import cm 
from matplotlib.patches import Circle, Rectangle, Arc, ConnectionPatch
from matplotlib.patches import Polygon

st.sidebar.title(":basketball: National Basketball Association")
page = st.sidebar.selectbox("Please Select One",('Team Contract Situation','NBA Contracts','Player Information', 'Player Statistics', 'Player Shot Chart', 'NBA History', 'Franchise History','Draft History','League Standings','About'))

if page == 'Team Contract Situation':
    st.title('View Team Cap Situation')

    location = teams.get_teams()
    for teams in location:

    	team_names = [l['full_name'] for l in location]
    	team_names = sorted(team_names)

    team_names.insert(0, "Select or Type a Team")

    hello = st.selectbox(
        ' ',
        (team_names))

    if hello == "Select or Type a Team":
        st.write('')
    else: 
        st.write(f'Here is the contract situation for the {hello}. Salary Cap info located at bottom.')
        st.write("* Player Option")
        st.write('** Team Option')

    players = []
    with open ('nbaplayersalaries.csv', 'r') as read_obj:
        csv_reader = reader(read_obj)
        for row in csv_reader:
            if row[2] == hello:  
                players.append({"League Rank":row[0],"Player":row[1],"2020-2021":row[3], "2021-2022":row[4], "2022-2023":row[5], "2023-2024":row[6], "2024-2025":row[7], "2025-26": row[8], "Signed Using":row[9], "Guarenteed":row[10]})          
    st.table(players)
    
    players = []
    with open ('capspace.csv', 'r') as read_obj:
        csv_reader = reader(read_obj)
        for row in csv_reader: 
            if row[0] == hello: 
                players.append({"Team":row[0],"2020-2021":row[1], "2021-2022":row[2],"2022-2023":row[3],"2023-2024":row[4],"2024-2025":row[5],"2025-2026":row[6]})
    
    if hello == "Select or Type a Team":
        st.write('')
    else: 
        st.write(f'Here is the {hello} total team cap in the upcoming seasons.') 
    
    st.table(players)

    if hello == "Select or Type a Team":
        st.write('')
    elif 109140000 -  float(players[0]['2020-2021'].replace('$', '').replace(',', ''))   > 0 :  
        st.write('They have ' + '${:,.0f}'.format(109140000-float(players[0]['2020-2021'].replace('$', '').replace(',', '')) ) + ' avaiable.')
        st.write('NBA Salary Cap for 2020-21 is $109.140M. Tax Level is $132.627M.') 
    else:
        st.write('They are currently over the cap by ' + '${:,.0f}'.format(float(players[0]['2020-2021'].replace('$', '').replace(',', '')) - 109140000))
        st.write('The NBA Salary Cap for 2020-21 is $109.140M. The Luxury Tax Level is $132.627M.')

elif page == 'NBA Contracts':
    st.title("League Wide Salary and Contract Details")
    st.write("Please choose a player to view contract details or explore list")
    st.write("* Player Option")
    st.write('** Team Option')

    player_dict = players.get_players()

    league = players.get_players()
    for player in league:

        player_information = [l['full_name'] for l in league]

    player_information.insert(0, "Select or Type a Player")

    name = st.selectbox(
        ' ',
        (player_information))

    players = []

    with open ('nbaplayersalaries.csv', 'r') as read_obj:
        csv_reader = reader(read_obj)
        for row in csv_reader:
            players.append({"Player Name":row[1],"Team":row[2],"2020-2021":row[3],"2021-2022":row[4], "2022-2023":row[5], "2023-2024":row[6], "2024-2025":row[7], "2025-2026":row[8], "Signed Using":row[9], "Guarenteed":row[10]})

    st.table(players)

elif page == 'Player Information': 
    st.title('Player Information')

    player_dict = players.get_players()

    league = players.get_players()
    for player in league:

   		player_information = [l['full_name'] for l in league]

    player_information.insert(0, "Select or Type a Player")

    name = st.selectbox(
        ' ',
        (player_information))

    if name == "Select or Type a Player":
        st.write('')
    else:
        st.write('')

    #name = st.text_input("Enter a player name")
    bron = [player for player in player_dict if player['full_name'].lower() == name.lower()]
    if len(bron) > 0:
        bron = bron[0]

    # get players information 
        player_stats = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).player_headline_stats.get_json())['data'][0][4]
        st.write(f'Stats: {player_stats}')
        all_star_appearences = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).player_headline_stats.get_json())['data'][0][6]
        st.write(f'All Star Appearences: {all_star_appearences}')
        birthday = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][7]
        st.write(f'Birthday: {birthday}')
        college = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][8]
        st.write(f'Education: {college}')
        drafted = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][29]
        drafted_round = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][30]
        drafted_number = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][31]
        st.write(f'Drafted: {drafted} Round {drafted_round} Pick {drafted_number}')
        country = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][9]
        st.write(f'Nationality: {country}')
        current_team = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][19]
        st.write(f'Most Recent Team: {current_team}')
        status = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][16]
        st.write(f'Career Status: {status}')
        height = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][11]
        st.write(f'Height: {height}')
        weight = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][12]
        st.write(f'Weight: {weight}')
        position = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][15]
        st.write(f'Position: {position}')
        season_exp = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][13]
        st.write(f'Experience: {season_exp} Seasons')
        season_exp = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][24]
        season_now = json.loads(commonplayerinfo.CommonPlayerInfo(player_id=bron['id']).get_json())['resultSets'][0]['rowSet'][0][25]
        st.write(f'Active From: {season_exp} to {season_now}')

elif page == 'Player Statistics': 
    st.title('Player Statistics')

    player_dict = players.get_players()

    league = players.get_players()
    for player in league:

    	player_information = [l['full_name'] for l in league]

    player_information.insert(0, "Select or Type a Player")

    name = st.selectbox(
        ' ',
        (player_information))

    scoring_type = st.selectbox(
        'Please choose between Totals, PerGame, and Per36',
        ('Totals', 'PerGame', 'Per36'))

    bron = [player for player in player_dict if player['full_name'].lower() == name.lower()]
    if len(bron) > 0:
        bron = bron[0]  

    #build empty dictionary
    players = []

    career_stats = json.loads(playercareerstats.PlayerCareerStats(player_id=bron['id'], per_mode36=scoring_type).get_json())['resultSets'][0]['rowSet']
    for row in career_stats:
        players.append({'Year':row[1], 'Team': row[4], 'Age': row[5], 'Points': row[26], 'Rebounds':row[20], 'Assists':row[21]})

    newlist = sorted(players, key=lambda k: k['Year'], reverse=True)

    if name == "Select or Type a Player":
        st.write('')
    else: 
        st.table(newlist)

elif page == 'Player Shot Chart': 
    st.title("NBA Shot Chart By Season")

    years = []
    for year in range (2019,1995, -1):
        years.append(str(year) + "-" + (str(year+1)[-2:]))

    years.insert(0, "Select a year")
    jonnyissexy = st.selectbox(
        'Please pick a year. Note: NBA provides shot data for every player in the league for each season since 1996-97',
        years)

    season_type = st.selectbox(
        'Please choose between Regular Season or Playoffs. Default is Regular Season',
        ('Regular Season', 'Playoffs'))

    #get player shot chart detail
    def get_player_shotchartdetail(player_name, season_id, season_type): 

        #player dictionary 
        nba_players = players.get_players()
        player_dict = [player for player in nba_players if player['full_name'] == player_name]

        if len(player_dict) == 0: 
            return None, None
    
        #career dataframe
        career = playercareerstats.PlayerCareerStats(player_id=player_dict[0]['id'])
        career_df = json.loads(career.get_json())['resultSets'][0]['rowSet']

        #team id during the season
        team_ids = [season[3] for season in career_df if season[1] == season_id]

        #st.write(career_df[0])
        shots = []
        for team_id in team_ids: 
            shotchartlist = shotchartdetail.ShotChartDetail(team_id=int(team_id),
                                                    player_id=int(player_dict[0]['id']),
                                                    season_type_all_star=season_type,
                                                    season_nullable=season_id,
                                                    context_measure_simple="FGA").get_data_frames()
            shots.extend(shotchartlist)

        return shotchartlist[0], shotchartlist[1]

    #draw court function
    def draw_court(ax=None, color="blue", lw=1, outer_lines=False):

            if ax is None:
                ax = plt.gca()

            #Basketball Hoop
            hoop = Circle((0,0), radius=7.5, linewidth=lw, color=color, fill=False)

            #backboard 
            backboard = Rectangle((-30, -12.5), 60, 0, linewidth=lw, color=color)

            #the paiint 
            #outer box 
            outer_box = Rectangle((-80, -47.5), 160, 190, linewidth=lw, color=color, fill=False)
            #inner box 
            inner_box = Rectangle((-60, -47.5), 120, 190, linewidth=lw, color=color, fill=False)

            #freethrow top arc 
            top_free_throw = Arc((0, 142.5), 120, 120, theta1=0, theta2=180, linewidth=lw, color=color, fill=False)

            #freethrow bottom arc 
            bottom_free_throw = Arc((0, 142.5), 120, 120, theta1=180, theta2=0, linewidth=lw, color=color)

            #restricted zone 
            restricted = Arc((0,0), 80, 80, theta1=0, theta2=180, linewidth=lw, color=color)
        
            #three point line 
            corner_three_a = Rectangle((-220, -47.5), 0, 140, linewidth=lw, color=color)
            corner_three_b = Rectangle((220, -47.5), 0, 140, linewidth=lw, color=color)
            three_arc = Arc((0,0), 475, 475, theta1=22, theta2=158, linewidth=lw, color=color)

            #center court 
            center_outer_arc = Arc((0, 422.5), 120, 120, theta1=180, theta2=0, linewidth=lw, color=color)
            center_inner_arc = Arc((0, 422.5), 40, 40, theta1=180, theta2= 0, linewidth=lw, color=color)

            court_elements = [hoop, backboard, outer_box, inner_box, top_free_throw, bottom_free_throw, restricted, corner_three_a, corner_three_b, three_arc, center_outer_arc, center_inner_arc]

            outer_lines=True
            if outer_lines:
                outer_lines = Rectangle((-250, -47.5), 500, 470, linewidth=lw, color=color, fill=False)
                court_elements.append(outer_lines)

            for element in court_elements:
                ax.add_patch(element)

    #shot chart function 
    def shot_chart(data, title="", color="b", xlim=(-250, 250), ylim=(422.5,-47.5), line_color="blue",
                court_color="white", court_lw=2, outer_lines=False,
                flip_court=False, gridsize=None,
                ax=None, despine=False):

        if (ax is None):
            ax = plt.gca()

        if not flip_court:
            ax.set_xlim(xlim)
            ax.set_ylim(ylim)
        else:
            ax.set_xlim(xlim[::-1])
            ax.set_ylim(ylim[::-1])

        ax.tick_params(labelbottom="off", labelleft="off")
        ax.set_title(title, fontsize=16)

        #draws the court using the draw_court()
        draw_court(ax, color=line_color, lw=court_lw, outer_lines=outer_lines)

        # seperate color by make or miss
        x_missed = data[data['EVENT_TYPE'] == 'Missed Shot']['LOC_X']
        y_missed = data[data['EVENT_TYPE'] == 'Missed Shot']['LOC_Y']

        x_made = data[data['EVENT_TYPE'] == 'Made Shot']['LOC_X']
        y_made = data[data['EVENT_TYPE'] == 'Made Shot']['LOC_Y']

        #plot missed shots 
        ax.scatter(x_missed, y_missed, c='r', marker="x", s=100, linewidths=3)
        #plot made shots 
        ax.scatter(x_made, y_made, facecolors='none', edgecolors='g', marker='o', s=100, linewidths=3)

    league = players.get_players()
    for player in league:

    	player_names = [l['full_name'] for l in league]

    player_names.insert(0, "Select a player")

    name = st.selectbox(
        'Please select or type a players name to view their shot chart from the season choosen above',
        player_names)

    player_dict = players.get_players()

    if __name__ == "__main__":
        player_shotchart_df, league_avg = get_player_shotchartdetail(name, jonnyissexy, season_type)
    
        if player_shotchart_df is None:
            st.write(" ")
        else:
            shot_chart(player_shotchart_df, title=name + " " + jonnyissexy)
            plt.show()

        #print(player_shotchart_df)
        #print(league_avg)
            xlim = (-250, 250)
            ylim = (422.5, -47.5)

            ax = plt.gca()
            ax.set_xlim(xlim[::-1])
            ax.set_ylim(ylim[::-1])
            draw_court(ax)
            plt.show()

    st.set_option('deprecation.showPyplotGlobalUse', False)
    st.pyplot()

elif page == 'NBA History':
    st.title('NBA History')

elif page == 'Franchise History':
    st.write('draft history')
    st.write('franchise history')
    st.write('team leaders')
    st.write('team info')
    st.write('team year by year')
    st.write('awards')

elif page == 'Draft History':
    st.title('Draft History')

elif page == 'League Standings':
    st.title("League Standings")

    years = []
    for year in range (2019,1969, -1):
        years.append(str(year) + "-" + (str(year+1)[-2:]))

    jonnyissexy = st.selectbox(
        'Note: NBA.com Only Provides Regular Season Standings Dating Back to 1970-71. Some Data is Only Available After 2002-03 Season',
        years)
    
    conference_type = st.selectbox(
        'Please choose between League Wide, Eastern Conference, or Western Conference',
        ('League Wide', 'East', 'West'))

    st.write('Indicators:')
    st.write('Clinched - ', 'E: east,', 'W: west,', 'NW: northwest,', 'P: pacific,', 'SW: southwest,', 'A: atlantic,', 'C: central,', 'SE: southeast')
    st.write('Misc - ', 'X: earned playoff berth,', 'O: eliminated from playoffs,', 'PI: play-in game')

    #build empty dictionary
    teams = []

    standings_league = json.loads(leaguestandings.LeagueStandings(season=jonnyissexy).get_json())['resultSets'][0]['rowSet']
    for row in standings_league: 
        if row[5] == conference_type:
            teams.append({'Team':row[3] + " " + row[4], 'Conference': row[5], 'Record':row[16], 'Home Record':row[17], 'Away Record':row[18], 'Last 10': row[19], 'Clinch Indicator':row[8], 'Playoff Seed':row[7], 'PPG':row[56],'Opp PPG':row[57], 'Point Diff.':row[58],'Conference Record': row[6],  'Longest Winning Streak':row[29], 'Longest Losing Streak':row[30]})
        elif conference_type == "League Wide":
            teams.append({'Team':row[3] + " " + row[4], 'Conference': row[5], 'Record':row[16], 'Home Record':row[17], 'Away Record':row[18], 'Last 10': row[19],'Clinch Indicator':row[8], 'Playoff Seed':row[7], 'PPG':row[56],'Opp PPG':row[57], 'Point Diff.':row[58],'Conference Record': row[6],  'Longest Winning Streak':row[29], 'Longest Losing Streak':row[30]})

    #ppg = 'PPG':row[56]
    #ppg_no_zero = str(round(ppg, 2))

    newlist = sorted(teams, key=lambda k: k['Record'], reverse=True)

    newlist.insert(0,{'Team':"", 'Conference':"", 'Record':"", 'Home Record':"", 'Away Record':"",'Last 10':"", 'Clinch Indicator':"", 'Playoff Seed':"",'PPG':"",'Opp PPG':"", 'Point Diff.':"",'Conference Record':"", 'Longest Winning Streak':"", 'Longest Losing Streak':""})

    st.table(newlist)
else: 
    st.title('What can I do with this app?')
    st.write('Made by Josh with help from Jonny')
    st.write('Contract Data Courtesy of basketballreference.com')


from nba_api.stats.library import http
print(http.STATS_HEADERS)
