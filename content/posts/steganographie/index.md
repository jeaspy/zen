---
title: "Stéganographie"
date: 2021-02-21T22:38:02+01:00
description: "Pas à pas avec NumPy. La stéganographie ou comment cacher un texte dans une image matricielle."
#categories: [NumPy, Python]
head_pict: false
weight: 1
dropCap: false
displayInMenu: true
displayInList: true
draft: false
resources:
- name: featuredImage
  src: "nature_1280f.jpg"
---
# Stéganographie
<br>

 {{<smallimg src="numpy.png" alt="Logo Numpy" smartfloat="left" width="100px">}}
 <span class="lettrine">C’</span>est à l’occasion d’un [MOOC](https://fr.wikipedia.org/wiki/Massive_Open_Online_Course "Wikipédia : Massive Open Online Course") sur [Python](https://fr.wikipedia.org/wiki/Python_(langage) "Wikipédia : Python (langage)"), proposé par [France Université Numérique](https://www.fun-mooc.fr/ "www.fun-mooc.fr"), que j’ai découvert [NumPy](https://fr.wikipedia.org/wiki/NumPy "Wikipédia : NumPy"). Cette bibliothèque permet de manipuler des matrices ou tableaux multidimensionnels de manière particulièrement rapide et efficace.

À titre d’exercice, j’ai voulu écrire un petit programme de [stéganographie](https://fr.wikipedia.org/wiki/St%C3%A9ganographie "Wikipédia : Stéganographie") qui permet d’encoder des données et de les insérer dans des [images matricielles](https://fr.wikipedia.org/wiki/Image_matricielle "Wikipédia : Image matricielle").

Par souci de simplicité, le programme ne prend en charge que certains types d’images matricielles (bitmap) compressibles sans perte de données (PNG, TIFF, BPM...). L’information ajoutée à l’image sera de type textuel, bien qu’il soit théoriquement possible d’encoder images, sons, vidéos, tout ce qui est numérisable.

#### Prérequis

- [Python 3.x][1] (Sur Linux, une version est en principe déjà installée)
- [NumPy][2]
- [PILLOW][3]
  - PILLOW est un *fork* de [PIL (Python Imaging Library)](https://en.wikipedia.org/wiki/Python_Imaging_Library "Wikipédia : Python Imaging Library") qui sera appelé par l’instruction `import PIL` dans le module principal.

[1]: https://www.python.org/ "python.org"
[2]: https://numpy.org/ "numpy.org"
[3]: https://pypi.org/project/Pillow/ "pypi.org"

Il existe plusieurs façons de procéder pour installer PIL et NumPy. Personnellement, j’utilise [pip](https://pip.pypa.io/en/stable/ "pip.pypa.io"), le gestionnaire de paquets standard.

Les modules [_imghdr_][4] et [_math_][5] seront aussi utilisés mais comme ils font partie de la [bibliothèque standard](https://docs.python.org/3/library/ "docs.python.org"), ils sont généralement installés avec Python.

[4]: https://docs.python.org/fr/3/library/imghdr.html#module-imghdr "docs.python.org"
[5]: https://docs.python.org/fr/3/library/math.html#module-math "docs.python.org"

#### Traitement de l’image — Stockage des valeurs des pixels dans un tableau NumPy

On commence par importer les modules dont on a besoin&nbsp;:

```python
# coding: utf8
import imghdr
from math import ceil
from PIL import Image
import numpy as np
```

On définit la liste des formats d’images qui seront acceptés&nbsp;:

```python
formats = ['png', 'bmp', 'tiff', 'ppm', 'pgm', 'pnm', 'pcx', 'sgi', 'tga']
```
<details>
<summary>  
		Précisions sur ces formats
</summary>  

  - **PNG** (Portable Network Graphics) est un format ouvert d’images numériques, qui a été créé pour remplacer le format GIF, à l’époque propriétaire et dont la compression était soumise à un brevet. Le PNG est un format sans perte spécialement adapté pour publier des images simples comprenant des aplats de couleurs.
  - **BMP** (Windows bitmap, connu aussi sous l’abréviation de BitMaP; en anglais, device-independent bitmap ou DIB), est un format d’image matricielle ouvert développé par Microsoft et IBM.
  - **TIFF** (Tag Image File Format) est un format de fichier pour image numérique. Adobe en est le dépositaire et le propriétaire initial (via Aldus). Plus exactement, il s’agit d’un format de conteneur (ou encapsulation), à la manière de avi ou zip, c’est-à-dire pouvant contenir des données de formats arbitraires.
  - **PPM** (Portable PixMap) et le **PGM** (Portable GrayMap) qui, avec le PBM (Portable BitMap), sont désignés comme le format **PNM** (Portable aNyMap) sont des formats de fichier graphique utilisés pour les échanges. Ils ont été définis et sont utilisés par le projet NetPBM. Ils proposent des fonctionnalités élémentaires et sont utilisés pour convertir les fichiers de type *pixmap*, *graymap* et *bitmap* entre différentes plateformes.
  - **PCX** (PiCture eXchange) est un format d’image numérique dont l’encodage est basé sur une forme de *run-length encoding*. PCX a été développé par la société ZSoft Corporation à Marietta, en Géorgie. C’était le format de base de leur logiciel PC Paintbrush, un des logiciels d’édition d’images les plus populaires sous DOS à l’époque.
  - **SGI** (Silicon Graphics Image) ou RGB file est le format de fichier graphique matriciel pour les stations de travail de Silicon Graphics. Il peut stocker de 8 à 32 bits par pixel.
  - **TGA** (Truevision (TARGA) Raster Graphics) est un format de fichier graphique matriciel créé par Truevision au milieu des années 1980.  
  [Sources : [Wikipedia](https://fr.wikipedia.org/), [MDN Web Docs](https://developer.mozilla.org/) et [FileSamples](https://filesamples.com/)]  
  *Vous trouverez sur FileSamples des [échantillons à télécharger](https://filesamples.com/categories/image) de ces différents formats et bien d’autres.*  
</details>  

{{< figure class="small" src="sample_1280×853.png" link="sample_1280×853.png" download="test" alt="Image exemple" title="sample_1280×853.png">}}

L’image ci-contre peut être téléchargée à partir de <a href="sample_1280×853.png" title="Lien de téléchargement" download>ce lien</a> ou depuis la page se trouvant [à cette adresse](https://filesamples.com/formats/png "filesamples.com"). Elle servira d’exemple dans la suite de l’exercice, mais n’importe quelle image peut faire l’affaire pourvu qu’elle soit dans l’un des formats indiqués plus haut.  
<hr style="border:0;">

Il faut d’abord vérifier que le format du fichier est bien l’un de ceux définis dans la liste, lever une exception dans le cas contraire, et enfin récupérer le code de l’ensemble des pixels pour le copier dans un tableau NumPy&nbsp;:

```python
pict = "sample_1280×853.png" # adresse locale de l’image
with Image.open(pict) as im:
     if (imghdr.what(pict) or im.format.lower()) not in formats:
        raise TypeError("Ce format n’est pas valable")
     else:
        im_array = np.array(im)
```

Le code a bien été enregistré, comme on peut le constater en imprimant quelques valeurs du tableau `im_array`&nbsp;:

```python
# Valeurs des octets des 9 premiers pixels
print(im_array[:3, :3, :])
```
<pre class="sortie">
[[[178 132 145]
  [177 131 144]
  [179 133 146]]

 [[170 124 137]
  [172 126 139]
  [181 135 148]]

 [[182 134 148]
  [176 128 142]
  [177 131 144]]]
</pre>

Chaque triplet correspond à un [pixel](https://fr.wikipedia.org/wiki/Pixel "Wikipédia : Pixel") de l’image, selon le [codage trichrome RVB](https://fr.wikipedia.org/wiki/Rouge_vert_bleu "Wikipédia : Rouge vert bleu") (ou RGB en anglais). Chacun des nombres qui représente un [octet](https://fr.wikipedia.org/wiki/Octet "Wikipédia : Octet") est compris entre 0 et 255.  
Le tableau présente les valeurs des octets pour chaque pixel, de gauche à droite et de bas en haut.  

Ainsi le 1er pixel, en haut à gauche, est représenté sur le tableau par la ligne `178 132 145` qui désigne la couleur [rgb(178,132,145)](https://color-hex.org/color/b28491 "color-hex.org")<span class="carre" style="background:rgb(178,132,145)"></span>, soit `#B28491` en [notation hexadécimale](https://fr.wikipedia.org/wiki/Syst%C3%A8me_hexad%C3%A9cimal "Wikipédia : Système hexadécimal").

Dans notre exemple, nous voyons que chaque pixel est représenté par trois valeurs de 0 à 255. Certaines images PNG possèdent en outre une quatrième valeur pour coder le [canal alpha](https://fr.wikipedia.org/wiki/Alpha_blending "Wikipédia : Alpha blending"). Ce dernier permet de préciser la transparence, qui peut varier d’invisible (valeur 0) à totalement opaque (valeur 255).

En revanche, d’autres images, en noir et blanc, n’ont qu’une seule valeur, comprise entre 0 et 255, pour coder un pixel. Dans ce cas, chaque chiffre correspondant à un niveau de gris.

L’injection du code pourra se faire dans tous les octets, peu importe qu’il y en ait 1, 3 ou 4 par pixel.

**Pourquoi 255 et pas 999 (par exemple)&nbsp;?**  

Tout simplement parce que 255 = 2<sup>8</sup>&nbsp;-&nbsp;1, ce qui en [notation binaire](https://fr.wikipedia.org/wiki/Syst%C3%A8me_binaire "Wikipédia : Système binaire") est représenté par `11111111`, soit un octet composé de huit «&nbsp;1&nbsp;».

Chacun des trois octets qui code un pixel, dans l’exemple que nous avons choisi, est composé de 8 [bits (ou unités d’information)](https://fr.wikipedia.org/wiki/Bit "Wikipédia : Bit"), 1 bit pouvant avoir pour valeur 0 ou 1.

Le premier pixel représenté par le triplet `178 132 145` peut aussi être représenté en binaire par les valeurs `10110010`, `10000100` et `10010001`.

Python permet de passer facilement d’une base à l’autre&nbsp;:

```python
# Représentation d’entiers en binaires
for n in [178, 132, 145]:
    print(f'{n:b}', end=' ') # bin(n) est aussi possible
```

<pre class="sortie">
10110010 10000100 10010001
</pre>

C’est avec l’ensemble des valeurs du tableau `im_array` que nous allons travailler pour coder notre image.
Examinons d’abord les caractéristiques de ce tableau&nbsp;:

```python
print('Dimensions :', im_array.ndim)
print('Format :', im_array.shape)
print('Taille :', im_array.size)  # nombre total d’éléments
print('Data Type :', im_array.dtype)
```

<pre class="sortie">
Dimensions : 3
Format : (853, 1280, 3)
Taille : 3275520
Data Type : uint8
</pre>

Les trois valeurs du format correspondent respectivement à la hauteur de l’image (853 px), à sa largeur (1280 px) et au nombre d’octets (3) qui codent chaque pixel.
La taille est le nombre total d’octets, soit 853&nbsp;\*&nbsp;1280&nbsp;\*&nbsp;3.
Enfin, le type «&nbsp;uint8&nbsp;» (*unsigned integer 8bits*) permet comme son nom l’indique de stocker des valeurs entières non-signées codées sur 8 bits (soit 1 octet).

Ce dernier point est important puisque chaque nombre entier du tableau étant représenté par 1 octet, tout dépassement de la valeur 255 (en binaire `11111111`) ramènera la valeur à 0 (zéro).

Vérifions cela en Python&nbsp;:

```python
ar = np.array([255, 255, 255]) #ffffff en hexa (couleur blanche)
# On ajoute 1 à chaque valeur du tableau
ar_plus = (1 + ar).astype(np.uint8)
print(ar_plus)
```

<pre class="sortie">
[0 0 0]
</pre>


En ajoutant simplement 1 aux valeurs du tableau, chacune d’elles est passée de 255 à 0.

Si `x1` est l’ancienne valeur de l’octet et `n` un entier que l’on souhaite ajouter à `x1`, alors la nouvelle valeur obtenue sera `x2 = (x1 + n) % 256`.

En effet, 255 est la limite supérieure que peut atteindre 1 octet dont tous les bits sont à «&nbsp;1&nbsp;» (`11111111` en binaire).

Or, passer de 255 à 0 signifie, pour la couleur, passer de la teinte la plus claire à la plus foncée.

Lorsque l’on modifiera légèrement les chiffres qui codent l’image pour y insérer le message, il faudra éviter que la valeur obtenue pour chaque octet ne dépasse la valeur de 255, sans quoi des artefacts dus à des changements importants de teinte apparaîtront sur l’image.

Par exemple, la différence entre les couleurs représentées par [rgb(100, 100, 240)](https://color-hex.org/color/6464f0 "color-hex.org")&nbsp;<span class="carre" style="background:rgb(100,100,240)"></span> et celle obtenue en augmentant de 10 la valeur du canal bleu, [rgb(100, 100, 250)](https://color-hex.org/color/6464fa "color-hex.org")&nbsp;<span class="carre" style="background:rgb(100,100,250)"></span>, est à peine perceptible. En revanche, si on ajoute encore 10, comme la valeur dépasse 255, elle est réinitialisée et passe de 250 à 4 (`(250 + 10) % 256` = 4). La couleur obtenue, [rgb(100, 100, 4)](https://color-hex.org/color/646404 "color-hex.org")&nbsp;<span class="carre" style="background:rgb(100,100,4)"></span> est alors très différente de celle d’origine.

#### Encodage du texte  

Après avoir stocké les valeurs de tous les pixels de l’image dans un tableau NumPy, nous allons récupérer, coder et stocker dans une autre variable le texte que nous souhaitons insérer dans le code de l’image.

À titre d’illustration, nous choisirons le [*Discours de la Méthode*](https://fr.wikipedia.org/wiki/Discours_de_la_m%C3%A9thode "Wikipédia : Discours de la méthode") de [Descartes](https://fr.wikipedia.org/wiki/Ren%C3%A9_Descartes "Wikipédia : René Descartes"), que nous trouvons en accès libre sur le site du [projet Gutenberg](https://fr.wikipedia.org/wiki/Projet_Gutenberg "Wikipédia : Projet Gutenberg") à [cette adresse](http://www.gutenberg.org/files/13846/13846-0.txt "www.gutenberg.org") au format Plain Text UTF-8.

```python
# Adresse locale du texte
with open ('13846-0.txt', 'r', encoding='utf8') as f:
    message = f.read()
```

On peut vérifier que le texte a bien été enregistré sous forme de chaîne de caractères dans la variable `message`, en imprimant quelques caractères, et en vérifiant la longueur de la chaîne&nbsp;:

```python
print('Début du texte :', message[:73])
print(f'Longueur du texte : {len(message)} caractères')
```
<pre class="sortie">
Début du texte : ﻿The Project Gutenberg EBook of Discours de la méthode, by René Descartes
Longueur du texte : 700602 caractères
</pre>

Bien entendu, il va falloir transformer la chaîne de caractères afin de pouvoir l’insérer dans le tableau de pixels.

La première opération est de transformer la chaîne en nombres, chaque nombre représentant 1 octet.

**Unicode et UTF-8**  

Associer des chiffres à des lettres (et à des signes, en général) ne pose aucune difficulté, puisque c’est le principe même de l’[Unicode](https://fr.wikipedia.org/wiki/Unicode "Wikipédia : Unicode") qui affecte à chaque signe ou caractère un unique identifiant numérique.

En Python, on obtient le [point de code](https://fr.wikipedia.org/wiki/Point_de_code "Wikipédia : Point de code") d’un caractère au moyen de la fonction `ord()` et, à l’inverse le caractère correspondant à un identifiant numérique grâce à la fonction `chr()`.

Quelques exemples&nbsp;:

```python
for lettre in 'Python':
    print(f'{lettre}:{ord(lettre)}', end=' ')
```
<pre class="sortie">
P:80 y:121 t:116 h:104 o:111 n:110
</pre>

```python
for car in 'éàïûç':
    print(ord(car), end= ' ')
```
<pre class="sortie">
233 224 239 251 231
</pre>

```python
print(ord('@'))
print(ord('😀'))
print(ord('あ')) # caractère japonais (hiragana) se prononçant [a]
```
<pre class="sortie">
64
128512
12354
</pre>

Dans la pratique, on trouve souvent le point de code exprimé sous forme hexadécimale (base 16). Par exemple le caractère «&nbsp;@&nbsp;» a pour point de code 64 en base 10, et c’est cette valeur que renvoie la fonction `ord()`. Mais 64, exprimé en base 16 s’écrit `40` (on peut le vérifier en exécutant dans Python `hex(64)`). Son point de code est donc souvent représenté par `U+0040`, mais il s’agit bien de la même valeur.

Unicode est constitué d’un répertoire de 137&#8239;929 caractères, permettant de coder toutes les langues du monde, et bien au-delà&nbsp;: caractères spéciaux, symboles, émoticônes, etc.  

Cependant, convertir les caractères en points de code selon le standard Unicode ne permet pas de délimiter les blocs correspondant aux différents caractères.

Par exemple, la suite de chiffres `12354` peut aussi bien correspondre à un identifiant unique, `chr(12354)`, qui renvoie le caractère japonais «&nbsp;あ&nbsp;») qu’à deux identifiants accolés `123` et `54` qui correspondent alors à la chaîne de caractères «&nbsp;{6&nbsp;» puisque `chr(123)` et `chr(54)` renvoient respectivement «&nbsp;{&nbsp;» et «&nbsp;6&nbsp;».

Pour résoudre ces ambiguïtés, il est commode d’utiliser un système d’encodage universellement reconnu, l’[UTF-8 (*Universal Character Set Transformation Format - 8 bits*)](https://fr.wikipedia.org/wiki/UTF-8 "Wikipédia : UTF-8").

L’encodage UTF-8 est un format qui présente deux particularités&nbsp;:
- Il représente les points de code sous forme d’octets, chaque caractère pouvant être représenté par un ou plusieurs octets.  

- Il adopte une convention qui permet de reconnaître d’après les [bits de poids fort](https://fr.wikipedia.org/wiki/Bit_de_poids_fort "Wikipédia : Bit de poids fort") de l’octet (ie. les bits situés sur la gauche de l’octet) si l’octet en question représente un seul ou plusieurs caractères, et dans le cas d’un caractère codé sur plusieurs octets, s’il est ou non l’octet de tête.

Exemple d’un caractère codé sur un octet&nbsp;:

```python
ord('a') # on cherche l’identifiant Unicode de 'a'
```
<pre class="sortie">
97
</pre>

```python
f'{97:08b}' # on convertit cet identifiant en binaire
            # et l’on ajoute des 0 à gauche pour arriver à
            # un total de 8 bits (= 1 octet)
```
<pre class="sortie">
'01100001'
</pre>  

Par convention, si le premier bit est un «&nbsp;0&nbsp;», on sait que le caractère est codé sur un seul octet. C’est le cas de tous les caractères définis par la [norme ASCII](https://fr.wikipedia.org/wiki/American_Standard_Code_for_Information_Interchange "Wikipédia : American Standard Code for Information Interchange").

Cas d’un caractère codé sur 2 octets (c’est généralement le cas des caractères accentués)&nbsp;:

```python
ord('é') # on cherche l’identifiant Unicode de 'é'
```  
<pre class="sortie">
233
</pre>

```python
f'{233:b}' # valeur binaire de 233
```  
<pre class="sortie">
'11101001'
</pre>

Théoriquement, il est possible de coder «&nbsp;é&nbsp;» sur 1 seul octet puisque son point de code est inférieur à 256.

Mais par convention, seuls 7 bits sont utilisés pour les caractères codés sur un octet. Ceux dont le point de code est supérieur à 127 sont codés sur deux à quatre octets.

Ainsi «&nbsp;é&nbsp;» est codé sur deux octets. Le premier commence par `110` et les deux «&nbsp;1&nbsp;» indiquent que deux octets seront utilisés pour ce caractère.

Le deuxième octet commence par `10`, ce qui est aussi une convention indiquant qu’il ne s’agit pas de l’octet de tête du caractère.

En fonction de ces conventions, le codage de `11101001` se trouve réparti sur deux octets&nbsp;: <code>**110**00011</code> et <code>**10**101001</code>.  

Les bits en gras renseignent sur le nombre d’octets et sur leur position. Le code du caractère proprement dit se trouve réparti sur les autres bits, avec éventuellement un remplissage de zéros à gauche.

Si l’on regroupe ces bits, on retrouve `11101001` qui la représentation binaire du point de code du caractère «&nbsp;é&nbsp;», soit 233 en base décimale.

Comme dernier exemple, nous prendrons le [caractère «&nbsp;和&nbsp;»](https://en.wiktionary.org/wiki/%E5%92%8C "en.wiktionary.org") qui est codé sur 3 octets comme la plupart des [caractères CJK](https://en.wikipedia.org/wiki/CJK_characters "Wikipédia : CJK characters"))

```python
ord('和') # on cherche l’identifiant Unicode du caractère '和'　
```

<pre class='sortie'>
21644
</pre>


```python
f'{21_644:b}' # valeur binaire de 21_644
```

<pre class='sortie'>
'101010010001100'
</pre>

Puisque par convention, l’octet de tête indique le nombre d’octets utilisés pour coder le caractère , on trouvera trois «&nbsp;1&nbsp;» suivi d’un «&nbsp;0&nbsp;».
Le premier octet est donc de la forme <code>**1110**xxxx</code>.  

Les second et troisième octets commencent par <code>**10**</code> puisque ce ne sont pas les octets de tête.

Les trois octets sont donc de la forme <code>**1110**xxxx **10**xxxxxx **10**xxxxxx</code>.

Il reste à remplir les «&nbsp;x&nbsp;» avec les bits correspondant au point de code du caractère en ajoutant un ou des «&nbsp;0&nbsp;» à gauche&nbsp;: <code>**1110**0101 **10**010010 **10**001100</code>

L’identifiant Unicode se trouve ainsi réparti à droite de chaque octet, exprimé sous forme binaire. Une fois ces 3 parties rassemblées, on obtient `10101001000110`, soit 21&#8239;644 en base 10&nbsp;:

```python
int('101010010001100', 2) # conversion de base 2 en base 10
```

<pre class='sortie'>
21644
</pre>

Pour d’autres précisions sur le nombre d’octets utilisés dans ce format de codage, on se référera à [ce tableau](https://fr.wikipedia.org/wiki/UTF-8#Description "Wikipédia : UTF-8") ou à [cette présentation](http://www.ac-grenoble.fr/loubet.valence/userfiles/file/Disciplines/Sciences/SPC/TS/R1/co/types_1.html "www.ac-grenoble.fr") très synthétique.

<p style="text-align:center;">&#8258;</p>

Il serait fastidieux d’encoder tout le texte de cette manière, caractère par caractère, avec une double opération à chaque fois puisqu’il faut chercher le *code point* et le convertir selon le standard UTF-8.

Heureusement, il existe en Python la fonction `bytes()` qui prend en charge les longues chaînes de caractères et réalise les deux opérations en même temps. Les chaînes sont directement converties en octets selon l’encodage demandé.

```python
# Conversion en octets d’une chaîne codée en UTF-8
bytes("Python", 'utf8')
```

<pre class="sortie">
b'Python'
</pre>

Cette représentation pourrait laisser penser que `bytes()` renvoie simplement une chaîne de caractères, mais il n’en est rien. Les octets codant les caractères ASCII sont représentés par les caractères eux-mêmes, mais une itération sur l’objet renvoyé par `bytes()` montre qu’il s’agit bien d’octets&nbsp;:

```python
for n in bytes('Python', 'utf8'):
    print(n, end=' ')
```

<pre class='sortie'>
80 121 116 104 111 110
</pre>

Ce sont bien les points de code de chaque caractère. On remarque que, dans cet exemple, ils sont tous codés sur un seul octet.  

Dans le cas de caractères codés sur plusieurs octets, `bytes()` renvoie bien la valeur des octets selon la norme UTF-8.

```python
bytes("é", 'utf8')
```

<pre class="sortie">
b'\xc3\xa9'
</pre>

`\x` est un séparateur, mais `c3` et `a9` représentent, sous forme hexadécimale, les deux octets qui transcrivent le caractère «&nbsp;é&nbsp;» selon la norme UTF-8. Nous pouvons les transcrire en binaire&nbsp;:

```python
# Conversion de base 16 en base 2
print(f'{int("c3", 16):b}', f'{int("a9", 16):b}')
```

<pre class="sortie">
11000011 10101001
</pre>

On retrouve bien les deux octets, dont les bits représentant le point de code sont `11` pour le premier et `101001` pour le second. Mis bout-à-bout, on obtient `11101001` qui est la représentation binaire de 233, point de code du caractère «&nbsp;é&nbsp;»&nbsp;:

```python
chr(233)
```
<pre class='sortie'>
'é'
</pre>

Pour un caractère codé sur 3 octets, comme dans l’exemple précédent (和), la fonction `bytes()` rend également compte de la représentation en octets selon la norme UTF-8&nbsp;:

```python
bytes('和', 'utf8')
```

<pre class="sortie">
b'\xe5\x92\x8c'
</pre>

```python
# Conversion de base 16 en base 2
for n in ('e5', '92', '8c'):
    print(f'{int(n, 16):b}', end=' ')
```
<pre class="sortie">
11100101 10010010 10001100
</pre>

Les bits représentant le point de code, ici répartis sur 3 octets, une fois regroupés, donnent `101010010001100`, soit 21&#8239;644 en base 10&nbsp;:

```python
int('101010010001100', 2) # conversion en base 10
```  

<pre class="sortie">
21644
</pre>

21&#8239;644 est bien le point de code du caractère «&nbsp;和&nbsp;»&nbsp;:

```python
chr(21_644)
```
<pre class="sortie">
'和'
</pre>

La fonction `bytes()` est donc extrêmement puissante puisqu’elle va nous permettre, d’un seul coup, de transformer la longue chaîne de caractères enregistrée dans la variable `message` en une chaîne de *bytes*. Il suffira ensuite de les représenter sous forme binaire pour pouvoir insérer le code bit par bit dans l’image.

```python
# Conversion d’une chaîne de caractères en octets
octets = bytes(message, 'utf8')
```  

On peut vérifier que les éléments enregistrés sont bien des octets&nbsp;:

```python
print(type(octets))
# Imprimer quelques éléments de la chaîne
print([n for n in octets[:9]])
```

<pre class="sortie">
<&#8203;class 'bytes'&#8203;>
[239, 187, 191, 84, 104, 101, 32, 80, 114]
</pre>

On reconnaît, au début, les 3 octets `239 187 191` (en hexadécimal `EF BB BF`) caractéristiques du BOM (Byte Order Mark) ou [indicateur d’ordre des octets](https://fr.wikipedia.org/wiki/Indicateur_d%27ordre_des_octets "Wikipédia : Indicateur d’ordre des octets"). Le texte proprement dit ne commence qu’au 4ème octet (`84 104 101`...). Le BOM, utilisé par certains logiciels, n’est d’aucune utilité ici mais comme il s’agit d’une espace insécable sans chasse, il n’est pas gênant de le garder.

Nous pouvons ensuite transformer cette chaîne en un tableau d’entiers (base décimale)&nbsp;:

```python
# Conversion des octets en entiers (base 10)
octets_dec = np.frombuffer(octets, dtype=np.uint8)
print(octets_dec.dtype)
print(octets_dec)
```
<pre class="sortie">
uint8
[239 187 191 ...  42  10  10]
</pre>

Il s’agit cette fois d’un tableau d’entiers et non plus d’octets.

Nous pouvons maintenant, avec la fonction `np.unpackbits()` convertir ce tableau d’entiers en binaires et les séparer en éléments d’un bit chacun. Il en résulte un tableau qui ne contient que des «&nbsp;0&nbsp;»&nbsp; et des «&nbsp;1&nbsp;»&nbsp;&nbsp;:

```python
# Conversion en binaire et répartition en éléments d’un bit
octets_bin = np.unpackbits(octets_dec)
# Imprimer les 16 premiers éléments du tableau
print(octets_bin[:16])
```

<pre class="sortie">
[1 1 1 0 1 1 1 1 1 0 1 1 1 0 1 1]
</pre>

Ayant tranformé la chaîne de caractères d’origine en un tableau dont les éléments constituent la plus petite unité d’information possible («&nbsp;1&nbsp;»&nbsp; ou «&nbsp;0&nbsp;»&nbsp;), nous pouvons considérer le codage du texte comme achevé.

<p style="text-align:center;">&#8258;</p>  

#### Comment insérer le code du texte dans le tableau de pixels&nbsp;?

À ce stade, nous avons un tableau de chiffres compris entre 0 et 255 codant chacun un octet, l’ensemble de ces octets représentant la valeur des pixels de l’image.

Les deux premiers pixels sont représentés par 2&nbsp;*&nbsp;3 octets, soit 6 octets&nbsp;:

```python
# Afficher « à plat » les octets codant les 2 premiers pixels
print(im_array[0, :2, :].ravel())
```
<pre class="sortie">
[178 132 145 177 131 144]
</pre>

Rappelons que l’octet est composé de 8 bits, que nous pouvons visualiser en convertissant les chiffres en base 2&nbsp;:

```python
for octet in [178, 132, 145, 177, 131, 144]:
    print(f'{octet:b}', end=' ')
```

<pre class="sortie">
10110010 10000100 10010001 10110001 10000011 10010000
</pre>

Le code sera inséré à droite de chaque octet. En effet, c’est à droite que se trouvent les bits dits [de poids faible](https://fr.wikipedia.org/wiki/Bit_de_poids_faible "Wikipédia : Bit de poids faible"). Si on les modifie, les couleurs de l’image ne devraient pas être très altérées. Si l’on change la valeur d’un ou deux bits à droite, il sera même impossible de discerner la différence à l’œil nu.

Avant d’insérer le code, il faut d’abord mettre à zéros les bits de poids faible qui seront utilisés&nbsp;: <code>1011001**0**</code><code>1000010**0**</code><code>1001000**0**</code><code>1011000**0**</code><code>1000001**0**</code><code>1001000**0**</code>

Ceux qui étaient déjà à zéro restent évidemment à zéro, mais ceux qui étaient à «&nbsp;1&nbsp;»&nbsp; passent à zéro.

Cette opération a un double avantage&nbsp;:

- Elle permet d’éviter, si une valeur se trouve à 255, que l’ajout d’un «&nbsp;1&nbsp;»&nbsp; ne réinitialise l’octet selon la formule que nous avons vue plus haut&nbsp;: `x2 = (x1 + n) % 256`.

- Cela permet aussi de décoder plus facilement le message. Si l’on sait que le bit de poids faible était à zéro avant l’insertion du code, il suffira de voir comment il a été modifié pour reconstituer le code.

Dans notre exemple portant sur les deux premiers pixels de l’image, une fois les bits de poids faible mis à zéro, nous pouvons les remplacer par le code du texte que nous avons plus haut converti en une suite de «&nbsp;0&nbsp;»&nbsp; et de «&nbsp;1&nbsp;»&nbsp;.

Les 6 premières valeurs de ce code sont&nbsp;:


```python
print(octets_bin[:6])
```
<pre class="sortie">
[1 1 1 0 1 1]
</pre>

Il suffira d’insérer ce code, bit par bit, à la place des bits de poids faible qui ont été mis à zéro.  

Nous obtenons donc&nbsp;:

| octet<br>original | reset<br>(1 bit)| code<br>texte | octet<br>final|
|:-----------------:|:---------------:|:-------------:|:-------------:|
| 10110010          | 1011001**0**    | 1             | 1011001**1**  |
| 10000100          | 1000010**0**    | 1             | 1000010**1**  |
| 10010001          | 1001000**0**    | 1             | 1001000**1**  |
| 10110001          | 1011000**0**    | 0             | 1011000**0**  |
| 10000011          | 1000001**0**    | 1             | 1000001**1**  |
| 10010000          | 1001000**0**    | 1             | 1001000**1**  |

Lors de l’opération de décodage, il suffira de regrouper les derniers bits de chaque octet pour retrouver le code qui a été inséré dans l’image&nbsp;: `111011`...

En pratique, nous n’aurons même pas besoin de convertir les entiers du tableau `im_array` en binaire puisque l’opération qui consiste à remettre à zéro le bit de poids faible revient à retrancher 1 si le chiffre est impair et à le laisser tel quel s’il est pair.

Autrement dit, si nous appelons «&nbsp;V&nbsp;»&nbsp; la valeur originale de l’octet et «&nbsp;Vzero&nbsp;»&nbsp; sa valeur après mise à zéro du bit de poids faible, nous avons `Vzero = V - V % 2`.

Nous pouvons même réaliser cette opération de manière plus efficace (et plus rapide) en utilisant l’opérateur logique «&nbsp;&&nbsp;»&nbsp;&nbsp;: `Vzero = V - (V & 1)`.

Ainsi, si nous appliquons cette opération aux octets codant les deux premiers pixels, nous obtenons&nbsp;:

| V - (V & 1)|
|:---------:|
| 178 → 178 |
| 132 → 132 |
| 145 → 144 |
| 177 → 176 |
| 131 → 130 |
| 144 → 144 |  

Les pairs restent pairs, les impairs sont diminués d’une unité.

L’insertion du code consistera alors simplement à ajouter «&nbsp;0&nbsp;»&nbsp; ou «&nbsp;1&nbsp;»&nbsp; à chaque entier codant un octet.

| octet<br>initial | reset<br>(1 bit) | code<br>texte | octet<br>final |
|:----------------:|:----------------:|:-------------:|:--------------:|
| 178              | 178              | 1             | 179            |
| 132              | 132              | 1             | 133            |
| 145              | 144              | 1             | 145            |
| 177              | 176              | 0             | 176            |
| 131              | 130              | 1             | 131            |
| 144              | 144              | 1             | 145            |

Si nous utilisons uniquement le 8ème bit (bit de poids faible) de chaque octet de l’image pour insérer le texte sous forme de données binaires, le poids en octets du texte devra logiquement être huit fois (1 octet = 8 bits) inférieur à celui de l’image.  

Si le code du texte ne tient pas dans cet espace, il peut être tentant d’utiliser 2 bits faibles de chaque octet au lieu d’un, ce qui revient à **doubler** l’espace disponible pour le code. Et si cet espace de données ne suffit toujours pas, d’élargir à 3 bits/octet, puis 4, 5, etc.

La seule limite, c’est celle des 8 bits qui composent chaque octet de chaque pixel. Si cette limite est atteinte, il ne reste rien de l’image car tout son code a été remplacé.

<p style="text-align:center;">&#8258;</p>  

**Comment décider, en fonction de la taille de l’image et de celle du texte, du nombre de bits par octet à remplacer&nbsp;?**  

Concrètement, le programme va d’abord vérifier si les données binaires encodant le texte peuvent être entièrement contenues dans le tableau des pixels en utilisant uniquement le bit de poids le plus faible de chaque octet. Si c’est impossible, on poursuit la vérification avec 2 bits de poids faible, puis 3, etc.

Plus on utilisera de bits en remontant des bits de poids faible (à droite) vers ceux de poids fort (à gauche), plus la qualité de l’image sera altérée, sa destruction totale étant atteinte lorsque les 8 bits de l’octet sont utilisés.

La formule que nous avons vue plus haut pour mettre à zéro les bits de poids faible des octets peut être généralisée pour _n_ bits. Si «&nbsp;V&nbsp;»&nbsp; est la valeur de l’octet d’origine et «&nbsp;Vzero&nbsp;»&nbsp; sa valeur après mise à zéro de _n_ bits (en commençant par les bits de poids le plus faible), on a <code>Vzero = V - (V & (2<sup>n</sup> - 1))</code>.

Au maximum, sur le premier octet, on pourra stocker 8 bits de données du texte codé.

```python
# Maximum stockable sur les 6 premiers octets de im_array
print(octets_bin[:8])
```

<pre class="sortie">
[1 1 1 0 1 1 1 1]
</pre>

Le tableau suivant permet de visualiser les modifications apportées au premier octet en fonction du nombre de bits mis à zéro et le résultat obtenu après insertion du code&nbsp;:

| n | octet reset<br>dec/bin | code texte<br>dec/bin | octet final<br>dec/bin |
|--:|-----------------------:|----------------------:|-----------------------:|
| 0 | 178<br>10110010        | -<br>-                | 178<br>10110010        |
| 1 | 178<br>1011001**0**    | 1<br>1                | 179<br>1011001**1**    |
| 2 | 176<br>101100**00**    | 3<br>11               | 179<br>101100**11**    |
| 3 | 176<br>10110**000**    | 7<br>111              | 183<br>10110**111**    |
| 4 | 176<br>1011**0000**    | 14<br>1110            | 190<br>1011**1110**    |
| 5 | 160<br>101**00000**    | 29<br>11101           | 189<br>101**11101**    |
| 6 | 128<br>10**000000**    | 59<br>111011          | 187<br>10**111011**    |
| 7 | 128<br>1**0000000**    | 119<br>1110111        | 247<br>1**1110111**    |
| 8 | 0<br>**00000000**      | 239<br>11101111       | 239<br>**11101111**    |

#### Construction d’un *header*  

Si l’on détermine de manière dynamique le nombre de bits à remplacer sur chaque octet en fonction de la taille du message, l’opération de décodage sera un peu plus complexe que si cette valeur était fixe puisqu’on ignorera sur combien de bits de chaque octet le message a été réparti.

On peut certes y parvenir de manière empirique, en essayant d’abord avec 1 bit, puis 2, etc. Comme le message est codé en UTF-8 et que cette écriture impose des contraintes sur les bits forts de chaque octet, dès lors qu’on voudra convertir une suite de chiffres supposément codée en UTF-8, le programme lèvera une exception `UnicodeDecodeError` si cette suite ne correspond pas au formatage UTF-8.

Par exemple, si l’on tente de décoder une suite aléatoire d’octets, il y a peu de chances de tomber sur quelque chose de cohérent en UTF-8&nbsp;:

```python
# Tableau de 10 entiers aléatoires (0 < n < 255)
alea = np.random.randint(0, 256, 10, dtype=np.uint8)
print(alea)
msg = bytes(alea).decode('utf8')
```

<pre class="sortie">
[245 127  67 253  48  44  70 198 219 167]
...
<span style='color:#e75c58;'>UnicodeDecodeError</span>: 'utf-8' codec can't decode byte 0xf5 in position 0: invalid start byte
</pre>

Il suffira alors d’intercepter l’erreur au moyen de `try` et `except` pour éviter qu’elle ne bloque le programme, puis de réessayer avec `n+1` etc, jusqu’à ce que, pour une valeur donnée, une conversion en UTF-8 soit possible.

Toutefois, si cette méthode «&nbsp;à tâtons&nbsp;»&nbsp; pour trouver `nbits` fonctionne, elle n’est pas très élégante.

De plus, si l’on n’a pas d’indication sur la longueur du message qui a été inséré dans le tableau de pixels, on va devoir extraire les bits de tout le tableau, ce qui peut être pénalisant en temps de calcul, surtout si le message est court par rapport à la taille de l’image.

Pour ces raisons, il est préférable, au moment de l’encodage, de réserver quelques bits à l’enregistrement de ces deux informations&nbsp;: le nombre de bits/octet (`nbits`) et la longueur du message.

Les opérations de décodage seront alors bien plus rapides.

**Taille du _header_**

Un demi-octet sera réservé à l’enregistrement du nombre de bits/octet (`nbits` de 1 à 8). Comme le message codé s’étalera sur un nombre d’octets dont la taille maximum sera celle de l’image, il suffira pour le noter (en binaire) de réserver `len(f'{im_array.size:b}')` bits. Ces bits étant regroupés en demi-octets, le nombre de bits utilisés devra être un multiple de 4, arrondi à l’excès.   

La longueur totale du *header*, exprimée en demi-octets, sera&nbsp;:

```python
header_size = 1 + ceil(len(f'{im_array.size:b}') / 4)
print(header_size)
```
<pre class="sortie">
7
</pre>

La taille utile de l’image (la partie où l’on pourra stocker l’information du message) est donc&nbsp;:

```python
im_size = im_array.size - header_size # taille utile en octets
print(im_size)
```
<pre class="sortie">
3275513
</pre>

#### Calcul de `nbits` (nombre de bits&nbsp;/&nbsp;octet)  

Puisqu’on connaît la taille utile de l’image et celle du message à encoder, il est facile de calculer le nombre de bits (`nbits`) par octet qui seront utilisés. La règle à observer est de choisir `nbits` le plus bas possible afin de préserver la qualité de l’image.

La boucle suivante nous permettra de déterminer le `nbits` le plus bas&nbsp;:

```python
for nbits in range(1, 9): # groupes d’octets
    if im_size >= len(octets) * 8/nbits:
        break # nbits minimal trouvé -> sortie de boucle
else:
    # Erreur si message trop long même avec nbits à 8
    raise ValueError("Texte trop long pour cette image")

# Si nbits à 8, avertissement et poursuite du programme
if nbits == 8:
    print("*"*60, "\nAvertissement : Les tailles sont compatibles mais"
          "\nl’image sera totalement écrasée par le texte."
          "\nPour un meilleur résultat, choisissez une image plus grande.")
    print("*"*60)
```

On notera qu’on a fait deux choix&nbsp;:
- Dans le cas où le message est trop long pour être inséré dans l’image, on lève une exception. On aurait pu également faire le choix de tronquer le message et de n’en insérer qu’une partie.

- On conserve la possibilité `nbits = 8`, alors que son intérêt est limité puisque l’image est alors totalement détruite. On avertit simplement l’utilisateur de cette situation. On pourrait faire ici le choix de ne pas proposer cette option en considérant que `nbits = 7` est le maximum acceptable.  

On peut calculer (et afficher) le taux d’occupation de l’image par le texte du message, sachant que le maximum est atteint lorsque `nbits = 8` et que tous les bits de tous les octets de l’image ont été remplacés.

```python
# Calcul du taux d’occupation de l’image
print("Nombre de bits/octet remplacés :", nbits)
print('Taux de bits utilisés : ', round(len(octets) * 100 / im_size, 2), '%')
```
<pre class='sortie'>
Nombre de bits/octet remplacés : 2
Taux de bits utilisés : 21.92 %
</pre>

Après avoir codé le texte sous forme de tableau de «&nbsp;0&nbsp;» et de «&nbsp;1&nbsp;» et déterminé le nombre de bits qui seront insérés dans chaque octet, il reste à regrouper ces bits par groupes de _n_ et convertir ces derniers dans le système décimal. Il ne restera plus alors qu’à les ajouter aux octets du tableau de pixels, après avoir au préalable mis à zéro les `nbits` de poids faible de ces derniers.

Puisque les éléments de `octets_bin` doivent être regroupés par paquets de `nbits`, leur nombre doit être un multiple de `nbits`. Si ce n’est pas le cas, il faudra faire un *padding* (remplissage), c’est-à-dire ajouter des «&nbsp;0&nbsp;» jusqu’à obtenir un multiple de `nbits`.

Ces opérations sont faciles et rapides grâce à NumPy.

```python
if octets_bin.size % nbits :  # n’est pas multiple de nbits -> padding
    masque = np.zeros(nbits * ceil(octets_bin.size / nbits), 
                      dtype=np.uint8)
    masque[:octets_bin.size] = octets_bin
    octets_bin = masque
```

On a ainsi complété avec des «&nbsp;0&nbsp;» le tableau `octets_bin` de façon à ce que le nombre d’éléments soit un multiple de `nbits`.  

Il ne reste qu’à regrouper les éléments par groupes de `nbits`, à les agréger et à convertir les groupes ainsi formés en base 10.

```python
code_txt = octets_bin.reshape(-1, nbits)
```

Notons que cette instruction produirait une erreur si le nombre d’éléments du tableau `octets_bin` n’était pas multiple de `nbits`.

Dans notre exemple, comme `nbits` est égal à 2, le regroupement se fait par paires, comme l’indiquent le format et les premiers éléments du tableau&nbsp;:


```python
print("Format :", code_txt.shape)
print(code_txt[:10, :])
```

<pre class='sortie'>
Format : (2872180, 2)
[[1 1]
 [1 0]
 [1 1]
 [1 1]
 [1 0]
 [1 1]
 [1 0]
 [1 1]
 [1 0]
 [1 1]]
</pre>

La fonction `np.packbits()` permet de convertir les paires de bits en entiers, mais sous forme d’octets complets. Elle va donc ajouter des «&nbsp;0&nbsp;» à droite&nbsp;:

```python
print(np.packbits(np.array([1, 1, 0, 1, 0])))
```
<pre class="sortie">
[208]
</pre>

La suite [1, 1, 0, 1, 0] a été convertie en `11010000`, en remplissant l’octet à droite avec trois «&nbsp;0&nbsp;».  

Si l’on précise le paramètre `bitorder='little'`, les bits d’entrée seront inversés&nbsp;:

```python
print(np.packbits(np.array([1, 1, 0, 1, 0]), bitorder='little'))
```
<pre class='sortie'>
[11]
</pre>

Cette fois [1, 1, 0, 1, 0] a été transposé en `00001011`, avec un remplissage de zéros à droite puis une inversion par symétrie gauche-droite.

Si l’on souhaite que [1, 1, 0, 1, 0] soit lu `00011010`, il faut donc d’abord procéder à l’inversion des bits, ce que permet la fonction `np.flip()`, puis utiliser `np.packbits()` avec le paramètre `bitorder='little'`&nbsp;:

```python
ar_inv = np.flip(np.array([1, 1, 0, 1, 0]))
print(np.packbits(ar_inv, bitorder='little'))
```
<pre class='sortie'>
[26]
</pre>

Cette fois, on obtient bien le résultat souhaité, 26 étant la représentation décimale de `00011010`.  

Comme ces deux opérations serviront plusieurs fois, regroupons-les dans une fonction&nbsp;:

```python
def decimalisation(code, axe):
    """
    Compresse les bits d’un tableau après renversement symétrique
    et retourne un tableau d’entiers (uint8) à une dimension.
    """   
    ar = np.flip(code, axis=axe)
    return np.packbits(ar, axis=axe, bitorder='little').ravel()
```

On a en outre remis le tableau «&nbsp;à plat&nbsp;» grâce à la méthode `ravel()`. Cela facilitera ensuite l’insertion du code dans la table de pixels.

On peut vérifier que les bits ont bien été regroupés et décimalisés&nbsp;:

```python
code_txt = decimalisation(code_txt, 1)
print(code_txt[:20])
```

<pre class='sortie'>
[3 2 3 3 2 3 2 3 2 3 3 3 1 1 1 0 1 2 2 0]
</pre>

#### Mise à zéro des bits et insertion du code

Le `nbits` (nombre de bits remplacés dans chaque octet) a été arbitrairement fixé à 4 pour le *header* mais il est variable pour le reste du tableau puisqu’il est déterminé dynamiquement en fonction de la taille des données.

Les opérations de mise à zéro des bits devront donc être réalisées séparément pour le *header* et pour le reste du texte.

Extrayons tout d’abord du tableau `im_array` les quelques octets de tête qui serviront de `header`&nbsp;:

```python
header = im_array.flatten()[:header_size]
```

On utilise ici `flatten()` de préférence à `ravel()` car `flatten()` retourne une copie tandis que `ravel()` renvoie une vue. Or, on veut pouvoir modifier le *header* sans modifier le tableau `im_array` et réciproquement, ce qui n’est possible qu’avec une vraie copie.

Comme on l’a vu plus haut, la taille du *header* étant de 7 octets, ce sont les 7 premiers octets codant les pixels de l’image que l’on retrouve dans ce tableau&nbsp;:

```python
print(header)
```

<pre class="sortie">
[178 132 145 177 131 144 179]
</pre>

La mise à zéro de `nbits` est donnée par la formule <code>Vzero = V - (V & (2<sup>n</sup> - 1))</code> où n vaut ici 4. Pour le reste du tableau, il vaut `nbits`.


```python
# Mise à zéro des 4 bits de poids faible des octets du header
header -= header & (2**4 - 1)
# Idem pour les nbits de poids faible des autres octets du tableau
im_array -= im_array & (2**nbits - 1)
```

Les bits de poids faible des octets du tableau et du *header* ont été mis à zéro. On peut s’en assurer en imprimant les nouvelles valeurs du *header*&nbsp;:

```python
print(header)
```

<pre class='sortie'>
[176 128 144 176 128 144 176]
</pre>

Si l’on convertit ces chiffres en base deux, on voit que les 4 bits de droite ont bien été mis à zéro&nbsp;:

```python
for n in header:
    print(f'{n:b}', end=' ')
```

<pre class="sortie">
10110000 10000000 10010000 10110000 10000000 10010000 10110000
</pre>  

<p style="text-align:center;">&#8258;</p>  

**Écriture des bits de poids faible du _header_**   

Le *header* sera composé du nombre `nbits` compris entre 1 et 8 (sur un demi-octet) et de l’indication de la taille (en octets) du code texte inséré dans le tableau de pixels.

Il sera d’abord écrit en binaire, puis encodé avec la fonction `decimalisation()` que nous avons déclarée plus haut.

```python
# Code du header en notation binaire
header_code = (f'{nbits:04b}' + 
               f'{code_txt.size:0{4 * (header_size - 1)}b}')
```

Dans notre exemple, on obtient&nbsp;:

```python
print(len(header_code)) # nombre de bits du header
print(header_code)
print(code_txt.size) # taille du texte codé en octets
```
<pre class="sortie">
28
0010001010111101001101110100
2872180
</pre>

Le *header* a une longueur de 7 octets (28/4) comme prévu. Il comprend d’abord l’indication du `nbits` (ici `0010` soit 2 en notation décimale) et le binaire `001010111101001101110100` (2&#8239;872&#8239;180 en base 10) qui indique la longueur du code (`code_txt`) à insérer.

Ces valeurs peuvent être regroupées par 4 et mises en forme dans un tableau à deux dimensions&nbsp;:

```python
header_code = np.array(list(header_code), dtype=np.uint8).reshape(-1, 4)
print(header_code)
```

<pre class='sortie'>
[[0 0 1 0]
 [0 0 1 0]
 [1 0 1 1]
 [1 1 0 1]
 [0 0 1 1]
 [0 1 1 1]
 [0 1 0 0]]
</pre>

Il ne reste qu’à agréger les groupes, à convertir le résultat en base 10, et à aplatir le tableau. Tout cela peut être réalisé au moyen de la fonction `decimalisation()`&nbsp;:

```python
header_code = decimalisation(header_code, 1)
print(header_code)
```
<pre class='sortie'>
[ 2  2 11 13  3  7  4]
</pre>

Enfin, on fusionne les deux codes, celui du texte et de l’image, en additionnant les valeurs des tableaux&nbsp;:

```python
im_array.ravel()[:header_size] = header + header_code
im_array.ravel()[header_size:header_size+code_txt.size] += code_txt
```

Les codes ont bien été fusionnés. On peut le vérifier en affichant le nouveau `im_array` dont les octets présentent de légères variations par rapport aux valeurs d’origine&nbsp;:

```python
# Valeurs des octets des 9 premiers pixels
print(im_array[:3, :3, :])  
```
<pre class='sortie'>
[[[178 130 155]
  [189 131 151]
  [180 135 146]]

 [[168 124 139]
  [174 124 138]
  [181 133 148]]

 [[182 132 149]
  [177 131 141]
  [177 129 147]]]
</pre>

Il ne reste qu’à sauvegarder l’image ainsi modifiée grâce à PILLOW&nbsp;:

```python
im = Image.fromarray(im_array)
im.save('image_codee.png') # sauvegarde locale de l’image codée
```

Si l’on compare les deux images, l’originale et celle nouvellement créée après injection du code, on ne perçoit pratiquement pas de différence&nbsp;:  
{{< figure class="small" src="sample_1280×853.png" link="sample_1280×853.png" alt="Image originale" title="Original">}}
{{< figure class="small" src="image_codee.png" link="image_codee.png" alt="Image + Le Discours de la Méthode (Descartes)" title="Image + code">}}
<!--
[<img style="margin:0.5em; padding:0.5em; width:200px; float: left" src="sample_1280×853.png">](sample_1280×853.png)
[<img style="margin:0.5em; padding:0.5em; width:200px; float: left" src="image_codee.png">](image_codee.png)
-->
Pourtant, la deuxième contient tout le volume encodé du *Discours de la méthode* de Descartes et pourrait contenir presque 5 fois plus de données, le taux d’occupation de l’image étant de 21,92&nbsp;%.

Comme l’image choisie était assez grande par rapport au volume de données insérées, seuls deux bits par octet ont été utilisés. Et comme il s’agit des bits dits «&nbsp;de poids faible&nbsp;», l’impact sur la qualité de l’image est assez limité.

Mais si l’on augmente le nombre de données (ou si l’on diminue la taille de l’image), le `nbits` (nombre de bits/octet) à remplacer augmente et l’altération de l’image commence à se faire sentir.

Voici à titre indicatif les transformations que subit la même image lorsqu’on remplace respectivement 5 bits, 6 bits, 7 bits et 8 bits par octet&nbsp;:

<details>
<summary>  
		Afficher...
</summary>
<table ><tr><td style='border: 0; font-family: inherit;'>
{{< figure class="small" src="sample_640×426.png" link="sample_640×426.png" alt="Image originale" title="sample_640×426.png">}}
{{< figure class="small" src="sample_4.png" link="sample_4.png" alt="nbits = 4" title="nbits = 4">}}
{{< figure class="small" src="sample_5.png" link="sample_5.png" alt="nbits = 5" title="nbits = 5">}}
{{< figure class="small" src="sample_6.png" link="sample_6.png" alt="nbits = 6" title="nbits = 6">}}
{{< figure class="small" src="sample_7.png" link="sample_7.png" alt="nbits = 7" title="nbits = 7">}}
{{< figure class="small" src="sample_8.png" link="sample_8.png" alt="nbits = 8" title="nbits = 8">}}
</td></tr></table>
</details>  

<hr>

#### Décodage

Pour décoder une image, on part bien sûr du principe qu’on ignore si l’image a été ou non encodée. On ignore également le nombre de bits/octet occupés par un éventuel message, de même que la longueur de ce dernier.

Ces deux dernières informations nous seront révélées par la lecture du *header*.

On commence un nouveau script, et donc on importe les modules nécessaires&nbsp;:

```python
# coding: utf8
from math import ceil
from PIL import Image
import numpy as np
```

On enregistre les valeurs des pixels dans un tableau NumPy&nbsp;:

```python
im_array = np.array(Image.open('image_codee.png'))
```

Vérifions que les données ont bien été enregistrées&nbsp;:

```python
print(im_array.size)
print(im_array[:2, :5, :]) # on imprime quelques octets
```
<pre class='sortie'>
3275520
[[[178 130 155]
  [189 131 151]
  [180 135 146]
  [167 123 134]
  [171 126 139]]

 [[168 124 139]
  [174 124 138]
  [181 133 148]
  [177 129 145]
  [181 132 147]]]
</pre>

Calculons la taille du *header* qui est fonction de la taille de l’image&nbsp;:

```python
head_binsize = len(f'{im_array.size:b}')
head_size = ceil(head_binsize / 4) + 1 # taille en octets
```
Sans surprise, `head_size` nous indique que le *header* s’étale sur 7 octets&nbsp;:

```python
print(head_size)
```
<pre class='sortie'>
7
</pre>

Imprimons maintenant les valeurs du *header*&nbsp;:

```python
header = im_array.ravel()[:head_size]
print(header)
```
<pre class='sortie'>
[178 130 155 189 131 151 180]
</pre>

Dans ces chiffres sont mêlés la valeur des pixels d’origine, et le code ajouté, celui-ci occupant (en base 2) les 4 bits faibles, ceux de droite.  

Pour récupérer ce code&nbsp;:

```python
header = header & (2**4 - 1) # formule équivalente à 'header % 2**4'
print(header)
```

<pre class='sortie'>
[ 2  2 11 13  3  7  4]
</pre>

On a bien récupéré, sous forme décimale, les valeurs représentées par les 4 bits à droite de chaque chiffre&nbsp;:

| octets<br>*header* | déc. | information |
|-------------------:|-----:|------------:|
| 178<br>1011**0010**| 2    | nbits = 2   |
| 130<br>1000**0010**| 2    | [00]10      |
| 155<br>1001**1011**| 11   | 1011        |
| 189<br>1011**1101**| 13   | 1101        |
| 131<br>1000**0011**| 3    | 0011        |
| 151<br>1001**0111**| 7    | 0111        |
| 180<br>1011**0100**| 4    | 0100        |

Les deux informations qui nous intéressent sont le `nbits` (ici égal à 2) indiqué par le 1er octet, et la longueur du message inséré dont la valeur (en binaire) est obtenue par concaténation des représentations binaires des octets suivants (`[00]10`+`1011`+`1101`+`0011`+`0111`+`0100`), transcrite ensuite en décimal&nbsp;:


```python
# Lecture du premier octet
nbits = header[0]
# Valeur en binaire de la longueur du message
bin_code_size = ''.join(f'{c:04b}' for c in header[1:])
# Conversion en base décimale
code_size = int(bin_code_size, 2)
```

```python
print(nbits)
print(code_size)
```
<pre class='sortie'>
2
2872180
</pre>

On procède de même sur le reste du tableau. On élimine d’abord tous les bits de poids fort (à gauche), puis on dispose les bits de poids faible sur une colonne moyen d’un `reshape()`, avant de décompresser les bits de chaque élément de la colonne&nbsp;:

```python
im_array = im_array & (2**nbits - 1)
im_array = im_array.reshape(-1, 1)[head_size:head_size + code_size]
code_txt = np.unpackbits(im_array, axis=1)
```

On obtient alors une colonne d’octets dont les premiers éléments sont&nbsp;:

```python
print(code_txt[:5])
```
<pre class='sortie'>
[[0 0 0 0 0 0 1 1]
 [0 0 0 0 0 0 1 0]
 [0 0 0 0 0 0 1 1]
 [0 0 0 0 0 0 1 1]
 [0 0 0 0 0 0 1 0]]
</pre>

On va ensuite demander à `np.packbits()` de concaténer les deux derniers bits de chaque octet (puisque dans notre exemple `nbits` est égal à 2) pour composer de nouveaux octets, et de renvoyer le tout en base décimale.  

Le premier octet devra donc être la concaténation `11`+`10`+`11`+`11`, soit le nombre binaire `11101111`, autrement dit 239 (`int('11101111', 2)`) en base décimale.


```python
code_txt = np.packbits(code_txt[:, 8-nbits:])
# On retire d’éventuels octets nuls
code_txt = np.trim_zeros(code_txt, 'b')
```

On vérifie qu’on a bien récupéré l’ensemble des octets du texte original&nbsp;:

```python
print(code_txt)
```
<pre class='sortie'>
[239 187 191 ...  42  10  10]
</pre>

Il ne reste plus qu’à décoder avec `bytes()`, selon le processus inverse de celui de l’encodage.

Si l’image ne contient pas de code (ou si le code a été altéré), c’est au moment de l’exécution de cette instruction qu’une exception de type `UnicodeDecodeError` devrait se produire. On va donc insérer le code dans un bloc `try`... `except` de façon à capturer l’exception si elle se produit.

```python
try:
    message = bytes(code_txt).decode('utf8')
except UnicodeDecodeError:
    print("Cette image ne contient pas de code" 
          " ou le code est illisible.")
else:
    print('Début du texte :', message[:73])
    print(f'Longueur du texte : {len(message)} caractères')
```

<pre class='sortie'>
Début du texte : ﻿The Project Gutenberg EBook of Discours de la méthode, by René Descartes
Longueur du texte : 700602 caractères
</pre>

L’ensemble du texte a bien été récupéré et transféré dans la variable `message`.

On peut, si on le souhaite, sauvegarder ce texte dans un fichier&nbsp;:

```python
# Sauvegarde locale du texte
with open('nouveau_texte.txt', 'w') as f:
  f.write(message)
```
<hr style="margin: 2em 0 0 0;">  
<!--
Voir le code sur GitHub&nbsp;:  
<https://github.com/...>
-->