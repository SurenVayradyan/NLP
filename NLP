
import pandas as pd
from spacy import displacy
from spacy.matcher import Matcher
import spacy
import re


class dialog_parser():
    def __init__(self,data_path):
        self.load_data(data_path)
        self.init_spacy()

    # загрузка данных
    def load_data(self,data_path):
        #загружаем диалоги
        self.df_dlg = pd.read_csv(data_path)
        self.df_dlg["comments"] ="" #сюда будем писать анализ, если есть реакция на инфу - приветствие, имя манагера, компании и прощание

    #тут инициализиируем  spacy и создаем паттерны
    def init_spacy(self):
        self.nlp = spacy.load('ru_core_news_md')
        self.matcher_comp = Matcher(self.nlp.vocab)
        my_pattern = [{"LEMMA": "компания"}, {'POS': {"IN": ['NOUN', 'ADV', 'VERB', 'ADJ', 'PROPN']}}]
        self.matcher_comp.add("Compamies_", [my_pattern])

        self.matcher_greets = Matcher(self.nlp.vocab)
        p1 = [{'TEXT': 'здравствуйте'}]
        p2 = [{'LEMMA': 'добрый'}, {'LEMMA':'день'}]
        p3 = [{'TEXT': 'приветствую'}]

        self.matcher_greets.add("Greetings_", [p1,p2,p3])

        self.matcher_goodby = Matcher(self.nlp.vocab)
        p1 = [{'TEXT': 'всего'},{'TEXT': 'доброго'} ]
        p2 = [{'TEXT': 'до'},{'TEXT': 'свидания'} ]
        p3 = [{'TEXT': 'всего'},{'TEXT': 'хорошего'} ]
        self.matcher_goodby.add("good_by_", [p1,p2,p3])

        self.matcher_intro = Matcher(self.nlp.vocab)
        p1 = [{'TEXT': 'меня'},{'TEXT': 'зовут'},{'POS':'PROPN'} ]
        p2 = [{'TEXT': 'меня'},{'POS':'PROPN'},{'TEXT': 'зовут'} ]
        self.matcher_intro.add("intro_", [p1,p2])


    #проверка имени компании по паттерну
    def match_company_name(self, text):
        document = self.nlp(text.lower())
        matches = self.matcher_comp(document)
        for match_id, start, end in matches:
            span = document[start:end]
            return span.text
        return None

    def match_greetings_phrase(self,text):
        document = self.nlp(text.lower())
        matches = self.matcher_greets(document)
        for match_id, start, end in matches:
            span = document[start:end]
            return span.text
        return None

    def match_goodby_phrase(self,text):
        document = self.nlp(text.lower())
        matches = self.matcher_goodby(document)
        for match_id, start, end in matches:
            span = document[start:end]
            return span.text
        return None


    def match_name(self,text):
        document = self.nlp(text.lower())
        matches = self.matcher_intro(document)
        for match_id, start, end in matches:
            span = document[start:end]
            return self.extract_name_from_match(span.text)
        return None

    def extract_name_from_match(self,text):
        document = self.nlp(text)
        for ents in document.ents:
            if (ents.label_ == 'PER'):
                return (ents.text)

    #анализ диалогов на нужные нам значения
    def dialogs_analisys(self):
        dlg_id = -1
        greetings = False
        goodbay = False
        manager_name = False
        company_name = False
        #если нужные значения находятся, они записываются в доп колонку comments
        for index, row in self.df_dlg.iterrows():
            if dlg_id!=row["dlg_id"]:
                greetings = False
                goodbay = False
                manager_name = False
                company_name = False
                dlg_id = row["dlg_id"]

            if row['role']=='client':
                continue

            #приветствие
            if not greetings:
                if self.match_greetings_phrase(row['text']) != None:
                    self.df_dlg.at[index, 'comments'] = row['comments'] + '/ ' + "greetings"
                    greetings = True
            #ищем имя компании
            if not company_name:
                result =self.match_company_name(row['text'])
                if result!=None:
                    self.df_dlg.at[index, 'comments'] = self.df_dlg.at[index, 'comments'] + '/ ' + "company name: " + result
                    company_name = True
            #ищем имя менеджера
            if not manager_name:
                result = self.match_name(row['text'])
                if result!=None:
                    self.df_dlg.at[index, 'comments'] = self.df_dlg.at[index, 'comments'] + '/ ' + "manager name: " + result
                    manager_name = True
            #ищем слова прощания
            if not goodbay:
                if self.match_goodby_phrase(row['text'])!=None:
                    self.df_dlg.at[index, 'comments'] = self.df_dlg.at[index, 'comments'] + '/ ' + "goodbye"
                    goodbay = True
        dp.df_dlg.to_csv("data_analyse.csv")


dp = dialog_parser("test_data.csv")
dp.dialogs_analisys()


