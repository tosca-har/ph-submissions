---
title: Normaliser des données textuelles avec Python
slug: normaliser-donnees-textuelles-python
original: normalizing-data
layout: lesson
collection: lessons
date: 2012-07-17
translation_date: YYYY-MM-DD
authors:
- William J. Turkel
- Adam Crymble
reviewers:
- Jim Clifford
- Francesca Benatti
- Frederik Elwert
editor:
- Miriam Posner
translator:
- Fantine Horvath
translation-editor:
- Forename Surname
translation-reviewer:
- Forename Surname
- Forename Surname
review-ticket: https://github.com/programminghistorian/ph-submissions/issues/564
difficulty: 2
activity: transforming
topics: [python]
abstract: L'objectif de cette leçon est de reprendre la liste créée dans la leçon &laquo;&nbsp;Du HTML à une liste de mots&nbsp;&raquo; et de la rendre plus simple à analyser en normalisant ses données
avatar_alt: Grande femme trainant un homme plus petit
doi: XX.XXXXX/phen0000
---

{% include toc.html %}

## Objectif de la leçon

Avant d'aller plus loin, nous avons besoin de *normaliser* la liste que nous avons créé dans la leçon [Du html à une liste de mots (2)](https://programminghistorian.org/en/lessons/from-html-to-list-of-words-2). La normalisation des données est une étape importante qui consiste à préparer les données pour le traitement automatique que l'on veut leur appliquer, en leur donnant une forme que nous pourrons manipuler facilement (par exemple, *normaliser* des données textuelles peut nécessiter de convertir tous les caractères en minuscules ou de retirer des caractères spéciaux qui ne nous intéressent pas pour la suite). Pour cela, nous allons appliquer des méthodes de traitement des chaînes de caractères, ainsi que des expressions régulières de Python. Une fois normalisées, nos données pourront être analysées plus facilement.

## Fichiers nécessaires pour cette leçon

-   `html-to-list-1.py`
-   `obo.py`

Si vous n'avez pas les fichiers de la leçon précédente citées ci-dessus, vous pouvez télécharger le fichier python-lessons3.zip ici : [`python-lessons3.zip`](https://github.com/programminghistorian/jekyll/blob/gh-pages/assets/python-lessons3.zip).

## Nettoyer notre liste

Dans la leçon [Du html à une liste de mots (2)](https://programminghistorian.org/en/lessons/from-html-to-list-of-words-2), nous avons rédigé un programme Python nommé `html-to-list-1.py`, qui après avoir téléchargé une [page web](http://www.oldbaileyonline.org/browse.jsp?id=t17800628-33&div=t17800628-33) en extrait le format et les métadonnées, et renvoie une liste de &laquo;&nbsp;mots&nbsp;&raquo; tel que présentée plus bas. En réalité, ces entités sont appellées des &laquo;&nbsp;*tokens*&nbsp;&raquo;, ou &laquo;&nbsp;jetons&nbsp;&raquo;, plutôt que &laquo;&nbsp;mots&nbsp;&raquo;. En effet, certains de ces éléments ne sont pas du tout des &laquo;&nbsp;mots&nbsp;&raquo; à proprement parler (par exemple, l'abbrévation &c. pour &laquo;&nbsp;etcetera&nbsp;&raquo;). D'autre peuvent aussi être considérés comme des groupes de plusieurs mots. Dans la liste qui suit, la forme possessive &laquo;&nbsp;Akerman's&nbsp;&raquo; par exemple est parfois analysée par les linguistes comme deux mots : &laquo;&nbsp;Akerman&nbsp;&raquo; accompagné d'un marqueur possessif. En français, on pourrait trouver de la même façon des formes analysables comme deux mots mais récupérées comme un token unique par le programme Python (des verbes pronominaux par exemple, comme &laquo;&nbsp;s'élancer&nbsp;&raquo;).

Reprenez votre programme `html-to-list-1.py` et vérifiez qu'il renvoie bien quelque chose comme suit :

``` python
['324.', '\xc2\xa0', 'BENJAMIN', 'BOWSEY', '(a', 'blackmoor', ')', 'was',
'indicted', 'for', 'that', 'he', 'together', 'with', 'five', 'hundred',
'other', 'persons', 'and', 'more,', 'did,', 'unlawfully,', 'riotously,',
'and', 'tumultuously', 'assemble', 'on', 'the', '6th', 'of', 'June', 'to',
'the', 'disturbance', 'of', 'the', 'public', 'peace', 'and', 'did', 'begin',
'to', 'demolish', 'and', 'pull', 'down', 'the', 'dwelling', 'house', 'of',
'\xc2\xa0', 'Richard', 'Akerman', ',', 'against', 'the', 'form', 'of',
'the', 'statute,', '&amp;c.', '\xc2\xa0', 'ROSE', 'JENNINGS', ',', 'Esq.',
'sworn.', 'Had', 'you', 'any', 'occasion', 'to', 'be', 'in', 'this', 'part',
'of', 'the', 'town,', 'on', 'the', '6th', 'of', 'June', 'in', 'the',
'evening?', '-', 'I', 'dined', 'with', 'my', 'brother', 'who', 'lives',
'opposite', 'Mr.', "Akerman's", 'house.', 'They', 'attacked', 'Mr.',
"Akerman's", 'house', 'precisely', 'at', 'seven', "o'clock;", 'they',
'were', 'preceded', 'by', 'a', 'man', 'better', 'dressed', 'than', 'the',
'rest,', 'who']
```

En soi, séparer ainsi le texte en mots n'est pas très utile, mais c'est une étape nécessaire. Nous pouvons maintenant utiliser le texte pour réaliser des mesures qui ne sont normalement pas faisables sans logiciels spécifiques. Nous allons commencer par calculer les fréquences des tokens et d'autres unités linguistiques, ce qui se fait fréquemment dans le cadre d'une analyse textuelle.

Il est clair que notre liste va avoir besoin d'être nettoyée avant d'être utilisée pour mesurer des fréquences. A l'aide des méthodes vues dans la leçon précédente [Du html à une liste de mots (1)](https://programminghistorian.org/fr/lecons/du-html-a-une-liste-de-mots-1), essayons dans un premier temps de décrire notre algorithme avec des phrases en français. Ce que nous voulons, c'est savoir combien de fois les mots sémantiquement importants apparaissent dans notre texte. Les étapes à réaliser devraient donc ressembler à cela :

-   Convertir tous les mots en minuscules, pour que &laquo;&nbsp;BENJAMIN&nbsp;&raquo; et
    &laquo;&nbsp;benjamin&nbsp;&raquo; soient comptés comme un seul token
-   Retirer tout caractère qui ne ferait pas partie des caractères qui nous intéressent (les emojis par exemple)
-   Compter, pour chaque mot, le nombre de fois où il apparaît
    (son nombre d'occurrences)
-   Retirer les mots outils (stopwords), des mots à faible poids sémantique mais très courants, 
    comme &laquo;&nbsp;it&nbsp;&raquo;, &laquo;&nbsp;the&nbsp;&raquo;, &laquo;&nbsp;and&nbsp;&raquo;, etc.
    (en français, &laquo;&nbsp;le&nbsp;&raquo;, &laquo;&nbsp;et&nbsp;&raquo;, &laquo;&nbsp;un&nbsp;&raquo;, etc.)


## Convertir en minuscules

Généralement, les tokens sont convertis en minuscules pour faire des mesures de fréquences, c'est ce que nous allons faire en appliquant la méthode `lower()` à chaque token, qui a été introduit dans la leçon [Manipuler des chaînes de caractères en Python](https://programminghistorian.org/fr/lecons/manipuler-chaînes-caracteres-python). Il s'agit d'une méthode applicable à des chaînes de caractères, nous allons donc devoir l'appliquer à notre chaîne de caractère `text` dans le programme `html-to-list1.py`. Modifier `html-to-list1.py` pour y ajouter le tag `lower()` à la fin de la chaîne `text`.

``` python
#html-to-list1.py
import urllib.request, urllib.error, urllib.parse, obo

url = 'http://www.oldbaileyonline.org/browse.jsp?id=t17800628-33&div=t17800628-33'

response = urllib.request.urlopen(url)
html = str(response.read().decode('UTF-8'))
text = obo.stripTags(html).lower() #ajouter la méthode applicable à une chaîne de caractères ici.
wordlist = text.split()

print(wordlist)
```

Normalement, vous devriez obtenir la même liste de mots que précédemment, mais cette fois avec tous les caractères en minuscules.

En effet, la fonction stripTags() du module obo retourne une chaîne de caractère à partir des données extraites (contenues dans la variable 'html'), que nous convertissons en minuscules avec la fonction lower(). En appliquant ainsi les deux fonctions sur une même ligne, nous gardons un code assez court tout en apportant des modifications majeures à notre programme.

Comme nous l'avons déjà vu, Python permet de faire beaucoup, facilement et avec peu de code !

À partir de là, nous pourrions parcourir un grand nombre d'autres entrées de &laquo;&nbsp;Old Bailey Online&nbsp;&raquo; et de nouvelles sources pour être surs qu'il n'y ait pas d'autres caractères spéciaux qui pourraient nous poser problème plus tard. Nous pourrions également anticiper toutes les situations où nous voudrions conserver la ponctuation (par exemple, pour distinguer des quantités monétaires comme &laquo;&nbsp;1300$&nbsp;&raquo; ou &laquo;&nbsp;1865£&nbsp;&raquo; des dates, ou reconnaître la différence entre &laquo;&nbsp;1629-40&nbsp;&raquo; et &laquo;&nbsp;1629 40&nbsp;&raquo;). C'est le travail des programmeurs professionnels : essayer de penser à tout ce qui pourrait clocher et traiter le problème en amont.

Nous allons utiliser une autre approche. Notre objectif est de développer des techniques utilisables par un historien en activité durant le processus de recherche. Cela signifie que nous favoriserons presque toujours des solutions approximativement correctes mais pouvant être développées rapidement. Alors plutôt que de prendre du temps tout de suite pour créer un programme solide face à l'exceptionnel, nous allons simplement nous débarrasser de tout ce qui n'est pas une lettre, accentuée ou non, ou un chiffre arabe. La programmation est par essence un processus &laquo;&nbsp;d'affinement pas à pas&nbsp;&raquo;. On commence avec un problème et le début d'une solution, puis on affine cette solution jusqu'à obtenir quelque chose qui fonctionne au mieux.

## Expressions Régulières en Python

Nous avons retiré les majuscules, il ne reste plus qu'à éliminer toute la ponctuation. Si on la laisse dans le texte, la ponctuation va perturber nos mesures de fréquences. En effet, nous voulons bien évidemment considérer &laquo;&nbsp;evening?&nbsp;&raquo; comme &laquo;&nbsp;evening&nbsp;&raquo; et &laquo;&nbsp;1780.&nbsp;&raquo; comme &laquo;&nbsp;1780&nbsp;&raquo;.

Il est possible d'utiliser la méthode `.replace()` sur la chaîne de caractères pour en retirer tous les types de ponctuation :

``` python
text = text.replace('[', '')
text = text.replace(']', '')
text = text.replace(',', '')
#etc...
```

Cependant, ce n'est pas optimal. Pour continuer à créer un programme court et puissant, nous allons utiliser ce qu'on appelle des *expressions* *régulières*. Les expressions régulières sont disponibles dans de nombreux langages de programmation, sous différentes formes.

Les expressions régulières permettent de rechercher des patrons lexicaux (patterns) bien définis et peuvent raccourcir drastiquement votre code. Par exemple, mettons que vous vouliez trouver une lettre de l'alphabet dans une chaîne de caractères. Plutôt que de créer une boucle `if`/`else` qui vérifie si chaque caractère de la chaîne correspond à &laquo;&nbsp;a&nbsp;&raquo;, puis à &laquo;&nbsp;b&nbsp;&raquo;, puis à &laquo;&nbsp;c&nbsp;&raquo;, etc., vous pouvez vous servir d'une expression régulière pour voir si le caractère de la chaîne est une lettre entre &laquo;&nbsp;a&nbsp;&raquo; et &laquo;&nbsp;z&nbsp;&raquo;. Vous pourriez aussi vous en servir pour chercher un chiffre, une lettre majuscule, un caractère alphanumérique, un retour chariot, ou encore une combinaison de ces différents éléments, et bien plus.

Dans Python, les expressions régulières sont disponibles dans un module. Ce dernier n'est pas chargé automatiquement, car il n'est pas nécessaire pour tous les programmes et le charger à chaque fois prendrait du temps inutilement. Il va donc falloir l'importer (`import` le module nommé `re`), comme vous aviez importé le module `obo.py`.

Comme nous ne nous intérressons qu'aux caractères alphanumériques, nous allons créer une expression régulière qui isole uniquement ces éléments, et retire tout le reste. Copiez la fonction ci-dessous et collez-la à la fin du module `obo.py`. Vous pouvez laisser les autres fonctions du module tranquilles, nous allons continuer à les utiliser.

``` python
# Prend une chaîne de caractère text, la segmente en liste avec pour délimiteurs
# les caractères non-alphanumériques (en utilisant la définition
# Unicode des alphanumériques) ou suite de caractères non-alphanumériques
# qui sont ainsi supprimés 

def stripNonAlphaNum(text):
    import re
    return re.compile(r'\W+', re.UNICODE).split(text)
```

L'expression régulière dans ce code est le contenu de la chaîne de caractères, autrement dit `\W+`. `W` est le diminutif utilisé pour la classe des caractères non-alphanumériques. Dans une expression régulière Python, le signe plus (+) correspond à une ou plusieurs occurrences d'un caractère donné. `re.UNICODE` informe l'interpréteur que nous voulons inclure les caractères des autres langues du monde dans notre définition &laquo;&nbsp;d'alphanumériques&nbsp;&raquo;, tout comme les lettres de A à Z, de a à z et les chiffres de 0 à 9 de l'anglais. Les expressions régulières doivent être *compilées* avant de pouvoir être utilisées. C'est ce que fait la dernière ligne de la fonction présentée plus haut. Inutile de vous embêter à comprendre la compilation pour le moment.

Après avoir peaufiné notre programme `html-to-list1.py`, il doit ressembler à cela :

``` python
#html-to-list1.py
import urllib.request, urllib.error, urllib.parse, obo

url = 'http://www.oldbaileyonline.org/browse.jsp?id=t17800628-33&div=t17800628-33'

response = urllib.request.urlopen(url)
html = response.read().decode('UTF-8')
text = obo.stripTags(html).lower()
wordlist = obo.stripNonAlphaNum(text)

print(wordlist)
```

En exécutant le programme et en regardant ce qu'il en ressort dans le panneau `Command Output`, vous verrez qu'il fait plutôt du bon travail. Ce code sépare les mots composés avec un trait d'union comme &laquo;&nbsp;coach-wheels&nbsp;&raquo; en deux mots, et compte le possessif anglais &laquo;&nbsp;'s&nbsp;&raquo; ou la forme &laquo;&nbsp;o'clock&nbsp;&raquo; comme des mots distincts, en retirant l'apostrophe. Il s'agit cependant d'une approximation  satisfaisante de ce que nous voulions obtenir, et nous pouvons continuer d'avancer vers nos mesures de fréquences avant d'essayer de l'améliorer. (Si les sources sur lesquelles vous travaillez sont dans plus d'une langue, vous aurez besoin d'en apprendre plus sur le standard [Unicode](https://home.unicode.org/) et sur sa [prise en charge Python](https://web.archive.org/web/20180502053841/http://www.diveintopython.net/xml_processing/unicode.html).)

## Pour aller plus loin

Si vous souhaitez pratiquer d'avantage les expressions régulières, le chapitre 7 de [Dive into Python](https://web.archive.org/web/20180416143856/http://www.diveintopython.net/regular_expressions/index.html) de Mark Pilgrim peut être un bon entrainement.

### Synchronisation des codes

Pour pouvoir continuer vers les leçons suivantes, il est important d'avoir les bons dossiers et les bons programmes dans votre répertoire programming-historian. A la fin de chaque chapitre de cette série de leçons, vous pouvez télécharger le fichier zip programming-historian pour être sur.e d'avoir le bon code.

- [`python-lessons4.zip`](https://github.com/programminghistorian/jekyll/blob/gh-pages/assets/python-lessons4.zip)
