# Recommendation System: Allrecipes.com
Recommending recipes with content-based filtering approach by feature extraction (nutrition values). 

## Data Source
Data from [Kaggle](https://www.kaggle.com/elisaxxygao/foodrecsysv1) user elisaxxygao, containing two sets of data about user interaction information and recipe information, recipe images are also provided.

## Feature Extraction
<img src="/images/nutrition_table.png" alt="nutrition_table" width=550> <br/>
For comparing similarity between recipes, 7 nutritions are selected and daily percent values are extracted which are based on a 2,000 calorie diet. 

## Distance Calculation Methods
After doing normalization for the nutrition data, three distance calculation methods are applied to experiment Top 3 recommendation results. 

<img src="/images/selected_recipe_222388.png" alt="selected_recipe_222388" width=450> <br/>

1. Cosine Distance <br/>
   ![cosine_222388](/images/cosine_222388.png)
   
2. Euclidean Distance <br/>
   ![euclidean_222388](/images/euclidean_222388.png)
   
3. Hamming Distance <br/>
   ![hamming_222388](/images/hamming_222388.png)

Nutrition data are similar and different results are generated by various distance calculation methods. Thus, a hybrid recommender is created to integrate recommendations from three approaches. 

## Hybrid Recommender
```python
"""
Hybrid Nutrition Recommender which integrates Top 2 recommendations from 3 different distance approaches 
(cosine, euclidean, hamming) and sort the results by selected criteria(s)

df_normalized: normalized nutrition data
recipe_id: find similar recipes based on the selected recipe
sort_order: must be in list, 4 options available: ['aver_rate'], ['review_nums'], ['aver_rate', 'review_nums'], ['review_nums', 'aver_rate']
N: Top N recipe(s)

return 1) recipe id, recipe name and image of Top N recommendation, 
2) nutrition data of selected recipe and Top N recommendation,
3) average rating and number of review of Top N recommendation
"""

def nutrition_hybrid_recommender(recipe_id, sort_order, N):
    start = time()
    
    allRecipes_cosine = pd.DataFrame(df_normalized.index)
    allRecipes_cosine = allRecipes_cosine[allRecipes_cosine.recipe_id != recipe_id]
    allRecipes_cosine["distance"] = allRecipes_cosine["recipe_id"].apply(lambda x: cosine(df_normalized.loc[recipe_id], df_normalized.loc[x]))
    
    allRecipes_euclidean = pd.DataFrame(df_normalized.index)
    allRecipes_euclidean = allRecipes_euclidean[allRecipes_euclidean.recipe_id != recipe_id]
    allRecipes_euclidean["distance"] = allRecipes_euclidean["recipe_id"].apply(lambda x: euclidean(df_normalized.loc[recipe_id], df_normalized.loc[x]))
    
    allRecipes_hamming = pd.DataFrame(df_normalized.index)
    allRecipes_hamming = allRecipes_hamming[allRecipes_hamming.recipe_id != recipe_id]
    allRecipes_hamming["distance"] = allRecipes_hamming["recipe_id"].apply(lambda x: hamming(df_normalized.loc[recipe_id], df_normalized.loc[x]))
    
    Top2Recommendation_cosine = allRecipes_cosine.sort_values(["distance"]).head(2).sort_values(by=['distance', 'recipe_id'])
    Top2Recommendation_euclidean = allRecipes_euclidean.sort_values(["distance"]).head(2).sort_values(by=['distance', 'recipe_id'])
    Top2Recommendation_hamming = allRecipes_hamming.sort_values(["distance"]).head(2).sort_values(by=['distance', 'recipe_id'])
    
    recipe_df = recipe.set_index('recipe_id')
    hybrid_Top6Recommendation = pd.concat([Top2Recommendation_cosine, Top2Recommendation_euclidean, Top2Recommendation_hamming])
    aver_rate_list = []
    review_nums_list = []
    for recipeid in hybrid_Top6Recommendation.recipe_id:
        aver_rate_list.append(recipe_df.at[recipeid, 'aver_rate'])
        review_nums_list.append(recipe_df.at[recipeid, 'review_nums'])
    hybrid_Top6Recommendation['aver_rate'] = aver_rate_list
    hybrid_Top6Recommendation['review_nums'] = review_nums_list
    TopNRecommendation = hybrid_Top6Recommendation.sort_values(by=sort_order, ascending=False).head(N).drop(columns=['distance'])
    
    recipe_id = [recipe_id]   
    recipe_list = []
    image_list = []
    image_path = "./foodrecsysv1/raw-data-images/{}.jpg"
    for recipeid in TopNRecommendation.recipe_id:
        recipe_id.append(recipeid)   # list of recipe id of selected recipe and recommended recipe(s)
        recipe_list.append("{}  {}".format(recipeid, recipe_df.at[recipeid, 'recipe_name']))
        image_list.append(image_path.format(recipeid))
    
    image_array = []
    for imagepath in image_list:
        img = image.load_img(imagepath)
        img = image.img_to_array(img, dtype='int')
        image_array.append(img)
        
    fig = plt.figure(figsize=(15,15))
    gs1 = gridspec.GridSpec(1, N)
    axs = []
    for x in range(N):
        axs.append(fig.add_subplot(gs1[x]))
        axs[-1].imshow(image_array[x])
    [axi.set_axis_off() for axi in axs]
    for axi, x in zip(axs, recipe_list):
        axi.set_title(x)
    
    end = time()
    running_time = end - start
    print('time cost: %.5f sec' %running_time)
    return df_normalized.loc[recipe_id, :], TopNRecommendation
```

<img src="/images/selected_recipe_222886.png" alt="selected_recipe_222886" width=450> <br/>

1. Sort by average rating
   ![hybrid_ar](/images/hybrid_ar.png) <br/>
   <img src="/images/nutrition_ar.png" alt="nutrition_ar" width=550> <br/>
   <img src="/images/topN_ar.png" alt="topN_ar" width=400> 
   
2. Sort by number of reviews
   ![hybrid_rn](/images/hybrid_rn.png) <br/>
   <img src="/images/nutrition_rn.png" alt="nutrition_rn" width=550> <br/>
   <img src="/images/topN_rn.png" alt="topN_rn" width=400> 
     
Average rating and number of reviews are different popularity standards, and with these two sorting criterias similar results are generated. It is surprised that with nutrition information, even alcohol recipes can be detected and recommended. 

## Deployment
<img src="/images/hybrid_deploy.gif" alt="hybrid_deploy" width=600> <br/>
Integrate Top 10 recommendation from three distance calculation approaches, then generate Top N recommendation sorted by various criterias, e.g. average rating or number of reviews

## Detailed Presentation
* Check out complete workflow with [Jupyter Notebook](./code).
* Check out complete code of [Flask Deployment](./flask_deployment).

## Skills Acquired
* Pandas: feature extraction, data cleaning and data imputation
* Keras: image processing (process image files to array and show them according to recommended recipes)
* Matplotlib: using GridSpec to do subplots visualization within a for loop
* Flask: deployment of recommender engine into web application

## Acknowledgements
Subplots code reference from Stack Overflow user [armatita](https://stackoverflow.com/questions/46713186/matplotlib-loop-make-subplot-for-each-category?rq=1) and [Nirmal](https://stackoverflow.com/questions/25862026/turn-off-axes-in-subplots). Thank you coders for sharing your experience! =]
