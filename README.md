<h1>Netflix Content Strategy & Data Analysis Project</h1>

<p>This project involves a comprehensive exploratory data analysis (EDA) of Netflix's library. Using PostgreSQL, we transform raw data into actionable insights, handling complex string manipulations, date conversions, and multi-valued attributes like cast and country to answer critical business questions.</p>

<h2>1. Database and Schema Design</h2> <p>The foundation of the project requires setting up a structured environment in PostgreSQL. The schema is specifically designed to accommodate various content attributes, including large text fields for extensive cast lists and plot descriptions.</p>

<pre> -- Netflix Project Database Setup CREATE TABLE netflix ( show_id VARCHAR(10), type VARCHAR(15), title VARCHAR(150), director VARCHAR(250), casts VARCHAR(1000), country VARCHAR(150), date_added VARCHAR(50), release_year INT, rating VARCHAR(10), duration VARCHAR(15), listed_in VARCHAR(100), description VARCHAR(300) ); </pre>

<h2>2. Business Problems and SQL Solutions</h2>

<p>Below are the 15 business problems solved using advanced SQL techniques including Window Functions, CTEs, and String Manipulation.</p>

<h3>Problem 1: Count the Number of Movies vs TV Shows</h3> <p>Determines the distribution of content types available on the platform.</p> <pre> SELECT type, COUNT(show_id) as total_content FROM netflix GROUP BY type; </pre>

<h3>Problem 2: Find the Most Common Rating for Movies and TV Shows</h3> <p>Uses the <code>RANK()</code> window function to identify the most frequent rating assigned to each content type.</p> <pre> SELECT type, rating FROM ( SELECT type, rating, COUNT(show_id), RANK() OVER (PARTITION BY type ORDER BY COUNT(*) DESC) as Ranking FROM netflix GROUP BY 1, 2 ) as t1 WHERE ranking = 1; </pre>

<h3>Problem 3: List All Movies Released in a Specific Year (e.g., 2020)</h3> <pre> SELECT * FROM netflix WHERE release_year = 2020 AND type = 'Movie'; </pre>

<h3>Problem 4: Find the Top 5 Countries with the Most Content</h3> <p>Since countries are often comma-separated, we unnest the array to count each country individually.</p> <pre> SELECT UNNEST(string_to_array(country, ',')) AS new_country, COUNT(show_id) as content_from_country FROM netflix GROUP BY new_country ORDER BY 2 DESC LIMIT 5; </pre>

<h3>Problem 5: Identify the Longest Movie</h3> <p>Uses a CTE to clean the duration string into a numeric format for comparison.</p> <pre> WITH movietable AS ( SELECT title, REPLACE(duration, ' min', '')::INT AS duration_minutes FROM netflix WHERE type = 'Movie' AND duration IS NOT NULL ) SELECT title, duration_minutes FROM movietable ORDER BY duration_minutes DESC LIMIT 1; </pre>

<h3>Problem 6: Find Content Added in the Last 5 Years</h3> <pre> SELECT title, date_added FROM netflix WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'; </pre>

<h3>Problem 7: Find All Content by Director 'Rajiv Chilaka'</h3> <pre> SELECT title, director FROM netflix WHERE director ILIKE '%Rajiv Chilaka%'; </pre>

<h3>Problem 8: List All TV Shows with More Than 5 Seasons</h3> <p>Uses <code>SPLIT_PART</code> to extract the numeric value from the duration string.</p> <pre> SELECT title, duration FROM netflix WHERE type = 'TV Show' AND SPLIT_PART(duration, ' ', 1)::numeric > 5; </pre>

<h3>Problem 9: Count Content Items in Each Genre</h3> <pre> SELECT UNNEST(string_to_array(listed_in, ',')) AS genre_name, COUNT(show_id) AS content_count FROM netflix GROUP BY 1; </pre>

<h3>Problem 10: Average Number of Content Releases in India per Year</h3> <pre> SELECT release_year, COUNT(*) as total_releases FROM netflix WHERE country LIKE '%India%' GROUP BY release_year ORDER BY release_year DESC; </pre>

<h3>Problem 11: List All Documentaries</h3> <pre> SELECT * FROM netflix WHERE listed_in ILIKE '%Documentaries%'; </pre>

<h3>Problem 12: Find All Content Without a Director</h3> <pre> SELECT * FROM netflix WHERE director IS NULL; </pre>

<h3>Problem 13: Find Movies Featuring 'Salman Khan' in the Last 10 Years</h3> <pre> SELECT * FROM netflix WHERE casts ILIKE '%Salman Khan%' AND release_year > EXTRACT(YEAR FROM CURRENT_DATE) - 10; </pre>

<h3>Problem 14: Top 10 Actors with Highest Content Count in India</h3> <pre> SELECT UNNEST(string_to_array(casts, ',')) AS actor_name, COUNT(*) AS Total_movies FROM netflix WHERE country LIKE '%India%' AND type = 'Movie' GROUP BY actor_name ORDER BY total_movies DESC LIMIT 10; </pre>

<h3>Problem 15: Content Categorization Based on Keywords</h3> <p>Categorizes content as 'Bad' or 'Good' based on the presence of specific sensitive keywords in the description.</p> <pre> WITH content_categorizer AS ( SELECT , CASE WHEN description ILIKE '%Kill%' OR description ILIKE '%Violence%' THEN 'Bad_Content' ELSE 'Good_Content' END category FROM netflix ) SELECT category, COUNT() AS total_content FROM content_categorizer GROUP BY 1; </pre>

<h2>Conclusion</h2> <p>This analysis successfully transforms a flat Netflix dataset into a multi-dimensional insight engine. By applying advanced PostgreSQL techniques—such as Common Table Expressions (CTEs), array unnesting, and string manipulation—we have identified peak production years, key international markets, and the most influential talent within the library.</p>
