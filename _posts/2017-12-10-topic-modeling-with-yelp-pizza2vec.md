---
layout: post
published: true
title: 'Topic Modeling with Yelp, Pizza2vec'
---
## Topic Modeling in Yelp
One of my favorite things in this wonderful world is a good pizza. Arguably my first real love, pizza has taught me some of the most important things to be learn in life. Beyond just having incredibly easy and consistent dinner suggestions (good pizza will ALWAYS make that list), I was fortunate enough to gain some foundational business experience having managed a local pizza shop in cozy St. Cloud MN as a teen. 

I have only been to New York City once in my life and was only there for a weekend, but of course pizza had to be a focal point of the trip. Knowing where to go was really easy thanks to my graduate school percussion teacher at the time who tipped me off to such greats as Lombardos, World of Pizza, and Angelos. In addition to spending time with friends, the awesome food made the trip that much more memorable and fun. I love checking out local pizza spots when I travel, but the problem is that I am usually really busy when I am traveling so I don’t have time to scourge through yelp reviews to find the most relevant reviews for what I want at a given time and place.

So I wondered, could I create a clustered topic model for pizza reviews on Yelp? It turns out Yelp offers a slice of their data for academic purposes as well as an API. To test the pizza waters in Yelp land, I am going to test out their academic dataset first to see if I can prototype something useful before considering  a more robust solution.


### Read the data into Python

The data that was used for the post can be downloaded from the following URL [Yelp Dataset](https://www.yelp.com/dataset/challenge "10th Iteration Data").

The data is available in both SQL and JSON, I chose to download the JSON version and read the data into python with JSON package I used codecs to iterate through the json files and write to txt files. Instead of topic modeling on the entire roughly 3 million review restaurants, I chose instead to subset to reviews that contained the word pizza. 

For the purpose of this blog I am going to skip over the code to clean the data, but have the build code for this project available in my github account HERE.

Next it is necessary to prepare the text data for meaningful results for our model.

### Preparing the text for modeling

Text data can be very highly dimensional. If proper steps are not taken to clean the text data then one should not expect to have a very accurate or useful model. To clean the data, I  tokenized the spaces and punctions, lemmatized the data which is is the process of text normaliztion, for casing of letters, contractions, and whatnot. I also created uni, bi, and trigram sentences, this combines common words found in the corpus together to a single word representation of the word such as ny_style_pizza.
All of the natural language processing has thus far been achieved using the python package Spacy. You can build you own stopword vocabulary in Spacy. I did choose to use NLTK to remove stopwords as they have a list ready to use out of the box. To do this step in Spacy I would have to build my own stopword dictionary and decided that wasn't necessary for this project. Finally I used Latent Direlecht Allocation to transform the data into a bag of words representation.


### Modeling the Data

The first step in modeling the data was to use the package Gensim to represnt the words in a highly dimensional vector space. 

Next we use a dimension reduction technique called t distributed neighbor embedding, t-sne. This reduced the dimensional space to an x and a y coordinate and similar topic words will be clustered together. 

The clusters make sense but are not labeled at this point, and would be more useful if they were. 

I attempted to use a variety of unsupervised learning techniques to find clusters that made sense. The Kmeans models seemed to miss inaccurately group some obvious clusters and the DBSCAN seemed to have trouble discerning a signal from the noise. Using a Spectral Clustering technique seemed to do really well though! 
Without tuning the Spectral Clustering algorithm, 8 kernals were detected. The center, or most dense region of group of words was represented as one cluster and most of the others were in another cluster and the outliers of the clusters represnted the other 6 kernals. Tuning this algorithm for nearest neighbors affinity and kmeans labels did very well! The code below represents how I trained the clustering algorithm using the python package sklearn.

```python
from sklearn.cluster import SpectralClustering
sc = SpectralClustering(affinity = 'nearest_neighbors', assign_labels = 'kmeans'

```

Prior to settling on a Spectral Clustering model, I had tested DBSCAN and 2 of the best silhouette scored Kmeans for 3 and 7 kernals. The DBSCAN model created 125 different clusters that didn't appear to be useful whatsoever. Both the KMeans models did an OK job of clustering the words, but the kmeans model with 3 kernals split the topic cluster of price into 2 differnt broader clusters and the kmeans model with 7 kernals split and obvious cluster of people names into 2 different broader clusters as well. The spectral clustering approach appeared to have done an excellent job of finding related words based on the words tsne coordinates.



<html lang="en">
    <head>
        <meta charset="utf-8">
        <title>Bokeh Plot</title>
        
<link rel="stylesheet" href="https://cdn.pydata.org/bokeh/release/bokeh-0.12.10.min.css" type="text/css" />
        
<script type="text/javascript" src="https://cdn.pydata.org/bokeh/release/bokeh-0.12.10.min.js"></script>
<script type="text/javascript" src="https://cdn.pydata.org/bokeh/release/bokeh-gl-0.12.10.min.js"></script>
<script type="text/javascript">
    Bokeh.set_log_level("info");
</script>
        <style>
          html {
            width: 100%;
            height: 100%;
          }
          body {
            width: 90%;
            height: 100%;
            margin: auto;
          }
        </style>
    </head>
    <body>
        
        <div class="bk-root">
            <div class="bk-plotdiv" id="bf059e28-4661-4287-8228-bd0c2d322a8b"></div>
        </div>
        
        <script type="text/javascript">
            (function() {
          var fn = function() {
            Bokeh.safely(function() {
              (function(root) {
                function embed_document(root) {
                  var render_items = [{"docid":"6837fae7-676c-4d37-9bf1-079b20ca94f3","elementid":"bf059e28-4661-4287-8228-bd0c2d322a8b","modelid":"44971e17-e06d-46ef-9ffe-e380a7ba58d5"}];
              
                  root.Bokeh.embed.embed_items(docs_json, render_items);
                }
              
                if (root.Bokeh !== undefined) {
                  embed_document(root);
                } else {
                  var attempts = 0;
                  var timer = setInterval(function(root) {
                    if (root.Bokeh !== undefined) {
                      embed_document(root);
                      clearInterval(timer);
                    }
                    attempts++;
                    if (attempts > 100) {
                      console.log("Bokeh: ERROR: Unable to embed document because BokehJS library is missing")
                      clearInterval(timer);
                    }
                  }, 10, root)
                }
              })(window);
            });
          };
          if (document.readyState != "loading") fn();
          else document.addEventListener("DOMContentLoaded", fn);
        })();
        
        </script>
    </body>
</html>

#### The words represented in the follow topic clusters:


<span style="color:black">O</span> Black clustered words are related to drinks, deserts, and texture.

<span style="color:yellow">O</span> Yellow clustered words are related to price.

<span style="color:navy">O</span> Navy clustered words are related to service.

<span style="color:pink">O</span> Pink clustered words are related to hours and events.

<span style="color:orange">O</span> Orange clustered words are related to location.

<span style="color:blue">O</span> Blue clustered words are related to Foriegn languages.

<span style="color:red">O</span> Red clustered words are related to upscale entrees.

<span style="color:green">O</span> Green clustered words are everything else.

### Conclusion

Great so I have an interactive plot that looks like a cheap knock off of a google logo, so what can I do with this? 
This model could be used for a number of applications. If this were a project for a real company/problem, this approach could be used to pipe customers to various treatments based on their reviewing behavior. Say someone posts about upscale entrees, grouped in red in the plot above, then your engagement with those customers may be more focused on menu items, where ash customers who reviews are more oriented towards price you may want to send more coupons or something. Even though tuning this model took some time for me to optimize and to try out different algorithms, it is still amazing to me that the models themselves can find such robust patterns in the data. While this was trained on the academic data set, I am seriously considering creating a version of this model with their API. I absolutely love the idea of tailoring my Yelp search to topics that I am interested in at any given time. Say I am in Portland and want to get a local brew with my pie, I could apply that model to only search for beer related pizza review! Perhaps I am in chicago and want the most upscale pizza I can find, I could use that cluster to find reviews in Chicago that are relevant. 