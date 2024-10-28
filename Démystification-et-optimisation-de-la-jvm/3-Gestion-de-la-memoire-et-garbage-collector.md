# Section 3 : GESTION DE LA MEMOIRE ET DU GARBAGE COLLECTOR

> Objectifs :
>
> - Comprendre la gestion de la mémoire par la JVM
> - Explorer le fonctionnement du garbage collector
> - Identifier et corriger un cas pratique de fuites de mémoires

## 1 - Présentation de la mémoire en JAVA

La mémoire en JAVA est gérée par la JVM. Elle est divisée en deux principales parties :

- **La Stack** : C'est l'espace ou sont stockés les variables locales, les objets de types primitifs(int, boolean, long...), les appels des méthodes. Sa taille est limitée et elle est gérée de manière LIFO (Last In First out).
- **La heap** : C'est l'espace ou sont stockées les instances des classes. Sa taille peut varier et elle est gérée par le garbage collector.

![JVM HEAP DESCRIPTION](/assets/heap.jpeg "JVM HEAP DESCRIPTION")Description de la Heap

La heap est subdivisée en deux espaces dits générationels :

- **La young generation** : Qui comprends plusieurs régions:
  - **l'eden** : Ou sont initialement crées les objets.
  - **les espaces survivants** **S0** et **S1** : deux zones où les objets qui ont survécu à un premier cycle de garbage collection sont déplacés.

L'idée est que la plupart des objets ont une durée de vie courte et sont rapidement collectés, ce qui rend la Young Generation assez dynamique.

- **La old generation** ou **tenured generation** : C'est l'endroit où les objets qui ont survécu à plusieurs cycles de garbage collection dans la Young Generation sont déplacés.
Les objets dans cette génération ont tendance à avoir une durée de vie plus longue, ce qui les rend moins susceptibles d'être collectés régulièrement.

Dans la section précédente, nous avons appris que les objets ont un certains poids en mémoire. Il est donc nécéssaire de gérer efficacement la mémoire disponible.
Deux exceptions peuvent être levées par la JVM en cas de manque d'espace mémoire :

- **StackOverflowError** : Lorsque la Stack est pleine ;
- **OutOfMemoryError** :  Lorsque la Heap est pleine.

## 2 - Présentation du garbage collector

Le garbage collector est le mécanisme responsable de la récupération de la mémoire inutilisée. Autrement dit, il est chargé de vérifier l’état de la Heap afin de supprimer les objets qui ne sont plus utilisés / plus référencés (c’est-à-dire que plus aucun autre objet ne pointe sur ces derniers), et ainsi libérer de l’espace mémoire pour les nouveaux objets à créer. Il en existe de plusieurs types, chacun adapté à des besoins différents.

### 2.1 - Quelques concepts du garbage collector

- **StopTheWorld** : Arrêt complet de tous les threads de l'application pendant la garbage collection.
- **Minor GC** : Lorsque la young generation est pleine, le garbage collector va exécuter un minor GC afin de supprimer les objets déréférencés et déplacer les survivants dans la old generation.
- **Major GC** : Lorsque la old generation est pleine, elle va à son tour être nettoyée. C'est ce qu'on appelle un major GC.
- **Full GC** : Il s'agit d'une garbage collection à la fois sur la young et la old generation.
- **Mark** : Etape de la garbage collection consistant à marquer les objets encore référencés.
- **Sweep** : Etape de la garbage collection consistant à la suppression des objets non référencés.
- **Compact** : Etape de la garbage collection consistant à compacter la mémoire en déplacant les objets de façon à les stocker de manière contigue.
- **Copy** : Etape de la garbage collection consistant à copier les objets vivants d'une région mémoire à une autre.

### 2.2 - Serial GC

Minor GC => Mark - Copy

Major GC => Mark - Sweep - Compact

Chaque étape est réalisée en stopTheWorld.

### 2.3 - Parallel GC

Minor GC => Mark - Copy

Major GC => Mark - Sweep - Compact

Chaque étape est réalisée en stopTheWorld et par plusieurs threads en parallèle.

### 2.4 - Garbage first GC (G1GC)

 Le G1GC est le plus performant des algorithmes de GC dans la mesure ou il est capable resizer les régions de la heap et d'effectuer des GC partiel. Par exemple, après un minor GC, il peut décider d'allouer un peu plus d'espace à l'eden de sorte à effectuer moins d'avoir des mineures moins fréquentes. Il est également capable de faire des major GC seulement sur une partie de la old generation ce qui permet de réduire la durée du stopTheWorld.
 Le G1GC est collecteur par défaut depuis Java 9.

## 3 - Les fuites de mémoires

On parle de fuites de mémoire lorsque des objets ne sont pas correctement libérés par le garbage collector. C'est à dire que des objets qui ne sont plus utiles à l'application persistent sur la heap. Les fuites de mémoires aboutissent généralement en *OutOfMemoryError*.
Le premier reflex face à cette exception pourrait etre d'augmenter la taille de la heap mais cela ne fera que retarder le moment ou l'erreur sera levée.
Pour corriger une fuite de mémoire, il faut tout d'abord en trouver l'origine :

- Heap dump: -XX:+HeapDumpOnOutOfMemoryError
- Profiling : VisualVM pour analyser le contenu de la mémoire

## 4 - Exercice pratique
