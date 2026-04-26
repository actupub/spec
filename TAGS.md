# ActuPub — Liste de base des catégories

Ce document définit une liste de base des catégories recommandées par le protocole ActuPub. Chaque instance est libre de l'adopter telle quelle, de l'étendre, ou de définir sa propre taxonomie. Cette liste constitue un point de départ commun pour faciliter le mapping inter-instances.

Les catégories sont organisées en domaines thématiques, puis en sous-catégories. Chaque entrée précise l'identifiant (`id`), le nom principal, des synonymes courants, et une description de ce qu'elle couvre.

*Note : cette liste a été générée par une IA, elle sera relue et confirmée par un humain très rapidement.*

## Sciences et technologie

| id | Nom | Synonymes | Description |
|---|---|---|---|
| `sciences` | Sciences | Science | Articles de vulgarisation ou de recherche scientifique, toutes disciplines confondues |
| `informatique` | Informatique | Tech, Technologie, IT | Articles sur l'informatique au sens large |
| `intelligence-artificielle` | Intelligence artificielle | IA, AI, Artificial Intelligence | Systèmes capables de simuler des capacités cognitives humaines |
| `machine-learning` | Machine learning | Apprentissage automatique, ML | Sous-domaine de l'IA basé sur l'apprentissage à partir de données |
| `deep-learning` | Deep learning | Apprentissage profond, DL | Machine learning basé sur des réseaux de neurones à plusieurs couches |
| `traitement-langage-naturel` | Traitement du langage naturel | NLP, TAL | Traitement automatique du langage humain (texte, parole) |
| `cybersecurite` | Cybersécurité | Sécurité informatique, InfoSec | Protection des systèmes, réseaux et données |
| `logiciel-libre` | Logiciel libre | Open source, FOSS | Logiciels dont le code source est ouvert et librement modifiable |
| `developpement` | Développement | Dev, Programmation, Code | Articles sur le développement logiciel et les pratiques de programmation |
| `materiel` | Matériel | Hardware | Composants physiques : processeurs, cartes graphiques, électronique |
| `physique` | Physique | - | Articles de physique fondamentale ou appliquée |
| `biologie` | Biologie | - | Sciences du vivant |
| `mathematiques` | Mathématiques | Maths | Articles de mathématiques fondamentales ou appliquées |
| `espace` | Espace | Astronomie, Cosmologie | Exploration spatiale, astronomie, cosmologie |
| `environnement` | Environnement | Écologie, Nature | Sciences de l'environnement, biodiversité, changement climatique |
| `medecine` | Médecine | Santé, Health | Articles médicaux et de santé publique |

## Société et politique

| id | Nom | Synonymes | Description |
|---|---|---|---|
| `politique` | Politique | - | Articles sur la vie politique au sens large, sans positionnement particulier |
| `geopolitique` | Géopolitique | Relations internationales | Relations entre États, enjeux de puissance, diplomatie |
| `economie` | Économie | - | Articles sur les systèmes économiques, la finance, les marchés |
| `droit` | Droit | Justice, Légal | Articles juridiques, décisions de justice, réglementation |
| `education` | Éducation | - | Systèmes éducatifs, pédagogie, enseignement |
| `sante-publique` | Santé publique | - | Politiques de santé, épidémiologie, systèmes de soins |
| `migrations` | Migrations | Immigration | Mouvements de populations, politiques migratoires |
| `religions` | Religions | Spiritualité, Foi | Articles sur les religions et le fait religieux |

### Courants politiques

Ces catégories sont utilisées pour annoter les articles qui expriment explicitement une position politique identifiable. Leur usage est **fortement recommandé** par le protocole pour la transparence éditoriale.

| id | Nom | Synonymes | Description |
|---|---|---|---|
| `anarchisme` | Anarchisme | Anarchie | Courant politique visant l'abolition de toute autorité coercitive |
| `liberalisme` | Libéralisme | Libéral | Courant fondé sur la liberté individuelle et l'économie de marché |
| `socialisme` | Socialisme | Socialist | Courant visant la propriété collective des moyens de production |
| `conservatisme` | Conservatisme | Conservateur, Traditionnel | Courant attaché aux institutions et aux valeurs établies |
| `nationalisme` | Nationalisme | Nationaliste | Courant centré sur l'identité et la souveraineté nationale |
| `ecologie-politique` | Écologie politique | Vert, Green | Courant politique organisant la société autour des enjeux environnementaux |
| `feminisme` | Féminisme | - | Courant militant pour l'égalité des genres |
| `libertarisme` | Libertarisme | Libertarien | Courant prônant la liberté individuelle maximale contre l'État |

## Culture et société

| id | Nom | Synonymes | Description |
|---|---|---|---|
| `culture` | Culture | - | Articles culturels au sens large |
| `art` | Art | Arts | Beaux-arts, arts visuels, arts de la scène |
| `litterature` | Littérature | Livres, Lecture | Critique littéraire, romans, essais |
| `cinema` | Cinéma | Film | Articles sur le cinéma et les séries |
| `musique` | Musique | - | Articles sur la musique, les artistes, les sorties |
| `jeux-video` | Jeux vidéo | Gaming, Jeux | Articles sur l'industrie et la culture du jeu vidéo |
| `architecture` | Architecture | - | Architecture, urbanisme, design urbain |
| `design` | Design | - | Design graphique, industriel, d'interface |
| `gastronomie` | Gastronomie | Cuisine, Food | Arts culinaires, cuisine, critique gastronomique |
| `sport` | Sport | - | Articles sportifs au sens large |
| `histoire` | Histoire | - | Articles historiques, archéologie |
| `philosophie` | Philosophie | - | Articles de philosophie et d'éthique |
| `media` | Médias | Journalisme, Presse | Articles sur les médias et le journalisme |

## Économie et entreprise

| id | Nom | Synonymes | Description |
|---|---|---|---|
| `startups` | Startups | - | Articles sur les startups et l'écosystème entrepreneurial |
| `finance` | Finance | - | Marchés financiers, investissement, crypto-monnaies |
| `industrie` | Industrie | - | Secteurs industriels, production, supply chain |
| `travail` | Travail | Emploi, RH | Organisation du travail, emploi, droits des travailleurs |

## Notes sur l'utilisation

**Hiérarchie** : certaines catégories sont naturellement parentes d'autres (ex. `machine-learning` est enfant de `intelligence-artificielle`, qui est enfant d'`informatique`). La hiérarchie n'est pas imposée dans ce document, chaque instance la configure selon sa propre logique dans son endpoint `/actupub/tags`.

**Catégories manquantes** : cette liste est intentionnellement généraliste. Les instances spécialisées (médecine, droit, sciences, etc.) sont encouragées à définir des sous-catégories adaptées à leur domaine.

**Proposer une catégorie** : si une catégorie importante manque à cette liste, ouvrez une Discussion dans le dépôt `actupub/spec`.
