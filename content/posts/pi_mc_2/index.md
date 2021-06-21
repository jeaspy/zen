---
title: "Calcul de π par la méthode de Monte-Carlo"
date: 2021-03-14T22:38:02+01:00
description: "Construire deux animations graphiques avec Matplotlib."
#categories: [NumPy, Matplotlib, Python]
head_pict: false
weight: 3
dropCap: false
displayInMenu: true
displayInList: true
draft: false
katex: true
resources:
- name: featuredImage
  src: "pi_vert.png"
  params:
    chapeau: "Ⅱ - Représentations"

---
# 2. Représentations
<br>

{{<smallimg src="matplotlib.jpg" alt="Logo Matplotlib" smartfloat="right" width="120px">}}
<span class="lettrine">G</span>râce à [Matplotlib](https://matplotlib.org/ "matplotlib.org"), nous allons créer deux animations qui vont nous permettre d’observer graphiquement la convergence vers {{< raw >}}\(\pi\){{< /raw >}}.

<h4 class="cool">1. Remplissage de la cible</h4>

La première permet d’observer le remplissage de la cible et de suivre en temps réel l’approximation de {{< raw >}}\(\pi\){{< /raw >}} qui en résulte. Le calcul est actualisé à chaque intervalle de l’animation.

Le script permet de modifier certains paramètres&nbsp;: le nombre total de points placés (_nb_total_), le nombre de points placés simultanément (_group_), d’exercice l’intervalle des _frames_ (_interval_) qui va régler la vitesse de l’animation, enfin d’autres aspects esthétiques&nbsp;: couleur, symboles (_markers_), transparence.

D’autres explications sont données directement sous forme de commentaires dans le code. 

Voici un aperçu du résultat, avec ici 10&#8239;000 points placés par groupes de 40 avec un intervalle de 80 ms&nbsp;: 
<div style="width: 100%; overflow: hidden;">
<video style="width: 100%;" controls autoplay muted>
    <source src="pi_target.mp4" type="video/mp4">
    Your browser does not support the video tag.  
</video>
</div>

```python
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
import locale

locale.setlocale(locale.LC_ALL, '') # séparation des milliers avec format {:n}
rng = np.random.default_rng()       # initialisation du PRNG

###############################################################################
# Choix du backend :
# On peut laisser le backend par défaut ou choisir dans les listes ci-dessous :

# Sur Jupyter Notebook / iPython
# avec nbAgg : 
# %matplotlib notebook
# ou choisir dans la liste : %matplotlib -l

# Autre choix (exemple) :
# matplotlib.use('GTK3Agg')       
# ou choisir dans la liste : matplotlib.rcsetup.interactive_bk

###############################################################################
# Paramètres modifiables :

nb_total = 10_000  # nombre de points au total (entier supérieur ou égal à 1)
group = 40         # points groupés (entier supérieur ou égal à 1)
interval = 80      # intervalle (ms) entre deux frames (vitesse de l’animation)

in_sz, out_sz = 60, 60                  # tailles des points (int. et ext.)
in_mk, out_mk = ',', ','                # types de symboles (int. et ext.)
in_co, out_co = 'tab:pink', 'tab:blue'  # couleurs (int.et ext.)
in_al, out_al = 0.12, 0.12              # transparence (0 à 1)
###############################################################################

# Tirage aléatoire des coordonnées x et y. On utilise ici 2 tableaux
# distincts car la question de la reproductibilité ne se pose pas.
x = rng.uniform(-1, 1, nb_total)
y = rng.uniform(-1, 1, nb_total)

in_bool = x*x + y*y < 1         # tableau booléen des tirs "réussis"
out_bool = ~in_bool             # tableau booléen inverse

# Initialisation de l’interface Matplotlib
fig, ax = plt.subplots(figsize=(6, 6))

ax.set_xlim(-1, 1)
ax.set_ylim(-1, 1)

ax.set_xticks([-1, 0, 1])
ax.set_yticks([0, 1])

# Affichage des informations
ax.set_title('Approche de $\pi$ par la méthode de Monte-Carlo')

box = dict(boxstyle='round', ec='dimgray', fc='whitesmoke', alpha=0.8)
strtxt = '\n'.join(('Total : {:n}', 'Cible : {:n}', '$\pi$ \u2248 {:.10f}'))

txt = ax.text(0.05, 0.95, '', transform=ax.transAxes, c='dimgray', 
              fontfamily='monospace', va='top', bbox=box)

# Nuages de points internes et externes
scat_in = ax.scatter([], [], c=in_co, s=in_sz, marker=in_mk, alpha=in_al)
scat_out = ax.scatter([], [], c=out_co, s=out_sz, marker=out_mk, alpha=out_al)


def animate(i):
    """ Mise à jour des frames de l’animation"""
    k = min(nb_total, i*group)
    for io, scat in zip ((out_bool, in_bool), (scat_out, scat_in)):
        xk, yk = x[:k][io[:k]], y[:k][io[:k]]
        data = np.hstack((xk.reshape(-1, 1), yk.reshape(-1, 1)))
        scat.set_offsets(data)
    txt.set_text(strtxt.format(k, xk.size, 4*xk.size/k))
    return scat_in, scat_out, txt
    
anim = FuncAnimation(fig, animate, interval=interval, blit=True, 
                     frames=range(1, nb_total//group+2), repeat=False)

plt.show()

# Si sauvegarde, supprimer ou commenter plt.show()
# anim.save('pi_target.mp4')
# anim.save('pi_target.gif')
# plt.close(fig)
```

Sur [Jupyter Notebook](https://jupyter.org/ "jupyter.org"), il est aussi possible de lire cette animation dans une balise HTML5 grâce à la méthode `to_html5_video`. Elle est alors encodée en base64 directement dans la balise vidéo. On peut également créer un widget JavaScript interactif avec la méthode `to_jshtml`. Dans ce cas, il peut être utile d’augmenter la mémoire allouée à Matplotlib. Notons que, dans les deux cas, le temps de chargement peut être assez long.


```python
from IPython.display import HTML
# html5
HTML(anim.to_html5_video()) 

# jshtml :
# matplotlib.rcParams['animation.embed_limit'] = 2**128 # augmentation mémoire
# HTML(anim.to_jshtml())
```
<br>

#### 2. Courbe de convergence

Une autre façon de représenter graphiquement la convergence est de tracer une courbe portant en abscisse le nombre total de points tracés et en ordonnée la valeur de {{< raw >}}\(\pi_{mc}\){{< /raw >}} obtenue. La principale difficulté, ici, est d’éviter que la courbe ne s’aplatisse trop rapidement. Les fluctuations étant de plus en plus faibles, c’est naturellement ce qu’elle tend à faire. 

Mais nous verrons plus loin que l’[écart-type](https://fr.wikipedia.org/wiki/%C3%89cart_type "Wikipédia : Écart type
") des valeurs de {{< raw >}}\(\pi_{mc}\){{< /raw >}} converge vers 0 en {{< raw >}}\(\dfrac{1}{\sqrt{n}}\){{< /raw >}}, _n_ désignant ici, comme dans la suite, le nombre total de points. Il suffira donc d’adapter la graduation sur l’axe des y en fonction de {{< raw >}}\(\dfrac{1}{\sqrt{n}}\){{< /raw >}}, et de la moduler au moyen d’un coefficient. Ainsi, la fluctuation demeurera visible, même pour des valeurs élevées de _n_.

Plusieurs des points relevés dans la partie précédente sont mis ici en application&nbsp;:

- La possibilité de définir un pas d’incrément (_step_), de façon à observer la convergence à des intervalles prédéfinis&nbsp;;

- La possibilité de partir d’un résultat intermédiaire connu, en utilisant la fonction `advance()` du générateur&nbsp;;

- La reproductibilité assurée par le choix possible d’une graine aléatoire (_seed_).

À cela s’ajoutent&nbsp;:

- La définition d’un point de focus. Si par exemple, on définit le pas d’incrément à 1&#8239;000&#8239;000 (1 million) mais qu’on veut observer la fluctuation au niveau du point 15&#8239;125&#8239;698, il est possible d’indiquer cette valeur comme «&nbsp;focus&nbsp;». L’animation affichera logiquement la courbe pour _n_&nbsp;=&nbsp;15&#8239;000&#8239;000, mais au lieu de passer directement à 16&#8239;000&#8239;000, elle passera par la valeur intermédiaire 15&#8239;125&#8239;698 qui lui a été indiquée&nbsp;;

- Point d’arrêt. Il est possible de stopper l’animation sur une étape prévue (multiple de *step*) ou sur le point de focus.

Cette animation permet donc d’observer en temps réel la convergence vers {{< raw >}}\(\pi\){{< /raw >}}, mais aussi de visualiser et de capturer des instants privilégiés où sont atteints des records de précision. 

Voici par exemple le moment où {{< raw >}}\(\pi_{mc}\){{< /raw >}} atteint une précision de 15 décimales exactes. Nous verrons plus loin comment découvrir de telles perles et comment programmer le PRNG pour l’amener sur ces records.

<div style="width: 100%; overflow: hidden;">
<video style="width: 100%;" controls autoplay muted>
    <source src="plotting_pi.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>
</div>

**À propos du script**

Comme le nombre de décimales de {{< raw >}}\(\pi\){{< /raw >}} exactes est calculé à chaque actualisation de l’animation, il nous faut un {{< raw >}}\(\pi\){{< /raw >}} de référence. 

Il nous sera fourni par le module local _pi\_string_ qui est appelé dans l’entête du script. Ce module contient une fonction qui renvoie une chaîne de caractères représentant {{< raw >}}\(\pi\){{< /raw >}} avec le nombre de décimales correspondant à la précision demandée. 

Nous pourrions certes nous contenter d’une simple chaîne «&nbsp;3.1415926...&nbsp;», mais il semble plus pratique d’avoir sous la main un module qui calcule {{< raw >}}\(\pi\){{< /raw >}} à la volée avec le nombre de décimales demandé et qui fixe, dans le même temps, la précision avec laquelle seront effectuées les opérations du module _decimal_. 

Cette fonction `pi()` est empruntée aux exemples de «&nbsp;[cas pratiques](https://docs.python.org/fr/3/library/decimal.html#recipes "docs.python.org")&nbsp;» cités dans la documentation officielle de Python à la fin du chapitre consacré au module _decimal_. À partir de cette fonction, nous créons le module _pi\_string.py_ qui pourra être importé dans le programme principal chaque fois que nous en aurons besoin.

En voici le contenu&nbsp;:


```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

"""
Fonction pour calculer Pi à la précision souhaitée en définissant le nombre 
de décimales et en renvoyant le résultat sous forme de chaîne de caractères.

100 décimales :
3.1415926535 8979323846 2643383279 5028841971 6939937510 
  5820974944 5923078164 0628620899 8628034825 3421170679

Référence :
https://docs.python.org/fr/3/library/decimal.html#module-decimal
"""

from decimal import Decimal, getcontext
getcontext().rounding = 'ROUND_DOWN'

def pi(prec):
    getcontext().prec = prec + 5
    three = Decimal(3)
    lasts, t, s, n, na, d, da = 0, three, 3, 1, 0, 0, 24
    while s != lasts:
        lasts = s
        n, na = n+na, na+8
        d, da = d+da, da+32
        t = (t * n) / d
        s += t
    getcontext().prec -= 4
    return str(+s)
```

Nous avons maintenant tous les éléments nécessaires pour coder l’animation&nbsp;:


```python
import locale
from decimal import Decimal as D, getcontext
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
from pi_string import pi            # pour calculer π à la précision voulue

locale.setlocale(locale.LC_ALL, '') # séparation des milliers avec format {:n}
pi_ref = pi(25)                     # π sous forme de string avec 25 décimales
getcontext().prec += 4              # meilleure précision pour les calculs

rec = 0
spotts = {}

###############################################################################
# Choix du backend :
# On peut laisser le backend par défaut ou choisir dans les listes ci-dessous :

# Sur Jupyter Notebook ou iPython
# avec nbAgg : 
# %matplotlib notebook
# ou choisir dans la liste %matplotlib -l

# Autre choix (exemple) :
# matplotlib.use('GTK3Agg')
# ou choisir dans la liste matplotlib.rcsetup.interactive_bk

###############################################################################
# Paramètres modifiables :

step = 1_000_000  # Pour une progression + rapide, le graphe affichera les 
                  # résultats par groupes de n points (= step). *step* est un 
                  # entier compris entre 1 et ~ 2 000 000 (voire plus).
                    
seed = None       # graine aléatoire (pour reproductibilité). 
                  # Entier ou séquence d’entiers.

ampli = 3.5       # amplitude verticale de la courbe, strictement positif. 
                  # Plus le chiffre est bas, plus la courbe est ample ; 
                  # plus il est haut, plus elle est plate. Si le chiffre 
                  # est bas, elle peut sortir du graphe + ou - longtemps.

inter = 80        # intervalle de temps (ms) entre deux frames 
                  # (vitesse de l’animation) 

ndec = 5          # nombre de décimales correctes minimal pour que 
                  # l’animation émette un signal (couleur/pause) 

nb_cible = 0      # Si l’on souhaite partir d’un résultat intermédiaire, on
nb_total = 0      # peut saisir ici les valeurs de départ : nombre de points
                  # dans le cercle (*nb_cible*) et nombre total (*nb_total*), 
                  # en n’oubliant pas de préciser le *seed* correspondant 
                  # à ces valeurs.

focus = None      # Si l’on souhaite que la courbe passe par un point précis
                  # qui n’est pas multiple de *step*, on peut l’indiquer ici. 
                  # Ce doit être un entier strictement supérieur à *step*.
                    
stop = None       # Si l’on souhaite que l’animation s’arrête une fois atteint
                  # un nombre de points précis, on indique ici cette limite. 
                  # C’est un entier qui peut être la variable *focus* (si elle
                  # a été déterminée) ou un multiple de *step*.
###############################################################################

# Traitement du bug de plt.pause() pour certains backends
has_bug = True if matplotlib.get_backend() in ('WebAgg', 'nbAgg') else False

# Initialisation du PRNG
rng = np.random.Generator(np.random.PCG64(seed).advance(nb_total*2))

# Si *focus* a été déterminé, calcul des intervalles intermédiaires
if focus and focus > step:
    n, fstep = divmod(focus, step)
    spotts = {n * step: fstep, focus: step - fstep}

def count_dec(pi_mc):
    """ 
    Comptage des décimales exactes de pi. Renvoie le nombre de décimales
    exactes.
    """
    k = 0
    while str(pi_mc)[:k] in pi_ref:
        k += 1
    return max(k-3, 0)

def get_new_pi():
    """
    Calcul d’une nouvelle approximation de pi_mc en fonction du nombre total de 
    points (nb_total) et de ceux à l’intérieur du cercle (nb_cible). 
    Ces deux variables sont actualisées.
    """
    global nb_cible, nb_total
    if nb_total in spotts:
        ntab = spotts[nb_total]
    else:
        ntab = step
    nb_total += ntab
    tab = rng.random((ntab, 2))
    nb_cible += np.sum(tab[:, 0]**2 + tab[:, 1]**2 < 1)
    pi_mc = (4 * D(int(nb_cible))/D(nb_total)).quantize(D(pi_ref))
    return pi_mc

def set_colors(cols=('m', 'gray'), c_line='#1f77b4'):
     """ Couleurs des annotations et de la courbe."""
     line.set_c(c_line)
     for item, col in zip((top, pi_logo), cols):
        item.set_c(col)
        item.get_bbox_patch().set(ec=col)

# Tracé de l’interface Matplotlib
fig = plt.figure(figsize=(10, 6))
plt.axhline(y = np.pi, c='r', ls='dotted', lw='0.5')

# Éléments textuels
topbox = dict(boxstyle="round4", fc="0.9")
pi_box = dict(boxstyle="circle", fc='1.0')

str_infos = f'\u2714 {rng} \u2714 step {step:n}'+(f' \u2714 seed {seed:n}' 
                                                  if seed is not None else '')
str_board = '\n'.join(('\u25cf {} décimales exactes', '$\pi$ \u2248 {}', 
                       'Nb. total : {:n}', 'Nb. cible : {:n}'))
top = plt.annotate('', (0.58, 0.75), fontfamily='monospace', 
                   xycoords='axes fraction', bbox=topbox)
pi_logo = plt.annotate('$\pi$', (0.52, 0.5), xycoords='axes fraction', 
                       va='center', size='large', bbox=pi_box)
board_txt = plt.annotate('', (0.01, 0.86), fontfamily='monospace', 
                         c='gray', xycoords='axes fraction')
infos = plt.annotate(str_infos, (0.01, 0.01), c='g', size='small', 
                     xycoords='axes fraction')

plt.title('Méthode de Monte-Carlo — Convergence vers $\pi$')
plt.xlabel(f'Nombre de tirs / {step:n}')
plt.ylabel('Valeur approchée de $\pi$')

# Initialisation du graphique.
x_vals = []
y_vals = []
line, = plt.plot([], lw=1.6, ds='steps', sketch_params=1)
set_colors()

def animate(i):
    """ Mise à jour des frames de l’animation."""
    global rec, inter
    pi_mc = get_new_pi()
    x_vals.append(i + 2)
    y_vals.append(pi_mc)
    delta = ampli/np.sqrt(nb_total) 
    plt.ylim(np.pi - delta, np.pi + delta)
    plt.xlim(i - 49, i + 51)
    line.set_data(x_vals, y_vals)
    nb_dec = count_dec(pi_mc)
    data = str_board.format(nb_dec, pi_mc, nb_total, nb_cible)
    board_txt.set_text(data) 
    
    if nb_dec > rec:   # record du nombre de décimales
        rec = nb_dec
        top.set_text(data)
        if nb_dec >= ndec:
            set_colors('rr', 'r')
            if not has_bug:
                plt.pause(1.2)
    else:
        set_colors()
        
    if nb_total == stop: # arrêt de l’animation si demandé
        set_colors(('gray', 'gray'), 'gray')
        ani.event_source.stop()


ani = FuncAnimation(plt.gcf(), animate, interval=inter, save_count=100)
plt.show()

# Si sauvegarde - save_count = Nb de frames - 1
#               - commenter (ou supprimer) plt.show()

#ani.save('plotting_pi.mp4')
#ani.save('plotting_pi.gif')
#plt.close(fig)

```
<br>
