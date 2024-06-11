# Topic modeling

## Summary

The latent Dirichlet allocation model was trained on the dataset of news in russian language. Visualization of probability distributions of words within topics and topics across documents was created. The trained model CV coherence score is **0.45**.

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

The dataset was preprocessed and saved to the file `datasets/processed_dataset.json`. This file is a list of objects of the following structure:
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