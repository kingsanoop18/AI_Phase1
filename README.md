{
 "cells": [
  {
   "cell_type": "code",
   "execution_count": 1,
   "metadata": {},
   "outputs": [],
   "source": [
    "from nltk import sent_tokenize, pos_tag\n",
    "from nltk.tokenize import TreebankWordTokenizer\n",
    "from nltk.stem import WordNetLemmatizer\n",
    "from nltk.corpus import wordnet as wn\n",
    "from nltk.corpus import sentiwordnet as swn\n",
    "from nltk.sentiment.util import mark_negation\n",
    "from string import punctuation\n",
    "from IPython.display import display\n",
    "import pandas as pd\n",
    "import numpy as np\n",
    "import seaborn as sns\n",
    "import matplotlib.pyplot as plt\n",
    "pd.set_option('display.max_columns', None)\n",
    "pd.set_option('display.max_colwidth', None)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Sentiment Scoring Using SentiWordNet"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 2,
   "metadata": {},
   "outputs": [],
   "source": [
    "def penn_to_wn(tag):\n",
    "    \"\"\"\n",
    "        Convert between the PennTreebank tags to simple Wordnet tags\n",
    "    \"\"\"\n",
    "    if tag.startswith('J'):\n",
    "        return wn.ADJ\n",
    "    elif tag.startswith('N'):\n",
    "        return wn.NOUN\n",
    "    elif tag.startswith('R'):\n",
    "        return wn.ADV\n",
    "    elif tag.startswith('V'):\n",
    "        return wn.VERB\n",
    "    return None"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "metadata": {},
   "outputs": [],
   "source": [
    "def get_sentiment_score(text):\n",
    "    \n",
    "    \"\"\"\n",
    "        This method returns the sentiment score of a given text using SentiWordNet sentiment scores.\n",
    "        input: text\n",
    "        output: numeric (double) score, >0 means positive sentiment and <0 means negative sentiment.\n",
    "    \"\"\"    \n",
    "    total_score = 0\n",
    "    #print(text)\n",
    "    raw_sentences = sent_tokenize(text)\n",
    "    #print(raw_sentences)\n",
    "    \n",
    "    for sentence in raw_sentences:\n",
    "\n",
    "        sent_score = 0     \n",
    "        sentence = str(sentence)\n",
    "        #print(sentence)\n",
    "        sentence = sentence.replace(\"<br />\",\" \").translate(str.maketrans('','',punctuation)).lower()\n",
    "        tokens = TreebankWordTokenizer().tokenize(text)\n",
    "        tags = pos_tag(tokens)\n",
    "        for word, tag in tags:\n",
    "            wn_tag = penn_to_wn(tag)\n",
    "            if not wn_tag:\n",
    "                continue\n",
    "            lemma = WordNetLemmatizer().lemmatize(word, pos=wn_tag)\n",
    "            if not lemma:\n",
    "                continue\n",
    "            synsets = wn.synsets(lemma, pos=wn_tag)\n",
    "            if not synsets:\n",
    "                continue\n",
    "            synset = synsets[0]\n",
    "            swn_synset = swn.senti_synset(synset.name())\n",
    "            sent_score += swn_synset.pos_score() - swn_synset.neg_score()\n",
    "\n",
    "        total_score = total_score + (sent_score / len(tokens))\n",
    "\n",
    "    \n",
    "    return (total_score / len(raw_sentences)) * 100"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "metadata": {},
   "outputs": [],
   "source": [
    "reviews = pd.read_csv(\"../data/small_corpus.csv\")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "(4500, 12)"
      ]
     },
     "execution_count": 5,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "reviews.shape"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 6,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>overall</th>\n",
       "      <th>verified</th>\n",
       "      <th>reviewTime</th>\n",
       "      <th>reviewerID</th>\n",
       "      <th>asin</th>\n",
       "      <th>reviewerName</th>\n",
       "      <th>reviewText</th>\n",
       "      <th>summary</th>\n",
       "      <th>unixReviewTime</th>\n",
       "      <th>vote</th>\n",
       "      <th>style</th>\n",
       "      <th>image</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>0</th>\n",
       "      <td>1.0</td>\n",
       "      <td>True</td>\n",
       "      <td>11 30, 2015</td>\n",
       "      <td>A3AC92K59QLYR8</td>\n",
       "      <td>B00503E8S2</td>\n",
       "      <td>ben</td>\n",
       "      <td>Game freezes over and over its unplayable</td>\n",
       "      <td>it just doesn't work</td>\n",
       "      <td>1448841600</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{'Format:': ' Video Game'}</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1</th>\n",
       "      <td>1.0</td>\n",
       "      <td>False</td>\n",
       "      <td>05 19, 2012</td>\n",
       "      <td>A334LHR8DWARY8</td>\n",
       "      <td>B00178630A</td>\n",
       "      <td>Xenocide</td>\n",
       "      <td>I have no problem with needing to be online to play and have had no problems with stability since the first 2 days. This review ignores those issues.\\n\\nThe game itself is fun and addictive, but so is Din's Curse (which was programmed by one guy) and is more re-playble than Diablo 3. Hopefully Torchlight 2 will take the mantle of this genere from Blizzard. One reason all these games are fun is because they are all similar in atmosphere, gameplay and loot system. As a stand alone title, D3 would be 4 stars, maybe- but as a successor to Diablo 2, with all the man-power and resources Blizzard had including the beautiful skill tree system from D2... If I could go lower than 1, I would.\\n\\nWhy was Diablo 2 addictive? One big reason was farming gear- loading magic find amd finding a group for Hell runs, kind of like a drug. But why? For the millions of regular players it was in order to gear up their next specialized build. Everything was centered around unique builds that become somewhat cookie cutter if good. Players perfectly balancing point distributions in attributes and skills to make that personalized zealot or blizzard sorc. The quest for constant gear for new builds was what drove the desire to aquire it. Every major nerf patch was like a ladder reset (which D3 lacks). Even if they add another 5 classes, the problem of no variation or ability to make a custom character means no constant demand for anything but a few top level items.\\n\\nThis seems intentional to keep server use lighter between expansions as there really is little reason to play unless your into farming for only a handful of desired endgame items. Everything else can be gotten off the AH for little gold. Unless they introduce raids that give special gear or something that would make that game into an MMO, none of the desire to keep playing is there unless it's more of a social hang-out.\\n\\nSo, where did all the money and time go? The skills are fun, as are the runes- but also very simplistic. The cut-screens are nicely done but, in this genere, most people watch them once or twice at most, they are basically useless. I'd hate to think all the resources went there when the cut-screens are movie quality where you see real looking people crying and giving decent lines, but then we shift back to real game-play and the next line from the smith who just killed his wife after she vomited and changed into an undead creature is a comically gruff and bemused... \"If you see my idiot apprentice out there, tell him to get back...\"\\n\\nThe same companion dialogue reapeated over and over and often when fighting so it's annoying. It's all just randomized for the most part. So, your talking to your companions in a casual way about something trivial while getting killed by a boss. Poor execution of useless content. It happens often as well.\\n\\nIt is just sad to see no ability to customize a build, choose your dialogue or do much outside of what this game present to you. You essentially walking through how the developers envisioned you to progress- from what you say to what skills you want to master.\\n\\nIt's really sad to see how they dumbed this game down to the point where there is no ability to customize any aspect of your gameplay.</td>\n",
       "      <td>The only real way to show Blizzard our feelings is through Ultimate ratings</td>\n",
       "      <td>1337385600</td>\n",
       "      <td>23</td>\n",
       "      <td>{'Format:': ' Computer Game'}</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>1.0</td>\n",
       "      <td>True</td>\n",
       "      <td>10 19, 2014</td>\n",
       "      <td>A28982ODE7ZGVP</td>\n",
       "      <td>B001AWIP7M</td>\n",
       "      <td>Eric Frykberg</td>\n",
       "      <td>NOT GOOD</td>\n",
       "      <td>One Star</td>\n",
       "      <td>1413676800</td>\n",
       "      <td>NaN</td>\n",
       "      <td>{'Format:': ' Video Game'}</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>3</th>\n",
       "      <td>1.0</td>\n",
       "      <td>True</td>\n",
       "      <td>09 6, 2015</td>\n",
       "      <td>A19E85RLQCAMI1</td>\n",
       "      <td>B00NASF4MS</td>\n",
       "      <td>Joe</td>\n",
       "      <td>Really not worth the money to buy this game on PS4 unless it becomes $10.... don't make the mistake I made.</td>\n",
       "      <td>Really not worth the money to buy this game on ...</td>\n",
       "      <td>1441497600</td>\n",
       "      <td>2</td>\n",
       "      <td>{'Format:': ' Video Game'}</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>4</th>\n",
       "      <td>1.0</td>\n",
       "      <td>False</td>\n",
       "      <td>05 28, 2008</td>\n",
       "      <td>AEMQKS13WC4D2</td>\n",
       "      <td>B00140P9BA</td>\n",
       "      <td>Craig</td>\n",
       "      <td>They need to eliminate the Securom. I purchased Mass Effect as a digital download hoping that the faulty disc protection software would not be on that version, however it seems the Securom is on all versions. Now every time I log on to play, it's hit or miss- sometimes an error pops up stating \"a required security module could not be activated\", and sometimes it works. It's like pulling a handle on a slot machine to see if Securom will allow you play or not. Ridiculous for a game I spent $50 on. There's a whole thread about this issue on the official forums. Don't have this issue with other games that use less intrusive copy protection methods.</td>\n",
       "      <td>Securom can ruin a great game</td>\n",
       "      <td>1211932800</td>\n",
       "      <td>55</td>\n",
       "      <td>{'Format:': ' DVD-ROM'}</td>\n",
       "      <td>NaN</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "   overall  verified   reviewTime      reviewerID        asin   reviewerName  \\\n",
       "0      1.0      True  11 30, 2015  A3AC92K59QLYR8  B00503E8S2            ben   \n",
       "1      1.0     False  05 19, 2012  A334LHR8DWARY8  B00178630A       Xenocide   \n",
       "2      1.0      True  10 19, 2014  A28982ODE7ZGVP  B001AWIP7M  Eric Frykberg   \n",
       "3      1.0      True   09 6, 2015  A19E85RLQCAMI1  B00NASF4MS            Joe   \n",
       "4      1.0     False  05 28, 2008   AEMQKS13WC4D2  B00140P9BA          Craig   \n",
       "\n",
       "                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   reviewText  \\\n",
       "0                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   Game freezes over and over its unplayable   \n",
       "1  I have no problem with needing to be online to play and have had no problems with stability since the first 2 days. This review ignores those issues.\\n\\nThe game itself is fun and addictive, but so is Din's Curse (which was programmed by one guy) and is more re-playble than Diablo 3. Hopefully Torchlight 2 will take the mantle of this genere from Blizzard. One reason all these games are fun is because they are all similar in atmosphere, gameplay and loot system. As a stand alone title, D3 would be 4 stars, maybe- but as a successor to Diablo 2, with all the man-power and resources Blizzard had including the beautiful skill tree system from D2... If I could go lower than 1, I would.\\n\\nWhy was Diablo 2 addictive? One big reason was farming gear- loading magic find amd finding a group for Hell runs, kind of like a drug. But why? For the millions of regular players it was in order to gear up their next specialized build. Everything was centered around unique builds that become somewhat cookie cutter if good. Players perfectly balancing point distributions in attributes and skills to make that personalized zealot or blizzard sorc. The quest for constant gear for new builds was what drove the desire to aquire it. Every major nerf patch was like a ladder reset (which D3 lacks). Even if they add another 5 classes, the problem of no variation or ability to make a custom character means no constant demand for anything but a few top level items.\\n\\nThis seems intentional to keep server use lighter between expansions as there really is little reason to play unless your into farming for only a handful of desired endgame items. Everything else can be gotten off the AH for little gold. Unless they introduce raids that give special gear or something that would make that game into an MMO, none of the desire to keep playing is there unless it's more of a social hang-out.\\n\\nSo, where did all the money and time go? The skills are fun, as are the runes- but also very simplistic. The cut-screens are nicely done but, in this genere, most people watch them once or twice at most, they are basically useless. I'd hate to think all the resources went there when the cut-screens are movie quality where you see real looking people crying and giving decent lines, but then we shift back to real game-play and the next line from the smith who just killed his wife after she vomited and changed into an undead creature is a comically gruff and bemused... \"If you see my idiot apprentice out there, tell him to get back...\"\\n\\nThe same companion dialogue reapeated over and over and often when fighting so it's annoying. It's all just randomized for the most part. So, your talking to your companions in a casual way about something trivial while getting killed by a boss. Poor execution of useless content. It happens often as well.\\n\\nIt is just sad to see no ability to customize a build, choose your dialogue or do much outside of what this game present to you. You essentially walking through how the developers envisioned you to progress- from what you say to what skills you want to master.\\n\\nIt's really sad to see how they dumbed this game down to the point where there is no ability to customize any aspect of your gameplay.   \n",
       "2                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    NOT GOOD   \n",
       "3                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 Really not worth the money to buy this game on PS4 unless it becomes $10.... don't make the mistake I made.   \n",
       "4                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                They need to eliminate the Securom. I purchased Mass Effect as a digital download hoping that the faulty disc protection software would not be on that version, however it seems the Securom is on all versions. Now every time I log on to play, it's hit or miss- sometimes an error pops up stating \"a required security module could not be activated\", and sometimes it works. It's like pulling a handle on a slot machine to see if Securom will allow you play or not. Ridiculous for a game I spent $50 on. There's a whole thread about this issue on the official forums. Don't have this issue with other games that use less intrusive copy protection methods.   \n",
       "\n",
       "                                                                       summary  \\\n",
       "0                                                         it just doesn't work   \n",
       "1  The only real way to show Blizzard our feelings is through Ultimate ratings   \n",
       "2                                                                     One Star   \n",
       "3                           Really not worth the money to buy this game on ...   \n",
       "4                                                Securom can ruin a great game   \n",
       "\n",
       "   unixReviewTime vote                          style image  \n",
       "0      1448841600  NaN     {'Format:': ' Video Game'}   NaN  \n",
       "1      1337385600   23  {'Format:': ' Computer Game'}   NaN  \n",
       "2      1413676800  NaN     {'Format:': ' Video Game'}   NaN  \n",
       "3      1441497600    2     {'Format:': ' Video Game'}   NaN  \n",
       "4      1211932800   55        {'Format:': ' DVD-ROM'}   NaN  "
      ]
     },
     "execution_count": 6,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "reviews.head()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 7,
   "metadata": {},
   "outputs": [],
   "source": [
    "reviews.dropna(subset=['reviewText'], inplace=True)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 8,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "(4499, 12)"
      ]
     },
     "execution_count": 8,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "reviews.shape"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 9,
   "metadata": {},
   "outputs": [],
   "source": [
    "reviews['swn_score'] = reviews['reviewText'].apply(lambda text : get_sentiment_score(text))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>reviewText</th>\n",
       "      <th>swn_score</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>1086</th>\n",
       "      <td>Anything with Activision involved will equal a piece of sh! t. Bungie should stick to making Halo because that's all they're good for. Game was repetitive mission wise. Story was disappointing because you develop any emotional connection to the story at all. You're just there to shoot enemies that re spawn in the same area and work your ass off to find \"rare\" items which 1/10 chances you'll actually find any. I'll give it 1 star because despite the crap Bungie had to offer, there's still some kind of enjoyment to be in the game. Shame that this crappy game is INCOMPLETE. So if you actually bought the game along with the season pass, that is actually the whole game right there. This game alone  is just some money magnet that in the end will leave you unsatisfied.</td>\n",
       "      <td>1.333333</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2</th>\n",
       "      <td>NOT GOOD</td>\n",
       "      <td>6.250000</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                reviewText  \\\n",
       "1086  Anything with Activision involved will equal a piece of sh! t. Bungie should stick to making Halo because that's all they're good for. Game was repetitive mission wise. Story was disappointing because you develop any emotional connection to the story at all. You're just there to shoot enemies that re spawn in the same area and work your ass off to find \"rare\" items which 1/10 chances you'll actually find any. I'll give it 1 star because despite the crap Bungie had to offer, there's still some kind of enjoyment to be in the game. Shame that this crappy game is INCOMPLETE. So if you actually bought the game along with the season pass, that is actually the whole game right there. This game alone  is just some money magnet that in the end will leave you unsatisfied.   \n",
       "2                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 NOT GOOD   \n",
       "\n",
       "      swn_score  \n",
       "1086   1.333333  \n",
       "2      6.250000  "
      ]
     },
     "execution_count": 10,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "reviews[['reviewText','swn_score']].sample(2)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 11,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>reviewText</th>\n",
       "      <th>swn_score</th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>2800</th>\n",
       "      <td>I get it, you want the most space possible for you Vita but the memory card prices are outrageous. Well, there is not much you can do about that. This card retails for $100. Ouch.\\n\\nI have heard about deals where people found it for $70, but I have never seen those (and boy have I looked).\\nSo this one seems to be the best deal out there. $88 when I bought it (it seems to fluctuate up and down about $1-2).\\nThis is the best deal I could find, and it is an everyday price. Not going to save the planet or anything, but it is $10 well saved.\\n\\nThe card works fine. I didn't expect otherwise.\\nI am happy that I got the 32GB because I won't have to worry about getting another card later. (And the cost of 2 cards is practically just as expensive but without the convenience of it being a single card). I have a ton of games downloaded thanks to PS+ and I am good for a while.\\nI know that I would be good with 16GB, but it wouldn't be future proof. This is the way to go, I just had to save a bit of money to do it.</td>\n",
       "      <td>4.023605</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2530</th>\n",
       "      <td>do not play it enought to give a review. that is all I can say at this time - thank you I will se what I can do at a later time and thank you for the opportunity to review</td>\n",
       "      <td>-0.312500</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       reviewText  \\\n",
       "2800  I get it, you want the most space possible for you Vita but the memory card prices are outrageous. Well, there is not much you can do about that. This card retails for $100. Ouch.\\n\\nI have heard about deals where people found it for $70, but I have never seen those (and boy have I looked).\\nSo this one seems to be the best deal out there. $88 when I bought it (it seems to fluctuate up and down about $1-2).\\nThis is the best deal I could find, and it is an everyday price. Not going to save the planet or anything, but it is $10 well saved.\\n\\nThe card works fine. I didn't expect otherwise.\\nI am happy that I got the 32GB because I won't have to worry about getting another card later. (And the cost of 2 cards is practically just as expensive but without the convenience of it being a single card). I have a ton of games downloaded thanks to PS+ and I am good for a while.\\nI know that I would be good with 16GB, but it wouldn't be future proof. This is the way to go, I just had to save a bit of money to do it.   \n",
       "2530                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  do not play it enought to give a review. that is all I can say at this time - thank you I will se what I can do at a later time and thank you for the opportunity to review   \n",
       "\n",
       "      swn_score  \n",
       "2800   4.023605  \n",
       "2530  -0.312500  "
      ]
     },
     "execution_count": 11,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "reviews[['reviewText','swn_score']].sample(2)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 12,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 1440x720 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "fig , ax = plt.subplots(nrows=1, ncols=1, figsize=(20,10))\n",
    "sns.histplot(x='swn_score', data=reviews.query(\"swn_score < 8 and swn_score > -8\"), ax=ax)\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 13,
   "metadata": {},
   "outputs": [],
   "source": [
    "reviews['swn_sentiment'] = reviews['swn_score'].apply(lambda x: \"positive\" if x>1 else (\"negative\" if x<0.5 else \"neutral\"))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 14,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "positive    2199\n",
       "negative    1823\n",
       "neutral      477\n",
       "Name: swn_sentiment, dtype: int64"
      ]
     },
     "execution_count": 14,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "reviews['swn_sentiment'].value_counts(dropna=False)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 15,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<AxesSubplot:xlabel='overall', ylabel='count'>"
      ]
     },
     "execution_count": 15,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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LPBsLZYmSWTFihWMGzeOBg0akJqaytSpUzn77LPZuHEj/fr1A6Br16588803R3Tn8qxZs3jhhRdITU3l5JNP5vbbb6dVq1ZMnDiRvLw89u7dS2pqKo8//jinnXZatesZOnQo1157LVOmTKk4sX048vLyKC4u5txzzwUiJ9JffPFFUlJSqn3PyJEjGT16NE2aNGHx4sU6byGHZJG/0/VLdna260l5yae4uJiOHTvGu4x6QduydtWV+yzMrNDds6tapuE+REQklIYoF6mBt956i/Hjx/+oLTMzk7lz58apIpHYUliI1ED//v3p379/vMsQiRsdhhIRkVAKCxERCaWwEBGRUDpnIfVej3EzwjsdhsIHr6rV9R2Jbdu28dJLL3HDDTcAkZsPb7rppiO6R0OkJrRnIVIHbdu2jSeeeKJivk2bNgoKiSqFhUgUVDc0+Nq1axkwYAA9evSgb9++fPbZZwCsXbuWnJwcunTpwh133EGzZs2AyICAubm5dO/enS5dujBvXmQ4tQkTJrB27VqysrIYN24cJSUlFcOV5OTksHLlyopazjvvPAoKCti5cydXX301vXr14pxzzqlYl0hNKCxEoqSqocFHjRrFo48+SmFhIZMnT644jDR27FjGjh3LihUrSE9Pr1hH48aNmTt3LsuWLWPhwoXceuutuDuTJk2iffv2FBUV/WgARIArrriCWbNmAbBhwwY2bNhAdnY29913H+effz5Lly5l4cKFjBs3jp07d8Zug0idprAQiZKqhgb/4IMPGDJkSMVDkTZsiDwLbPHixQwZMgSIjGS7j7tz++2307VrVy644ALWr1/Pxo0bD/m5l19+ecUhqVmzZjF48GAg8syLSZMmkZWVxXnnnUdZWRnr1q2r7a8t9ZROcItEyYFDg2/cuJGWLVtWPHypJvLz89m8eTOFhYWkpqaSkZFBWVnZId/Ttm1bWrduzfLly5k5c2bF0/TcnTlz5nDmmWce0feR5KY9C5EYadGiBZmZmbz66qtA5I/3J598AkTOM8yZMweAV155peI927dv58QTTyQ1NZWFCxdWDHUeNrz4FVdcwZ/+9Ce2b99O165dgchd6I8++ij7Bg/9+OOPa/9LSr2lPQup9xLhUtd98vPzuf7665k4cSK7d+9m6NChdOvWjUceeYThw4dz3333MWDAAI477jgAhg0bxiWXXEKX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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "sns.countplot(x='overall', hue='swn_sentiment' ,data = reviews)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 16,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<AxesSubplot:xlabel='swn_sentiment', ylabel='overall'>"
      ]
     },
     "execution_count": 16,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "sns.boxenplot(x='swn_sentiment', y='overall', data = reviews)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 17,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 864x504 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "fig, ax = plt.subplots(nrows=1, ncols=1, figsize = (12,7))\n",
    "sns.boxenplot(x='overall', y='swn_score', data = reviews, ax=ax)\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 18,
   "metadata": {},
   "outputs": [],
   "source": [
    "reviews['true_sentiment'] = \\\n",
    "    reviews['overall'].apply(lambda x: \"positive\" if x>=4 else (\"neutral\" if x==3 else \"negative\"))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 19,
   "metadata": {},
   "outputs": [],
   "source": [
    "y_swn_pred, y_true = reviews['swn_sentiment'].tolist(), reviews['true_sentiment'].tolist()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 20,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "(4499, 4499)"
      ]
     },
     "execution_count": 20,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "len(y_swn_pred), len(y_true)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 21,
   "metadata": {},
   "outputs": [],
   "source": [
    "from sklearn.metrics import confusion_matrix\n",
    "cm = confusion_matrix(y_true, y_swn_pred)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 22,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 576x432 with 2 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "fig , ax = plt.subplots(nrows=1, ncols=1, figsize=(8,6))\n",
    "sns.heatmap(cm, cmap='viridis_r', annot=True, fmt='d', square=True, ax=ax)\n",
    "ax.set_xlabel('Predicted')\n",
    "ax.set_ylabel('True');"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Performance Assessment"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Negative Sentiment Prediction Assessment"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 23,
   "metadata": {},
   "outputs": [],
   "source": [
    "tp, tn, fp, fn = 1088, 70+242+142+1310, 265+647, 188+547"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 24,
   "metadata": {},
   "outputs": [],
   "source": [
    "recall = tp / (tp+fn)\n",
    "specifity = tn / (tn+fp)\n",
    "precision = tp/(tp+fp)\n",
    "f1 = (2*tp) / (2*tp + fp + fn)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 25,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "recall: 0.5968184311574328\n",
      "precission: 0.544\n",
      "f1 score: 0.5691865027465342\n"
     ]
    }
   ],
   "source": [
    "print(\"recall: {}\\nprecission: {}\\nf1 score: {}\".format(recall, precision, f1))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Positive Sentiment Prediction Assessment"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 26,
   "metadata": {},
   "outputs": [],
   "source": [
    "tp, tn, fp, fn = 1310, 1088+265+70+188, 242+647, 142+547"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 27,
   "metadata": {},
   "outputs": [],
   "source": [
    "recall = tp / (tp+fn)\n",
    "specifity = tn / (tn+fp)\n",
    "precision = tp/(tp+fp)\n",
    "f1 = (2*tp) / (2*tp + fp + fn)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 28,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "recall: 0.655327663831916\n",
      "precission: 0.5957253296953161\n",
      "f1 score: 0.6241067174845164\n"
     ]
    }
   ],
   "source": [
    "print(\"recall: {}\\nprecission: {}\\nf1 score: {}\".format(recall, precision, f1))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "## Sentiment Scoring Model Using NLTK Opinion Lexicon"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 29,
   "metadata": {},
   "outputs": [],
   "source": [
    "import nltk\n",
    "from nltk.corpus import opinion_lexicon\n",
    "from nltk.tokenize import word_tokenize, sent_tokenize"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 30,
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "[nltk_data] Downloading package opinion_lexicon to\n",
      "[nltk_data]     /Users/koosha.tahmasebipour/nltk_data...\n",
      "[nltk_data]   Package opinion_lexicon is already up-to-date!\n"
     ]
    },
    {
     "data": {
      "text/plain": [
       "True"
      ]
     },
     "execution_count": 30,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "nltk.download(\"opinion_lexicon\")"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 31,
   "metadata": {},
   "outputs": [],
   "source": [
    "pos_words = list(opinion_lexicon.positive())\n",
    "neg_words = list(opinion_lexicon.negative())"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 32,
   "metadata": {},
   "outputs": [],
   "source": [
    "def get_sentiment_score_oplex(text):\n",
    "    \n",
    "    \"\"\"\n",
    "        This method returns the sentiment score of a given text using nltk opinion lexicon.\n",
    "        input: text\n",
    "        output: numeric (double) score, >0 means positive sentiment and <0 means negative sentiment.\n",
    "    \"\"\"    \n",
    "    total_score = 0\n",
    "\n",
    "    raw_sentences = sent_tokenize(text)\n",
    "    \n",
    "    for sentence in raw_sentences:\n",
    "\n",
    "        sent_score = 0     \n",
    "        sentence = str(sentence)\n",
    "        sentence = sentence.replace(\"<br />\",\" \").translate(str.maketrans('','',punctuation)).lower()\n",
    "        tokens = TreebankWordTokenizer().tokenize(text)\n",
    "        for token in tokens:\n",
    "            sent_score = sent_score + 1 if token in pos_words else (sent_score - 1 if token in neg_words else sent_score)\n",
    "        total_score = total_score + (sent_score / len(tokens))\n",
    "\n",
    "    \n",
    "    return total_score"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 33,
   "metadata": {},
   "outputs": [],
   "source": [
    "reviews['oplex_sentiment_score'] = reviews['reviewText'].apply(lambda x: get_sentiment_score_oplex(x))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 34,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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JJnmf2eSz21tyQAAAAAcyyICpk8leVhVfes0ltL5ST6a5G1JLpyWuTjJG6bn107Tmea/tbt7C+sFAAAA4DgWMQbT9VkdrPt9ST401XBlkucmuayqDmZ1jKWXTqu8NMk9p/bLkjxvq2sGAAAA4Nj2nHiRzdfdz0/y/KOaP5HkIWss+9dJnrwVdQEAAACwcYu4RA4AAACAHUTABAAAAMAQARMAAAAAQwRMAAAAAAwRMAEAAAAwRMAEAAAAwBABEwAAAABDBEwAAAAADBEwAQAAADBEwAQAAADAEAETAAAAAEMETAAAAAAMETABAAAAMETABAAAAMAQARMAAAAAQwRMAAAAAAwRMAEAAAAwRMAEAAAAwBABEwAAAABDBEwAAAAADBEwAQAAADBEwAQAAADAEAETAAAAAEMETAAAAAAMETABAAAAMETABAAAAMAQARMAAAAAQwRMAAAAAAwRMAEAAAAwRMAEAAAAwBABEwAAAABDBEwAAAAADBEwAQAAADBEwAQAAADAEAETAAAAAEMETAAAAAAMETABAAAAMETABAAAAMAQARMAAAAAQwRMAAAAAAwRMAEAAAAwRMAEAAAAwBABEwAAAABDBEwAAAAADBEwAQAAADBEwAQAAADAEAETAAAAAEMETAAAAAAMETABAAAAMETABAAAAMAQARMAAAAAQwRMAAAAAAwRMAEAAAAwRMAEAAAAwBABEwAAAABDBEwAAAAADBEwAQAAADBEwAQAAADAEAETAAAAAEMETAAAAAAMETABAAAAMETABAAAAMAQARMAAAAAQwRMAAAAAAwRMAEAAAAwRMAEAAAAwBABEwAAAABDBEwAAAAADBEwAQAAADBEwAQAAADAEAETAAAAAEMETAAAAAAMETABAAAAMETABAAAAMAQARMAAAAAQwRMAAAAAAwRMAEAAAAwRMAEAAAAwBABEwAAAABDBEwAAAAADBEwAQAAADBEwAQAAADAEAETAAAAAEMETAAAAAAMETABAAAAMETABAAAAMAQARMAAAAAQwRMAAAAAAwRMAEAAAAwRMAEAAAAwBABEwAAAABDBEwAAAAADBEwAQAAADBEwAQAAADAEAETAAAAAEMETAAAAAAMETABAAAAMETABAAAAMAQARMAAAAAQxYSMFXV3avqmqr6k6q6qaoeXlX3qKrrqurm6efp07JVVb9VVQer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\n",
      "text/plain": [
       "<Figure size 1440x720 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "fig , ax = plt.subplots(nrows=1, ncols=1, figsize=(20,10))\n",
    "sns.histplot(x='oplex_sentiment_score',\\\n",
    "             data=reviews.query(\"oplex_sentiment_score < 1 and oplex_sentiment_score>-1\"), ax=ax)\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 35,
   "metadata": {},
   "outputs": [],
   "source": [
    "reviews['oplex_sentiment'] = \\\n",
    "    reviews['oplex_sentiment_score'].apply(lambda x: \"positive\" if x>0.1 else (\"negative\" if x<0 else \"neutral\"))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 36,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "positive    1875\n",
       "neutral     1582\n",
       "negative    1042\n",
       "Name: oplex_sentiment, dtype: int64"
      ]
     },
     "execution_count": 36,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "reviews['oplex_sentiment'].value_counts(dropna=False)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 37,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<AxesSubplot:xlabel='overall', ylabel='count'>"
      ]
     },
     "execution_count": 37,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "iVBORw0KGgoAAAANSUhEUgAAAYUAAAEGCAYAAACKB4k+AAAAOXRFWHRTb2Z0d2FyZQBNYXRwbG90bGliIHZlcnNpb24zLjMuMywgaHR0cHM6Ly9tYXRwbG90bGliLm9yZy/Il7ecAAAACXBIWXMAAAsTAAALEwEAmpwYAAAdtElEQVR4nO3de3QV9b338fdXLgYUBAGtEDSxVS4SiBBTLM2RmopoEYoFoYVKioggWLxx4LTYVpau0pYClVI5VitqQYNBKo+Pq8c2olIEaRKjXG3RJ2KQCkbkCEgh8n3+2JMxQIIbyL4k+bzWysrc5zsjOx/nNzO/be6OiIgIwGmJLkBERJKHQkFEREIKBRERCSkUREQkpFAQEZFQ00QXcCrat2/vaWlpiS5DRKReKS4u/tDdO9Q0r16HQlpaGkVFRYkuQ0SkXjGzd2ubp+YjEREJKRRERCSkUBARkVC9vqcgjcOhQ4coLy/nwIEDiS6lQUtJSSE1NZVmzZoluhRJIIWCJL3y8nJatWpFWloaZpbochokd6eiooLy8nLS09MTXY4kkJqPJOkdOHCAdu3aKRBiyMxo166drsZEoSD1gwIh9nSOBRQKIiJSjUJBRERCutEsjUL//v2ZPXs2WVlZiS4lVFpayvvvv8+1114LwIoVK9i0aRPTp0+P2T5feuklmjdvzte+9rWY7UNOTb/5/epkO6tvW31S6+lKQSRBSktLef7558PxwYMHxzQQIBIKr776akz3IfWbQkHqrTlz5tCjRw969OjBvHnzKCsro2vXrowaNYpu3boxbNgw9u/ff8x6L7zwApdffjm9e/dm+PDh7N27l3fffZeLLrqIDz/8kMOHD5OTk8MLL7xQ43737dvHt771LXr16kWPHj3Iz88HoLi4mCuuuII+ffpw9dVXs2PHDiBylTJt2jSys7O5+OKLWbVqFQcPHuQnP/kJ+fn5ZGZmkp+fz6JFi5g8eTIAeXl5TJw4kb59+3LhhRfy0ksvMXbsWLp160ZeXt5xjwUi/YL99Kc/pXfv3mRkZLBlyxbKyspYuHAhc+fOJTMzk1WrVtXlfw5pIBQKUi8VFxfz6KOP8tprr7F27Vp+//vfs3v3bt566y1uvfVWNm/eTOvWrfnd7353xHoffvgh9913H3/9618pKSkhKyuLOXPmcMEFFzBt2jQmTpzIr3/9a7p3786AAQNq3Pef//xnOnbsyBtvvMGGDRsYOHAghw4d4rbbbqOgoIDi4mLGjh3Lj3/843CdyspK1q1bx7x587j33ntp3rw5M2fOZMSIEZSWljJixIhj9rN7927WrFnD3LlzGTx4MHfccQcbN25k/fr1lJaW1nosVdq3b09JSQkTJ05k9uzZpKWlMWHCBO644w5KS0vJycmpo/8a0pDonoLUS3/7298YOnQoZ5xxBgDXX389q1atonPnzvTrF2mTHT16NA888AB33313uN7atWvZtGlTuMzBgwe5/PLLARg3bhxPP/00CxcupLS0tNZ9Z2RkcNdddzFt2jQGDRpETk4OGzZsYMOGDVx11VUAfPbZZ5x33nnhOtdffz0Affr0oaysLKpjvO666zAzMjIyOPfcc8nIyADgkksuoaysjPLy8lqP5eh9PvPMM1HtU0ShIA3K0c/aHz3u7lx11VU8+eSTx6y7f/9+ysvLAdi7dy+tWrWqcR8XX3wxJSUlPP/888yYMYPc3FyGDh3KJZdcwpo1a2pc5/TTTwegSZMmVFZWRnUsVeucdtpp4XDVeGVlJU2aNKn1WE52nyJqPpJ6KScnhz/96U/s37+fffv2sXz5cnJycti2bVv4h3nJkiV8/etfP2K9vn37snr1arZu3QpE7g/84x//AGDatGmMGjWKmTNncvPNN9e67/fff5+WLVsyevRopk6dSklJCV26dGHXrl3hvg8dOsTGjRuPewytWrXik08+OelzcLxjidU+peFTKEi91Lt3b/Ly8sjOzuarX/0q48aNo23btnTp0oUFCxbQrVs3du/ezcSJE49Yr0OHDixatIjvfve79OzZk8svv5wtW7bw8ssv8/e//z0MhubNm/Poo4/WuO/169eTnZ1NZmYm9957LzNmzKB58+YUFBQwbdo0evXqRWZm5hc+5fONb3yDTZs2hTeaT1Rtx3I81113HcuXL9eNZqmVuXuiazhpWVlZrm9ea/g2b95Mt27dvnC5srIyBg0axIYNG+JQVcMU7bmW2InHewpmVuzuNb60oysFEREJ6UazNBhpaWl1epVQUVFBbm7uMdMLCwtp165dne1HJJkoFERq0a5du+M+mirSEKn5SEREQgoFEREJKRRERCQU03sKZnYHMA5wYD3wA+A84CmgHVAMfN/dD5rZ6cDjQB+gAhjh7mWxrE8alj5TH6/T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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "sns.countplot(x='overall', hue='oplex_sentiment' ,data = reviews)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 38,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "<AxesSubplot:xlabel='oplex_sentiment', ylabel='overall'>"
      ]
     },
     "execution_count": 38,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "sns.boxenplot(x='oplex_sentiment', y='overall', data = reviews)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 39,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 864x504 with 1 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "fig, ax = plt.subplots(nrows=1, ncols=1, figsize = (12,7))\n",
    "sns.boxenplot(x='overall', y='oplex_sentiment_score', data = reviews, ax=ax)\n",
    "plt.show()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 40,
   "metadata": {},
   "outputs": [],
   "source": [
    "y_oplex_pred = reviews['oplex_sentiment'].tolist()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 41,
   "metadata": {},
   "outputs": [],
   "source": [
    "oplex_cm = confusion_matrix(y_true, y_oplex_pred)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 42,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 576x432 with 2 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "fig , ax = plt.subplots(nrows=1, ncols=1, figsize=(8,6))\n",
    "sns.heatmap(oplex_cm, cmap='viridis_r', annot=True, fmt='d', square=True, ax=ax)\n",
    "ax.set_xlabel('Predicted')\n",
    "ax.set_ylabel('True');"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 43,
   "metadata": {},
   "outputs": [],
   "source": [
    "oplex_cm = list(oplex_cm.ravel())"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 44,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "[804, 701, 495, 106, 195, 199, 132, 686, 1181]"
      ]
     },
     "execution_count": 44,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "oplex_cm"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Negative Label Assessment"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 47,
   "metadata": {},
   "outputs": [],
   "source": [
    "tp, tn, fp, fn = 804, 195+199+686+1181, 106+132, 701+495"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 48,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "recall: 0.402\n",
      "precission: 0.7715930902111324\n",
      "f1 score: 0.5285996055226825\n"
     ]
    }
   ],
   "source": [
    "recall = tp / (tp+fn)\n",
    "specifity = tn / (tn+fp)\n",
    "precision = tp/(tp+fp)\n",
    "f1 = (2*tp) / (2*tp + fp + fn)\n",
    "\n",
    "print(\"recall: {}\\nprecission: {}\\nf1 score: {}\".format(recall, precision, f1))"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Positive Label Assessment"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 49,
   "metadata": {},
   "outputs": [],
   "source": [
    "tp, tn, fp, fn = 1181, 804+701+106+195, 495+199, 132+686"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 50,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "recall: 0.5907953976988495\n",
      "precission: 0.6298666666666667\n",
      "f1 score: 0.6097057305110997\n"
     ]
    }
   ],
   "source": [
    "recall = tp / (tp+fn)\n",
    "specifity = tn / (tn+fp)\n",
    "precision = tp/(tp+fp)\n",
    "f1 = (2*tp) / (2*tp + fp + fn)\n",
    "\n",
    "print(\"recall: {}\\nprecission: {}\\nf1 score: {}\".format(recall, precision, f1))"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": null,
   "metadata": {},
   "outputs": [],
   "source": []
  }
 ],
 "metadata": {
  "kernelspec": {
   "display_name": "Python 3",
   "language": "python",
   "name": "python3"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.6.12"
  }
 },
 "nbformat": 4,
 "nbformat_minor": 4
}