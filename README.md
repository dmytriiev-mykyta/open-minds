# Topic modeling

## Summary

The latent Dirichlet allocation model was trained on the dataset of news in russian language. Visualization of probability distributions of words within topics and topics across documents was created. The trained model CV coherence score is **0.45**. The solution could be found in the file **identify_topics.ipynb**.

## The problem
There is a dataset of news messages in russian language collected from various sources. The task is to identify topics of these messages and key words that are characteristic for each topic.

## Dataset

The dataset consists of 12097 rows. Each row contains two columns -- **fullText** and **pubTime**. Each entry in the **fullText** column may contain some information beside a news message. It was found out that footnotes and tags sequences are present in some news entries.

## Dataset preprocessing

Some news messages in the initial dataset contain incomplete words, as, for example, the one given below:
```italic
Специалисты столичного комплекса \nского хозяйства начали подготавливать \nские фонтаны к новому сезону. Об этом сообщил заммэра по вопросам ЖКХ и благоустройства Петр Бирюков.По словам заммэра, фонтаны по праву считаются визитной карточкой и украшением Москвы. Есть как традиционные сооружения, так и "сухие", плавающие, музыкальные и светодинамические, указал Бирюков. Он отметил, что обычно фонтаны запускают в конце апреля, поэтому специалисты уже проводят подготовительные мероприятия.Бирюков добавил, что после консервации фонтанов на зиму их конструкции, находящиеся под землей и в коллекторах сооружений, проверили. В данный момент, продолжил он, начался плановый ремонт электромеханического оборудования и подводной подсветки, систем управления и автоматики всех коммуникаций, кроме того, проводится диагностика гидравлики. Помимо этого, специалисты приводят в порядок насосное оборудование, фильтры, струеобразующие элементы, проверяются системы вентиляции и отопления помещений насосных станций, уточнил Бирюков.В заключение он сказал, что на финальном этапе будут подготовлены внешние чаши фонтанов. Все сооружения расконсервируют, промоют после зимы специальным шампунем, отрегулируют форсунки и проверят лампы внешней подсветки. Если будет необходимо, специалисты также проведут ремонт облицовки чаш и парапетов.Фонтаны были законсервированы на зиму в конце октября прошлого года. Из всех сооружений слили воду, очистили скульптуры, чаши и оборудование, а также промыли конструкции. Также были убраны фонари подводной подсветки и струеобразующих элементов.\n\t\t\t\t\t\t\t\n\t\t\t\t\t\t\t\n\t\t\t\t\t\t\t\t\n\t\t\t\t\t\t\t\n\t\t\t\t\t\t\tЕщё больше новостей — в телеграм-канале Москва 24 Подписывайтесь!\nTags: город
```
Here the `город` root was replaced with the new line character in the first sentence of the passage. Seven such entries were found in the dataset. Incomplete words in these entries were manually corrected, so completed version of the dataset was created and saved in the file *datasets/processed_test_assignment_data.csv*.

Automatic text segmentation was performed over each news entry to extract from it a news message, sequence of tags and footnotes. We applied further processing to these pieces of information.

### News messages processing

We found out that many news messages contain the same phrases with references to a news source. Since such phrase does not contain any information specific to a corresponding news message, we consider it irrelevant, or "stop phrase". Several stop phrases were identified:
 1. `Ещё больше новостей — в телеграм-канале Москва 24 Подписывайтесь!`
 2. `Подробнее – в эфире телеканала Москва 24.`
 3. `РИА «Сахалин-Курилы»`
There are also references to the agency `РИА Новости`, websites and photo/video credits that could be described by a regular expression. We find such references by their pattern and delete them.

Some news message are written in Tatar language. We omit them from our consideration for now. 

When irrelevant information is deleted, the text is tokenized. Each token is lemmatized, and only alphabetical tokens are retained. All tokens are transformed to a lower case. Tokens that are stop words were omitted. In our analysis collection of stop words consists of the set provided in the NLTK library and a number of words selected from the corpus manually.

All news were published from the 19th to the 21st of February, 2024. Hence, the month name mentions in the news are highly likely to be common across the corpus, so they are irrelevant for topic modeling and we add the word `февраль` to the stop words list.

New line characters were deleted from the corpus. Sequences of several spaces and tabulations were replaced with single spaces, as well as all tabulations.

### Tags processing
We identify sequences of tags by the key word `Tags:`. Each sequence is split into separate tags, and only tags that contain a single word are retained. Hashtag prefix `#` is deleted from the tags.

### Footnotes processing

We do not process footnotes in our analysis.

### Serialization of the preprocessed dataset

The dataset was preprocessed and saved to the file **datasets/processed_dataset.json**. This file is a list of objects of the following structure:
```json
{
    "text": "<preprocessed text of a news message>", 
    "tokens": <list of lemmatized tokens of the preprocessed news message>, 
    "tags": <list of tags found in an initial text of the news entry>, 
    "footnotes": <list of footnotes found in an initial text of the news entry>
}
```

## Solution

We apply the latent Dirichlet allocation (LDA) approach to identify main topics and corresponding key words in the given corpus. We choose the LDA in our analysis since it is unsupervised. Within the LDA, probability distributions of topics over documents and words over topics are derived.

We trained an LDA model with hyperparameters `no_below=5`, `no_above=0.9` and `num_topics = 30`, since CV coherence score for this model is the highest among the tested ones.

Topics identified within the trained model were displayed with the *pyLDAvis* utility. Probabilities distributions of words within topics are shown in the bar chart in the right part of the visualization. Probabilities of topics across the documents are displayed as sizes of circles in the left part of the visualization.

The described solution reaches CV coherence score of **0.45**, so its quality is poor. There might be more stop words in the corpus, which are not deleted at the preprocessing step. We propose to identify these words and add them to the list `TextPreprocessor.stop_words`

## Discussion

In this analysis we identified several topics that allow clear interpretation. Here is a list of selected topics with proposed interpretations:
| Topic number | Ten most probable words of a topic | Interpretation |
| :---: | :---: | :---: |
| 1 | `0.136*"украина" + 0.081*"сша" + 0.039*"заявить" + 0.036*"американский" + 0.029*"киев" + 0.028*"страна" + 0.027*"ранее" + 0.023*"нато" + 0.021*"президент" + 0.019*"помощь"` | Relationships between Ukraine and the USA in the context of international military aid in repelling russian aggression |
| 11 | `0.099*"врач" + 0.064*"продукт" + 0.056*"здоровье" + 0.041*"изменение" + 0.034*"регулярный" + 0.032*"почему" + 0.032*"учёный" + 0.027*"специалист" + 0.027*"часто" + 0.025*"организм"` | Healthcare advices |
| 14 | `0.164*"банк" + 0.052*"организация" + 0.051*"бизнес" + 0.038*"физический" + 0.036*"средство" + 0.032*"счёт" + 0.031*"кредитный" + 0.029*"россия" + 0.026*"ограничение" + 0.025*"лицо"` | Banking policy |
| 16 | `0.279*"ребёнок" + 0.175*"школа" + 0.158*"семья" + 0.056*"детский" + 0.054*"родитель" + 0.034*"школьник" + 0.027*"ученик" + 0.020*"учитель" + 0.016*"школьный" + 0.013*"фонд"` | School education |
| 18 | `0.053*"температура" + 0.048*"среда" + 0.040*"неделя" + 0.039*"градус" + 0.038*"снег" + 0.037*"день" + 0.033*"воздух" + 0.031*"ночь" + 0.030*"погода" + 0.025*"подмосковье"` | Weather forecast |
| 19 | `0.095*"компания" + 0.068*"рынок" + 0.040*"экономика" + 0.040*"труд" + 0.033*"цб" + 0.033*"дальнейший" + 0.025*"эксперт" + 0.025*"основный" + 0.024*"рост" + 0.024*"признаться"` | Economical policy |
| 22 | `0.138*"дом" + 0.054*"здание" + 0.044*"улица" + 0.042*"огонь" + 0.040*"площадь" + 0.039*"пожар" + 0.036*"жилой" + 0.035*"метр" + 0.029*"техника" + 0.020*"мчс"` | Fire accidents |
| 24 | `0.110*"россия" + 0.109*"президент" + 0.082*"путин" + 0.077*"владимир" + 0.038*"глава" + 0.032*"государство" + 0.031*"встреча" + 0.027*"российский" + 0.026*"рф" + 0.026*"международный"` | Politics |
| 30 | `0.059*"игра" + 0.043*"команда" + 0.037*"матч" + 0.034*"спорт" + 0.025*"соревнование" + 0.025*"победа" + 0.024*"сезон" + 0.024*"чемпионат" + 0.023*"клуб" + 0.021*"первый"` | Sports news |
| 49 | `0.050*"военный" + 0.042*"всу" + 0.041*"авдеевка" + 0.038*"сила" + 0.036*"российский" + 0.024*"оборона" + 0.024*"украина" + 0.023*"войско" + 0.022*"вооружить" + 0.021*"россия"` | Representation of a russian war against Ukraine in news format |