# The Simpsons Project

I am a big fan of **The Simpsons**, I have seen them since my childhood and to this day I still laught at those jokes that I heard probably a hundred times. However, even though the series is currently in its 35th season, it is no longer what it used to be. Many people attribute the cause to the new episodes, the death of those who gave their voice to the characters, or just simply boresom. That's why I decided to investigate when this successful TV series declines. 

> To note: the information in this database that we will analyze corresponds to the North American audience, and the success/failure may be different from my home country. 

*To complete this project, I will use SQlite (in this case DB browser) and Tableau to be able to visualize our results.*

# Code

    ## Insert the database, i’ll be working with this database:
   (https://www.kaggle.com/datasets/jonbown/simpsons-episodes-2016)
    
      
    
    1.  Cleaning: First of all, let's check that there are no repeated Id's.
    
    SELECT id, COUNT(*)
    FROM simpsons_episodes
    GROUP BY id
    ORDER BY id DESC;
    
    ## Let’s check for nulls on the most important columns
    SELECT
    COUNT(*) FILTER (WHERE id IS NULL) AS id_nulls,
    COUNT(*) FILTER (WHERE directed_by IS NULL) AS directed_by_nulls,
    COUNT(*) FILTER (WHERE title IS NULL) AS title_nulls,
    COUNT(*) FILTER (WHERE original_air_date IS NULL) AS original_air_date_nulls,
    COUNT(*) FILTER (WHERE written_by IS NULL) AS written_by_nulls,
    COUNT(*) FILTER (WHERE season IS NULL) AS season_nulls,
    COUNT(*) FILTER (WHERE imdb_rating IS NULL) AS imdb_rating_nulls,
    COUNT(*) FILTER (WHERE tmdb_rating IS NULL) AS tmdb_ratingnulls FROM simpsons_episodes;
    
    ## This columns should be numeric and bigger than 0 so let’s check it out
    
    SELECT id
    FROM simpsons_episodes
    WHERE us_viewers_in_millions < 0 AND imdb_rating < 0 AND tmdb_rating <0;
    SELECT id,us_viewers_in_millions, us_viewers_in_millions/1 AS us_is_numeric
    FROM simpsons_episodes
    ORDER BY us_is_numeric ASC
    
      
    SELECT id, imdb_rating, imdb_rating/1 AS imdb_is_numeric
    FROM simpsons_episodes
    ORDER BY imdb_is_numeric ASC
    
    SELECT id, tmdb_rating, tmdb_rating/1 AS tmdb_is_numeric
    FROM simpsons_episodes
    ORDER BY tmdb_is_numeric ASC
    
    ## So we have one episode without a numeric character, I’ll isolate that id for future calculations. And also there is one episode without a tmdb rating so let’s keep it mind too.
    
  
    2.  Transform: Let’s transform the database so it’s easier to work with. I’ll delete: description, number_in_series , production_code and tmdb_vote_count
    
      
 
    3.  Analysis: Time to analyze, let’s see some general numbers. Most popular episode
    
    SELECT title, season, max(us_viewers_in_millions)
    FROM simpsons_episodes
    WHERE id IS NOT '422'
      
    ## Less popular episode
    SELECT title, season, us_viewers_in_millions
    FROM simpsons_episodes
    ORDER BY us_viewers_in_millions ASC;
      
  
    ## Worst season
    SELECT season, sum(us_viewers_in_millions) as total_views
    FROM simpsons_episodes
    GROUP BY season
    ORDER BY total_views ASC;
    
    ## And best season
    SELECT season, sum(us_viewers_in_millions) as total_views
    FROM simpsons_episodes
    GROUP by season
    ORDER BY total_views DESC;
    
    ## Best rating in Imdb and Tmdb
    SELECT title, season,max(imdb_rating)
    FROM simpsons_episodes
    
    SELECT title, season,max(tmdb_rating)
    FROM simpsons_episodes;
   
    ##Worst rating in Imdb and Tmdb
    SELECT title, season,min(imdb_rating)
    FROM simpsons_episodes
    Order by imdb_rating DESC;
    
    SELECT title, season,min(tmdb_rating)
    FROM simpsons_episodes
    WHERE id IS NOT '746';
    
    ## I’m going to calculate the median. The number of views and the season may give me an idea of how much they have dropped.
    
    SELECT us_viewers_in_millions, title, season
    FROM simpsons_episodes
    ORDER BY us_viewers_in_millions
    LIMIT 2 - (SELECT COUNT(*) FROM simpsons_episodes) % 2
    OFFSET (SELECT (COUNT(*) - 1) / 2 FROM simpsons_episodes)
  
    
    ## I am also interested in seeing the best directors and writers with respect to ranking rather than episodes. Maybe that can give me a clue as to what has happened.
    
    SELECT directed_by, avg(imdb_rating) as imdb, avg(tmdb_rating) as tmdb
    FROM simpsons_episodes
    GROUP by directed_by
    ORDER BY imdb DESC, tmdb DESC;
    
    
    SELECT written_by, avg(imdb_rating) as imdb, avg(tmdb_rating) as tmdb
    FROM simpsons_episodes
    GROUP by written_by
    ORDER BY imdb DESC, tmdb DESC;
    
    ## I wonder if there will be any successful combination of writers and directors
    
    SELECT *
    FROM simpsons_episodes
    WHERE directed_by IN ('Mark Kirkland', 'Steven Dean Moore', 'Bob Anderson' )
    AND written_by IN ('Joel H. Cohen', 'John Swartzwelder','Tim Long')
    ORDER BY imdb_rating desc, tmdb_rating DESC;
    
    
    ## It seems that John Swartzwelder is one of the best writers the series has had, his chapters have good ratings.
    
    SELECT count(written_by) as count_writter, written_by, avg(imdb_rating)
    FROM simpsons_episodes
    GROUP by written_by
    ORDER BY count_direc DESC ;
   
    ## Indeed, it has many chapters and a good rating from the public. Let's see when was the last chapter he wrote.
  
    SELECT max(original_air_date), season
    FROM simpsons_episodes
    WHERE written_by = 'John Swartzwelder';
      
## Visualization
Now it is time to translate all the analysis into a visualization to make it easier to draw conclusions.

[The Simpson's Visualization on Tableau](https://public.tableau.com/app/profile/oriana.rasello/viz/TheendofaneraTheSimpsons/Dashboard1)


## Conclusions


Although the answer to the question "When have the Simpsons declined?" is somewhat subjective and everyone will probably have a different answer, I will give an objective answer based on the analysis.

I will take two axes: the North American audience and the audience ranking on Imdb and Tmdb. The first season presents the highest audience of all, even containing half as many episodes as the rest of the seasons. For the Imdb and Tmdb audience, on the other hand, the best seasons are 7, 5 and 6, placing the first season in 9th place. The best episodes of course belong to seasons 8, 5 and 2. Everything seems to indicate that **the best seasons would be from 1 to 10** for both the audience and the public. The worst episodes are located in season 23, 33 and 34.

Taking into account the median, for that episode of season 21, the audience had already fallen about 75%. This should not be surprising, since from s*eason 14 onwards it was impossible for this series to regain the amount of audience it once had, losing almost 50%*.

It may be a hypothesis to be developed, but the analysis tells us that there are certain writers who have made excellent episodes with very good audience ratings, namely *John Swartzwelder, who wrote his last episode on 2003-11-23, equivalent to season 15*.
