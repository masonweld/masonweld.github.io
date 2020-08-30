# Creating a Function that Outputs User Performance Statistics

The online chess website https://lichess.org/ keeps a database of all the games played on its site, which can be found at https://database.lichess.org/.  I have downloaded the portable game notation (PGN) of every game played during Feb 2017, which amounts to over 10 million chess games.  The goal is to create a function that takes in a database of chess games and a chess player's username, and outputs some relative statistics for that user based on the games played during the timeframe on lichess.org.  I originally created this to evaluate my games during different time periods, but found it to be more beneficial to allow the user to input any username to calculate.

After importing the packages we need (including the useful PGN parser chess), we can parse all the PGNs of the 10+ million chess games.


```python
import chess
import chess.pgn
```


```python
pgn = open("lichess_db_standard_rated_2017_02.pgn", 'r')
```

To get an idea of what the format is, below is the first game in the dataset printed.


```python
print(chess.pgn.read_game(pgn))
```

    [Event "Rated Classical game"]
    [Site "https://lichess.org/ntf4qW5C"]
    [Date "????.??.??"]
    [Round "?"]
    [White "cocinda"]
    [Black "mehran35"]
    [Result "0-1"]
    [BlackElo "1827"]
    [BlackRatingDiff "+6"]
    [ECO "B01"]
    [Opening "Scandinavian Defense"]
    [Termination "Normal"]
    [TimeControl "300+8"]
    [UTCDate "2017.01.31"]
    [UTCTime "23:00:01"]
    [WhiteElo "1627"]
    [WhiteRatingDiff "-6"]
    
    1. e4 d5 2. f3 d4 3. d3 e5 4. h4 Nc6 5. a3 a6 6. g3 f5 7. exf5 Bxf5 8. g4 Bd7 9. h5 Bd6 10. Be2 e4 11. f4 e3 12. f5 Bg3+ 13. Kf1 Qg5 14. Nh3 Qf6 15. Kg2 Qe5 16. c3 Nh6 17. cxd4 Nxd4 18. Nc3 Bc6+ 19. Kf1 Bxh1 20. Ne4 Bxe4 21. dxe4 Qxe4 22. Qa4+ c6 23. Ng1 Nxg4 24. Nf3 O-O-O 25. Kg2 Bf2 26. b3 Ne5 27. Bb2 Qg4+ 28. Kh2 Qg3+ 29. Kh1 Ndxf3 30. Bxe5 Qh3+ 31. Bh2 Qxh2# 0-1
    

Now we create a funciton that takes in the input of the pgn data and chess username, and parses through each game to extract the needed data for each game the user played.


```python
def eval_player(player, chess_db):
    data = { 
            "number_white_games" : 0,
            "number_black_games" : 0,
            "number_white_wins" : 0,
            "number_black_wins" : 0,
            "number_white_loss" : 0,
            "number_black_loss" : 0,
            "number_white_draw" : 0,
            "number_black_draw" : 0,
            "best_win" : 0,
            "best_win_player" : " ",
            "worst_loss" : 0,
            "worst_loss_player" : " ",
            "avg_rated_opp" : 0,
            }
    for i in range(1000000):
        match = chess.pgn.read_game(pgn)
        white_player = match.headers["White"]
        black_player = match.headers["Black"]
        if white_player == player:
            data["number_white_games"] += 1
            result = match.headers["Result"]
            data["avg_rated_opp"] += int(match.headers["BlackElo"])
            if result == "1-0":
                data["number_white_wins"] += 1
                if int(match.headers["BlackElo"]) > data["best_win"]:
                    data["best_win"] = int(match.headers["BlackElo"])
                    data["best_win_player"] = black_player                    
            elif result == "0-1":
                data["number_white_loss"] += 1
                if int(match.headers["BlackElo"]) < data["worst_loss"] or int(data["worst_loss"]) == 0:
                    data["worst_loss"] = int(match.headers["BlackElo"])
                    data["worst_loss_player"] = black_player 
            else:
                data["number_white_draw"] += 1
        elif black_player == player:
            data["number_black_games"] += 1
            result = match.headers["Result"]
            data["avg_rated_opp"] += int(match.headers["WhiteElo"])
            if result == "1-0":
                data["number_black_loss"] += 1
                if int(match.headers["WhiteElo"]) < data["worst_loss"] or int(data["worst_loss"]) == 0:
                    data["worst_loss"] = int(match.headers["WhiteElo"])
                    data["worst_loss_player"] = white_player
            elif result == "0-1":
                data["number_black_wins"] += 1
                if int(match.headers["WhiteElo"]) > data["best_win"]:
                    data["best_win"] = int(match.headers["WhiteElo"])
                    data["best_win_player"] = white_player
            else:
                data["number_black_draw"] += 1
    return(data)
```

It is also helpful to have a function that prints the results of the above function.


```python
def print_stats(data):
    total_number_games = data["number_white_games"] + data["number_black_games"]
    if total_number_games != 0:
        wins = data["number_white_wins"] + data["number_black_wins"]
        loss = data["number_white_loss"] + data["number_black_loss"]
        draws = data["number_white_draw"] + data["number_black_draw"]
        
        print("Total Number of Games Played = " + str(total_number_games))
        print("Average Rating of Opponents Played = " + str(round(data["avg_rated_opp"]/total_number_games,2)))
        print("Total Number of Wins = " + str(wins) + " (" + str(round((wins/total_number_games)*100,2)) + "%)")
        print("\t Wins as White = " + str(data["number_white_wins"]))
        print("\t Wins as Black = " + str(data["number_black_wins"]))
        print("\t Hightest Rated Win = " + data["best_win_player"] + " (" + str(data["best_win"]) + ")")
        print("Total Number of Losses = " + str(loss) + " (" + str(round((loss/total_number_games)*100,2)) + "%)")
        print("\t Losses as White = " + str(data["number_white_loss"]))
        print("\t Losses as Black = " + str(data["number_black_loss"]))
        print("\t Worst Rated Loss = " + data["worst_loss_player"] + " (" + str(data["worst_loss"]) + ")")
        print("Total Number of Draws = " + str(draws) + " (" + str(round((draws/total_number_games)*100,2)) + "%)")
        print("\t Draws as White = " + str(data["number_white_draw"]))
        print("\t Draws as Black = " + str(data["number_black_draw"]))
    else:
        print("No Games Played to Analyze")
```

I will now test the function on Grandmaster Andrew Tang, with lichess username "penguingim1".


```python
player = "penguingim1"
data = eval_player(player, pgn)
print("lichess.org Game Data for User: " + player)
print_stats(data)
```

    lichess.org Game Data for User: penguingim1
    Total Number of Games Played = 211
    Average Rating of Opponents Played = 2342.55
    Total Number of Wins = 155 (73.46%)
    	 Wins as White = 73
    	 Wins as Black = 82
    	 Hightest Rated Win = ChessJuice (2800)
    Total Number of Losses = 49 (23.22%)
    	 Losses as White = 28
    	 Losses as Black = 21
    	 Worst Rated Loss = pavelpavel (1486)
    Total Number of Draws = 7 (3.32%)
    	 Draws as White = 4
    	 Draws as Black = 3
    
