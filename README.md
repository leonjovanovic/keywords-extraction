# Keyword Extraction

&nbsp;&nbsp;&nbsp;&nbsp;The goal of this project is to extract meaningful keywords from English text. We considered five different approaches to keyword extraction, namely:

* Approach without pre-trained LM
* Cosine similarity + candidate embeddings from LM
* Token classification encoder with O, B and I
* Generative encoder decoder to generate whole keywords
* OpenAI ChatGPT API

&nbsp;&nbsp;&nbsp;&nbsp;The fourth approach was abandoned due to the complexity and length of training a full transformer architecture model. For the third solution, we need to find a dataset to train on.

## Scake

&nbsp;&nbsp;&nbsp;&nbsp;Scake is a method that doesn’t use language models to extract keywords. First, it preprocesses data using tokenization, stop-word removal, stemming, and part-of-speech tagging. Next, Scake selects candidates for the applied text. Thirdly, Scake generates a graph using candidates as nodes. Finally, Scake uses a semantic connectivity-based word scoring method (SCScore) to compute a word score for the candidates and outputs candidates with the highest scores.

## KeyBERT

&nbsp;&nbsp;&nbsp;&nbsp;KeyBERT uses LM to generate embeddings that will be compared to text embeddings to determine which candidate is the most similar. Similar to Scake, the method first preprocesses the text with lowercasing, stop-word removal, POS tagging using the SpaCy library. Using POS tags, adjectives + nouns are selected as candidates. We pass candidates and text through some variation of BERT language model to get their embeddings. Calculate cosine similarity between each candidate embedding and text embedding and output candidates with the highest scores.

## Fine-tuned BERT based model for Token classification

&nbsp;&nbsp;&nbsp;&nbsp;To fine-tune a BERT-based model for token classification, we first had to find a dataset that contains texts and gold keywords. We decided on the biggest dataset (Inspec), which was already on Hugging Face ready to be imported. We also preprocessed the dataset to align labels with input tokens and map them to 0, 1, and 2 integers, which represent classes.

&nbsp;&nbsp;&nbsp;&nbsp;For the model, we used DistilRoBERTa, which is loaded with a TokenClassification head that outputs three classes. We set up hyperparameters such as learning rate, batch size, and LR scheduler for better convergence. We also applied DataCollator to make sure the whole batch has the same shape.

&nbsp;&nbsp;&nbsp;&nbsp;Since the classes are not balanced (O class in the majority), we used F1 score per class and macro-averaged as a metric. We trained the model until it stops learning and starts overfitting (val loss stops decreasing). Finally, we applied softmax on logits and extracted only keywords that start with B. Post-processing was used to remove duplicates.

## ChatGPT API

&nbsp;&nbsp;&nbsp;&nbsp;Using ChatGPT API, we first set the system settings of the model to keyword extraction using several rules. After that forwarding whole text as an example will result in several keywords, by the previously set rules (max 10 keywords, ranked by score etc.)

## Improvements

There are several ways to improve the performance of the models, including:

* Choose a better model for embeddings based on the efficiency/quality trade-off.
* Use a bigger spacy model for more accurate adjective/noun detection.
* Do a post-processing to remove similar candidates, or even better, calculate cosine similarity between candidates and discard ones that are too similar to already accepted candidates.
* Remove some spacy pipes for faster computation.
* Use more datasets combined for more data.
* Use less data for validation and test and more for training (e.g., 60/20/20).
* Remove stop words.
* Based on the efficiency/quality trade-off, choose a better model for training (e.g., RoBERTa, BERT, etc.).
* Play around with hyperparameters for better results.
* Better post-processing to remove similar keywords. We can use Levenshtein similarity for a naïve approach or cosine similarity of candidate embeddings.
* If the text is long, split it and combine outputs.
