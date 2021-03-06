# -Stock-Market-Sentiment-Analysis-Using-Python-Machine-Learning
pip install vaderSentiment

import pandas as pd
import numpy as np
from textblob import TextBlob
import re
import nltk
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis

from google.colab import files
files.upload()

df1 = pd.read_csv('datasets_129_792900_upload_DJIA_table.csv')
df2 = pd.read_csv('129_792900_compressed_Combined_News_DJIA.csv.zip')

df1.head(3)

df2.head(3)

merge = df1.merge(df2, how='inner', on='Date', left_index = True)
merge.head(3)

headlines = []
for row in range(0,len(merge.index)):
    headlines.append(' '.join(str(x) for x in merge.iloc[row,2:27]))
    
clean_headlines = []
for i in range(0, len(headlines)):
  clean_headlines.append(re.sub("b[(')]+", '', headlines[i] ))
  clean_headlines[i] = re.sub('b[(")]+', '', clean_headlines[i] )
  clean_headlines[i] = re.sub("\'", '', clean_headlines[i] )
  
merge['Combined_News'] = clean_headlines

def getSubjectivity(text):
  return TextBlob(text).sentiment.subjectivity
  
def getPolarity(text):
  return  TextBlob(text).sentiment.polarity
  
merge['Subjectivity'] =merge['Combined_News'].apply(getSubjectivity)
merge['Polarity'] =merge['Combined_News'].apply(getPolarity)

def getSIA(text):
  sia = SentimentIntensityAnalyzer()
  sentiment = sia.polarity_scores(text)
  return sentiment
  
compound = []
neg = []
neu = []
pos = []
SIA = 0
for i in range(0, len(merge['Combined_News'])):
  SIA = getSIA(merge['Combined_News'][i])
  compound.append(SIA['compound'])
  neg.append(SIA['neg'])
  neu.append(SIA['neu'])
  pos.append(SIA['pos'])
  
merge['Compound'] =compound
merge['Negative'] =neg
merge['Neutral'] =neu
merge['Positive'] = pos

keep_columns = [ 'Open',  'High', 'Low',    'Volume', 'Subjectivity', 'Polarity', 'Compound', 'Negative', 'Neutral' ,'Positive',  'Label' ]
df = merge[keep_columns]
df

X = df
X = np.array(X.drop(['Label'], 1))

y = np.array(df['Label'])

x_train, x_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state = 0)

model = LinearDiscriminantAnalysis().fit(x_train, y_train)

predictions = model.predict(x_test)
predictions

print( classification_report(y_test, predictions) )
  
  
