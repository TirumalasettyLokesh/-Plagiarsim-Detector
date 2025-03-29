#modules are imported:
==================
#admin.py:
from django.contrib import admin

#apps.py:
=====

from django.apps import AppConfig


class PlagiarismcheckerConfig(AppConfig):
    name = 'plagiarismchecker'
#models.py:
===
from django.db import models

tests.py:
=====
from django.test import TestCase
    





# -Plagiarsim-Detector
from nltk.corpus import stopwords
from plagiarismchecker.algorithm import webSearch
import sys
import re

# Given a text string, remove all non-alphanumeric
# characters (using Unicode definition of alphanumeric).
main.py
================
def getQueries(text, n):
    sentenceEnders = re.compile("['.!?]")
    sentenceList = sentenceEnders.split(text)
    sentencesplits = []
    en_stops = set(stopwords.words('english'))

    for sentence in sentenceList:
        x = re.compile(r'\W+', re.UNICODE).split(sentence)
        for word in x:
            if word.lower() in en_stops:
                x.remove(word)
        x = [ele for ele in x if ele != '']
        sentencesplits.append(x)
    finalq = []
    for sentence in sentencesplits:
        l = len(sentence)
        if l > n:
            l = int(l/n)
            index = 0
            for i in range(0, l):
                finalq.append(sentence[index:index+n])
                index = index + n-1
                if index+n > l:
                    index = l-n-1
            if index != len(sentence):
                finalq.append(sentence[len(sentence)-index:len(sentence)])
        else:
            if l > 4:
                finalq.append(sentence)
    return finalq


def findSimilarity(text):
    # n-grams N VALUE SET HERE
    n = 9
    queries = getQueries(text, n)
    print('GetQueries task complete')
    q = [' '.join(d) for d in queries]
    output = {}
    c = {}
    i = 1
    while("" in q):
        q.remove("")
    count = len(q)
    if count > 100:
        count = 100
    numqueries = count
    for s in q[0:count]:
        output, c, errorCount = webSearch.searchWeb(s, output, c)
        print('Web search task complete')
        numqueries = numqueries - errorCount
        # print(output,c)
        sys.stdout.flush()
        i = i+1
    totalPercent = 0
    outputLink = {}
    print(output, c)
    prevlink = ''
    for link in output:
        percentage = (output[link]*c[link]*100)/numqueries
        if percentage > 10:
            totalPercent = totalPercent + percentage
            prevlink = link
            outputLink[link] = percentage
        elif len(prevlink) != 0:
            totalPercent = totalPercent + percentage
            outputLink[prevlink] = outputLink[prevlink] + percentage
        elif c[link] == 1:
            totalPercent = totalPercent + percentage
        print(link, totalPercent)

    print(count, numqueries)
    print(totalPercent, outputLink)
    print("\nDone!")
    return totalPercent, outputLink

#cosinesim.py:
================================
# module to check cosine similarity of two strings

import re
import math
from collections import Counter

WORD = re.compile(r'\w+')

# returns cosine similarity of two vectors
# input: two vectors
# output: integer between 0 and 1.

def get_cosine(vec1, vec2):
    intersection = set(vec1.keys()) & set(vec2.keys())
#      print(intersection)
    matchWords = {}
    for i in intersection:
        if(vec1[i] > vec2[i]):
            matchWords[i] = vec2[i]
        else:
            matchWords[i] = vec1[i]
        # print(i)
    # print(matchWords)
	
    # calculating numerator
    numerator = sum([vec1[x] * matchWords[x] for x in intersection])
    # print(numerator)
    # calculating denominator
    sum1 = sum([vec1[x]**2 for x in vec1.keys()])
    sum2 = sum([matchWords[x]**2 for x in matchWords.keys()])
    # print(sum1,sum2)
    denominator = math.sqrt(sum1) * math.sqrt(sum2)
    # print(denominator)
    # checking for divide by zero
    if denominator == 0:
        return 0.0
    else:
        return float(numerator) / denominator

# converts given text into a vector

def text_to_vector(text):
    # uses the Regular expression above and gets all words
    words = WORD.findall(text)
    # returns a counter of all the words (count of number of occurences)
    return Counter(words)

# returns cosine similarity of two words

def cosineSim(text1, text2):
    t1 = text1.lower()
    t2 = text2.lower()
    # print('t1 : ',t1, '\nt2 : ', t2)
    vector1 = text_to_vector(t1)
    vector2 = text_to_vector(t2)
    cosine = get_cosine(vector1, vector2)
    return cosine


    import re
import math
from nltk.corpus import stopwords


def findFileSimilarity(inputQuery, database):

	universalSetOfUniqueWords = []
	matchPercentage = 0

	lowercaseQuery = inputQuery.lower()
	en_stops = set(stopwords.words('english'))

	# Replace punctuation by space and split
	queryWordList = re.sub("[^\w]", " ", lowercaseQuery).split()
	# queryWordList = map(str, queryWordList)					#This was causing divide by zero error

	for word in queryWordList:
		if word not in universalSetOfUniqueWords:
			universalSetOfUniqueWords.append(word)

	database1 = database.lower()

	# Replace punctuation by space and split
	databaseWordList = re.sub("[^\w]", " ", database1).split()
	# databaseWordList = map(str, databaseWordList)			#And this also leads to divide by zero error

	for word in databaseWordList:
		if word not in universalSetOfUniqueWords:
			universalSetOfUniqueWords.append(word)

	for word in universalSetOfUniqueWords:
		if word in en_stops:
			universalSetOfUniqueWords.remove(word)

	queryTF = []
	databaseTF = []

	for word in universalSetOfUniqueWords:
		queryTfCounter = 0
		databaseTfCounter = 0

		for word2 in queryWordList:
			if word == word2:
				queryTfCounter += 1
		queryTF.append(queryTfCounter)

		for word2 in databaseWordList:
			if word == word2:
				databaseTfCounter += 1
		databaseTF.append(databaseTfCounter)

	dotProduct = 0
	for i in range(len(queryTF)):
		dotProduct += queryTF[i]*databaseTF[i]

	queryVectorMagnitude = 0
	for i in range(len(queryTF)):
		queryVectorMagnitude += queryTF[i]**2
	queryVectorMagnitude = math.sqrt(queryVectorMagnitude)

	databaseVectorMagnitude = 0
	for i in range(len(databaseTF)):
		databaseVectorMagnitude += databaseTF[i]**2
	databaseVectorMagnitude = math.sqrt(databaseVectorMagnitude)

	matchPercentage = (float)(
		dotProduct / (queryVectorMagnitude * databaseVectorMagnitude))*100

# 	print (universalSetOfUniqueWords)
# 	print()
# 	print (databaseWordList)


# 	print (queryTF)
# 	print (databaseTF)

	return matchPercentage
#websearch.py
============================
from plagiarismchecker.algorithm import ConsineSim
from apiclient.discovery import build

# searchEngine_API = 'AIzaSyAoEYif8sqEYvj1P6vYLw6CGMrQbDMmaq8'
# searchEngine_API = 'AIzaSyCUYy9AtdMUddiNA0gOcsGPQcE372ytyCw'
# searchEngine_API = 'AIzaSyAQYLRBBeDQNxADPQtUnApntz78-urWEZI'
searchEngine_API = 'AIzaSyCAeR7_6TTKzoJmSwmOuHZvKcVg_lhqvCc'
searchEngine_Id = '758ad3e78879f0e08'

def searchWeb(text, output, c):
    text = text
    # print(text)
    try:
        resource = build("customsearch", 'v1',
                         developerKey=searchEngine_API).cse()
        result = resource.list(q=text, cx=searchEngine_Id).execute()
        searchInfo = result['searchInformation']
        # print(searchInfo)
        if(int(searchInfo['totalResults']) > 0):
            maxSim = 0
            itemLink = ''
            numList = len(result['items']) 
            if numList >= 5:
                numList = 5
            for i in range(0, numList):
                item = result['items'][i]
                content = item['snippet']
                simValue = ConsineSim.cosineSim(text, content)
                if simValue > maxSim:
                    maxSim = simValue
                    itemLink = item['link']
                if item['link'] in output:
                    itemLink = item['link']
                    break
            if itemLink in output:
                print('if', maxSim)
                output[itemLink] = output[itemLink] + 1
                c[itemLink] = ((c[itemLink] *
                                (output[itemLink]-1) + maxSim)/(output[itemLink]))
            else:
                print('else', maxSim)
                print(text)
                print(itemLink)
                output[itemLink] = 1
                c[itemLink] = maxSim
    except Exception as e:
        print(text)
        print(e)
        print('error')
        return output, c, 1
    return output, c, 0
