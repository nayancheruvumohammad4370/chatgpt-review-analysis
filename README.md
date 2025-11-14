# chatgpt-review-analysis
Analysis of ChatGPT user reviews, sentiment scoring, and insights based on real feedback.

# ChatGPT Reviews Analysis
# (Basic sentiment + review insights)

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from textblob import TextBlob
import re
from wordcloud import WordCloud

# -----------------------
# Load the dataset
# -----------------------
df = pd.read_csv("chatgpt_reviews.csv")
df.head()

# -----------------------
# Basic cleaning
# -----------------------
def clean_text(text):
    text = str(text).lower()
    text = re.sub(r'http\S+', '', text)
    text = re.sub('[^a-zA-Z ]', ' ', text)
    text = re.sub('\s+', ' ', text)
    return text.strip()

df["clean_review"] = df["Review"].apply(clean_text)

# -----------------------
# Sentiment score (TextBlob)
# -----------------------
def get_sentiment(txt):
    return TextBlob(txt).sentiment.polarity

df["sentiment_score"] = df["clean_review"].apply(get_sentiment)

# -----------------------
# Convert score to label
# -----------------------
def label(s):
    if s > 0.1:
        return "Positive"
    elif s < -0.1:
        return "Negative"
    return "Neutral"

df["sentiment_type"] = df["sentiment_score"].apply(label)

# -----------------------
# Add another useful column → review word count
# -----------------------
df["word_count"] = df["Review"].astype(str).apply(lambda x: len(x.split()))

# -----------------------
# Quick summary
# -----------------------
print("Sentiment Split:")
print(df["sentiment_type"].value_counts())

# -----------------------
# Plot sentiment distribution
# -----------------------
plt.figure(figsize=(6,4))
df["sentiment_type"].value_counts().plot(kind="bar")
plt.title("Sentiment Distribution")
plt.xlabel("Sentiment")
plt.ylabel("Count")
plt.show()

# -----------------------
# Wordclouds to understand themes
# -----------------------
pos_text = " ".join(df[df.sentiment_type=="Positive"]["clean_review"])
neg_text = " ".join(df[df.sentiment_type=="Negative"]["clean_review"])

wc = WordCloud(width=900, height=400)

# Positive
plt.figure(figsize=(10,4))
plt.imshow(wc.generate(pos_text))
plt.axis("off")
plt.title("Positive Review WordCloud")
plt.show()

# Negative
plt.figure(figsize=(10,4))
plt.imshow(wc.generate(neg_text))
plt.axis("off")
plt.title("Negative Review WordCloud")
plt.show()

# -----------------------
# Export enriched dataset
# -----------------------
df.to_excel("chatgpt_review_analysis_output.xlsx", index=False)

print("Analysis complete. File saved.")

