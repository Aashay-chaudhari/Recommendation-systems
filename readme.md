I created this project to understand the workings of recommendation systems based on the following methods: content-based filtering, collaborative filtering, and knowledge-based filtering. 

The dataset I have used is https://www.kaggle.com/datasets/rounakbanik/the-movies-dataset, which consists of 50,000 movies and more than 2 Million ratings by various users. The task of this project was to create a recommendation engine capable of recommending new movies to users based on their previous ratings of different films. 

**Approach/ Steps:**

Preprocessing the datasets (Movies and Ratings): 

**-> Movies Dataset: **

First, we must decide which attributes to select from the movie dataset to create the movie feature matrix. I have decided to use movie genres as the movie feature attributes. 

I performed some cleanup steps like dropping movies with no genres specified, with primary languages other than English. I could also drop some movies with a popularity (movie attribute from dataset) of less than 1 to make sure only popular movies are recommended to the user, but this would create some kind of bias. 

For both of the above-mentioned preprocessing steps, we can use SQL using the where condition that allows us to leverage Spark (distributed data processing). 

Next, I converted the genre column/feature into it's one hot encoded form which results in an appropriate movie-feature matrix. 

**-> User Ratings Dataset:** 

After preprocessing the movies dataset, some movies have been dropped based on above mentioned conditions, we need to accordingly drop user ratings from this dataset that contain ratings about the dropped movies. 

After this, we use the pivot_table functionality of pandas to create a user-movie matrix, with users as rows and movies as columns. 

After the preprocessing steps, we are ready with the user-movies matrix and the movie-feature matrix. 

The Algorithm (Content-Based Filtering): 

As the name suggests, content-based filtering refers to recommendations based on the content(attributes of products/movies). 

Let's suppose we have 50 users and 100 movies. They have rated n number of movies. 

Let's take one user vector, which will be in the form of 1*100. Most of these values will be 0 in this vector, as the user will rate significantly fewer movies than the total movie pool. 

Then we will substitute the genre value (represented by 1 in the movie-feature matrix) with the rating given to the movie by the user. This will result in a movie matrix for each user with an appropriate genre column for each movie substituted by the user. 

Once we have this intermediate matrix, we will sum over all rows to understand the preferential genre of each user, represented as a user vector. This user vector will be of the form 1*20 (as there are 20 genres).

Then we proceed to normalize this user vector. Once we have a normalized user vector, we understand what kind of genres the user likes, based on values associated with the user vector in different positions (as each position corresponds to a genre).

Once we have this user vector, we have an option of what we want to do. We can recommend based on content-based or collaborative filtering based on this user vector. 

**Content-based filtering: **

We multiply this user vector of the form (1*100) with each movie present in the movie feature matrix (100*20). This will result in a movie score vector of the form (1*20) * (20*100) = (1*100). These are the corresponding scores of each movie with the user, where a higher score means a greater chance of the user liking this movie. We need to create a mask to disable the already rated movies by the user, and then we return the top n (user-specified, depending on how many items to recommend) movies with the highest scores to the users. 

**Observations: **

-> This method faces a cold start problem when the user has no ratings or when a new user is created. 

Solution: We can create a knowledge-based recommendation system. In this method, we explicitly ask the user to directly rate their preference of genres, or ask them to select their favorite movies. 

  -> If they provide us with their genre preferences, we can readily create corresponding user vectors, and map that against the movie-feature matrix to get movie scores and recommend movies with the highest scores. Here, no masking needs to be applied. 
  -> If they provide us with their favorite movies, we create user vectors following the content-based filtering method above, where we will substitute genre values of corresponding movies in the movie-feature matrix, add all rows of the resulting matrix, normalize the   resulting vector, and treat it as a user vector to rank all other movies. Here, we will apply a mask to recommend new movies to the user. 

-> When a new movie is added, we can directly add it to the movie feature matrix, and calculate the user scores for only that movie. This will result in a computation of the magnitude (num_of_users * num_of_genres). 

**Collaborative filtering:** 

In this method, we do not need a movie-feature matrix. We just need the user-movie interaction matrix and we can suggest movies a user has not seen based on the similarity between the user to different users or the similarity between the items he has liked to different items. In the first case, we would create a user vector, compute cosine similarities between different users, and then take the top-most rated item/movie by this similar user and recommend it to this user. In the second case, we would create a movie vector from the movies this user has rated, and get the most similar movie vectors from the user-movie matrix. Once we have these movie vectors, we can suggest the most relevant movies to the user. 


Advantages and Disadvantages of Content-Based and Collaborative Filtering: 

**Content-Based Advantages:** 

-> This method can recommend less popular items/movies that are relevant to the users' interests.

-> This method is independent of the number of users, which makes it highly scalable. 

**Content-Based Disadvantages: **

-> This method can not enhance the user's interest in other categories/ genres as all the recommendations are based on pre-specified user interests captured by the user vector.

-> The item features (genres in this case) have to be hand-engineered and must be comparable. This takes a lot of domain knowledge and is very crucial as all the recommendations are based on the scores calculated from this matrix. 

**Collaborative Filtering Advantages:** 

-> It can increase the scope of a user's interests, as other products can be recommended to the user with whom he has not previously interacted. 

-> No domain knowledge about item features is required, which makes this method a great starting point and robust.

**Collaborative Filtering Disadvantages:**

-> New items are less likely to be recommended, as they have very few user ratings. 
