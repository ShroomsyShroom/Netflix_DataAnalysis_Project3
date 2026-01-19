<h1>Netflix Content Strategy & Data Analysis Project</h1>

<p>This project involves a comprehensive exploratory data analysis (EDA) of Netflix's library. Using PostgreSQL, we transform raw data into actionable insights, handling complex string manipulations, date conversions, and multi-valued attributes like cast and country to answer critical business questions.</p>

<h2>1. Database and Schema Design</h2> <p>The initial phase requires setting up a structured environment. The schema is designed to accommodate various content attributes, including large text fields for cast lists and descriptions.</p>

<pre> -- Netflix Project Database Setup CREATE TABLE netflix ( show_id VARCHAR(10), type VARCHAR(15), title VARCHAR(150), director VARCHAR(250), casts VARCHAR(1000), country VARCHAR(150), date_added VARCHAR(50), release_year INT, rating VARCHAR(10), duration VARCHAR(15), listed_in VARCHAR(100), description VARCHAR(300) ); </pre>

<h2>2. Quantitative Content Analysis</h2> <p>Understanding the balance between different formats is essential for content acquisition strategies.</p>

<h3>Content Distribution: Movies vs. TV Shows</h3> <pre> SELECT type, COUNT(show_id) as total_content FROM netflix GROUP BY type; </pre>

<h3>Market Dominance: Top 5 Countries</h3> <p>Since the <code>country</code> column often contains multiple entries, we utilize <code>UNNEST</code> and <code>string_to_array</code> to count individual country occurrences accurately.</p> <pre> SELECT UNNEST(string_to_array(country, ',')) AS new_country, COUNT(show_id) as content_from_country FROM netflix GROUP BY new_country ORDER BY 2 DESC LIMIT 5; </pre>

<h2>3. Deep Dive into Content Metadata</h2>

<h3>Duration Analysis: Identifying the Longest Movie</h3> <p>By stripping the 'min' suffix and casting to an integer, we can perform numerical comparisons on content length.</p> <pre> WITH movietable AS ( SELECT title, REPLACE(duration, ' min', '')::INT AS duration_minutes FROM netflix WHERE type = 'Movie' AND duration IS NOT NULL ) SELECT title, duration_minutes FROM movietable ORDER BY duration_minutes DESC LIMIT 1; </pre>

<h3>Trend Analysis: Content Added in Last 5 Years</h3> <p>Using <code>TO_DATE</code>, we convert the string-based <code>date_added</code> into a proper date object to perform interval-based filtering.</p> <pre> SELECT title, date_added FROM netflix WHERE TO_DATE(date_added, 'Month DD, YYYY') >= CURRENT_DATE - INTERVAL '5 years'; </pre>

<h2>4. Targeted Search & Genre Exploration</h2>

<h3>Genre Distribution</h3> <p>Categorizing the library to see which genres dominate the streaming platform.</p> <pre> SELECT UNNEST(string_to_array(listed_in, ',')) AS genre_name, COUNT(show_id) AS content_count FROM netflix GROUP BY 1; </pre>

<h3>Talent Tracking: Top Actors in Indian Productions</h3> <p>This identifies the most frequent collaborators in specific regional markets.</p> <pre> SELECT UNNEST(string_to_array(casts, ',')) AS actor_name, COUNT(*) AS Total_movies FROM netflix WHERE country LIKE '%India%' AND type = 'Movie' GROUP BY actor_name ORDER BY total_movies DESC LIMIT 10; </pre>

<h2>5. Content Sentiment & Keywords Analysis</h2> <p>We use <code>CASE</code> statements to label content based on specific descriptors, which helps in understanding the library's content tone for safety or marketing purposes.</p>

<pre> WITH content_categorizer AS ( SELECT , CASE WHEN description ILIKE '%Kill%' OR description ILIKE '%Violence%' THEN 'Bad_Content' ELSE 'Good_Content' END category FROM netflix ) SELECT category, COUNT() AS total_content FROM content_categorizer GROUP BY 1; </pre>

<h2>Conclusion</h2> <p>The analysis reveals significant insights into Netflix's catalog, from the concentration of content in specific geographic hubs to the identification of long-standing creative partners. By leveraging PostgreSQL's array functions and CTEs, we transformed a flat dataset into a relational structure capable of complex multi-dimensional reporting.</p>
