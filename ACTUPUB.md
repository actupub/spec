# ActuPub

> Protocole ouvert de syndication d'articles fédérée. Format JSON Feed 1.1 étendu, conçu pour la communication instance-à-instance et la consommation directe par des clients lecteurs.

---

## Table des matières

1. [Ce qu'est ActuPub, et ce qu'il n'est pas](#1-ce-quest-actupub-et-ce-quil-nest-pas)
2. [Vocabulaire](#2-vocabulaire)
3. [Format de base : JSON Feed 1.1 + extension `_actupub`](#3-format-de-base--json-feed-11--extension-_actupub)
4. [Schéma des objets](#4-schéma-des-objets)
5. [Couche 1 : Découverte](#5-couche-1--découverte)
6. [Couche 2 : Flux et administration](#6-couche-2--flux-et-administration)
7. [Couche 3 : Flux temps réel (SSE, optionnel)](#7-couche-3--flux-temps-réel-sse-optionnel)
8. [Déduplication et référence officielle](#8-déduplication-et-référence-officielle)
9. [Confiance, validation humaine et publication](#9-confiance-validation-humaine-et-publication)
10. [Catégories](#10-catégories)
11. [Embeddings (optionnel)](#11-embeddings-optionnel)
12. [La règle NSFW](#12-la-règle-nsfw)
13. [Hors scope v1](#13-hors-scope-v1)
14. [Fondements techniques](#14-fondements-techniques)
15. [Ce qui reste à spécifier](#15-ce-qui-reste-à-spécifier)

---

## 1. Ce qu'est ActuPub, et ce qu'il n'est pas

### Ce qu'ActuPub est

ActuPub est un protocole de syndication d'articles conçu pour deux usages simultanés :

- **Instance → instance** : une instance peut s'abonner au flux d'une autre instance, avec des filtres fins (étiquettes, langue, densité, NSFW), ingérer les références dans sa propre base, et les repartager dans son propre flux, sous réserve de validation humaine ou automatique selon le niveau de confiance configuré.
- **Instance → lecteur** : un lecteur peut s'abonner directement à un flux ActuPub, avec des paramètres de filtre bien plus expressifs qu'un flux RSS ou Atom classique.

ActuPub est pensé comme un **successeur naturel de RSS et Atom**, ajoutant la fédération inter-instances, des métadonnées structurées, un mécanisme de validation humaine, et des filtres paramétrables, tout en restant lisible par les clients JSON Feed 1.1 existants.

### Ce qu'ActuPub n'est pas

- **Pas un réseau social** : pas d'interactions (like, commentaire, follow au sens ActivityPub). ActuPub ne transporte que des références d'articles et leurs métadonnées.
- **Pas un protocole push** : une instance ne pousse jamais de références à une autre de manière spontanée. Tout échange de flux est initié par une requête du consommateur (sauf le mécanisme de correction de référence officielle, voir section 8).

### Comment les instances interagissent

```
Instance B (consommatrice)               Instance A (source)
        |                                        |
        |-- GET /.well-known/actupub ----------->|  Découverte
        |<-- {capacités, endpoints} -------------|
        |                                        |
        |-- GET /actupub/tags ------------------>|  Récupération taxonomie
        |<-- {hiérarchie d'étiquettes} ----------|
        |                                        |
        |-- GET /actupub/feed?tags=ia&lang=fr -->|  Pull filtré
        |<-- {liste de références JSON Feed} ----|
        |                                        |
        |-- GET /actupub/stream (SSE) ---------->|  Temps réel (optionnel)
        |<-- event: create / update / delete ----|
```

Instance B ne reçoit que ce qu'elle demande. Instance A ne contacte jamais B spontanément, sauf pour signaler l'existence d'une référence officielle (section 8).

---

## 2. Vocabulaire

**Article** : contenu publié sur le web par un auteur ou une organisation sur leur propre site. Le flux ActuPub ne transporte pas cet article, uniquement une référence vers lui. ActuPub prévoit en revanche le transport du contenu de l'article via une API dédiée (voir section 6).

**Référence** (ou référence ActuPub) : objet JSON créé par une instance ActuPub qui pointe vers un article et en décrit les métadonnées (étiquettes, densité, NSFW, etc.). Plusieurs instances peuvent créer une référence pour le même article.

**Référence officielle** : référence créée par l'instance ActuPub gérée par l'auteur de l'article lui-même. Elle fait autorité sur les métadonnées d'origine.

**Référence curatée** : référence créée par une instance tierce qui recommande l'article sans en être l'auteur.

**Instance** : serveur qui implémente le protocole ActuPub. Peut être à la fois source pour ses abonnés et consommatrice d'autres instances.

**Instance source** : instance depuis laquelle une autre instance consommatrice tire des références via fédération.

**Instance consommatrice** : instance qui s'abonne aux flux d'une ou plusieurs instances sources.

**Instance curative** : instance qui ne fait que repartager des références vers des articles publiés ailleurs, sans héberger de contenu propre.

**Client** (ou lecteur) : application ou personne qui consomme un flux ActuPub en lecture seule, sans capacité de fédération.

**Fédération** : mécanisme par lequel une instance consommatrice s'abonne automatiquement au flux d'une instance source et ingère ses références dans sa propre base.

**File de validation** : file d'attente dans laquelle les références entrantes par fédération sont placées si leur score calculé est inférieur au seuil de publication automatique. L'opérateur doit les examiner dans le délai imparti.

---

## 3. Format de base : JSON Feed 1.1 + extension `_actupub`

JSON Feed 1.1 est un format de syndication JSON établi, disposant nativement des champs nécessaires : `id`, `url`, `title`, `summary`, `content_html`, `image`, `date_published`, `language`, `tags`, `authors`. Il prévoit un mécanisme d'extension explicite : tout champ préfixé par `_` est ignoré par les clients qui ne le reconnaissent pas.

**Conséquence directe** : un flux ActuPub est lisible immédiatement par les clients JSON Feed existants. Ils lisent les champs standard et ignorent `_actupub`. Un client ActuPub natif lit tout.

ActuPub n'est **pas** une implémentation de JSON Feed. C'est un protocole qui utilise JSON Feed comme format de sérialisation et l'étend via le mécanisme prévu à cet effet.

### MIME types

| Contexte | MIME type |
|---|---|
| Clients JSON Feed standard | `application/feed+json` |
| Clients ActuPub natifs | `application/actupub+json` |

Un client indique sa préférence via le header `Accept`. Si le serveur reçoit `application/actupub+json`, il peut inclure des métadonnées étendues non-JSON-Feed. Dans le cas contraire, il reste dans les limites JSON Feed standard.

---

## 4. Schéma des objets

### Objet Feed (enveloppe du flux)

L'objet Feed est le document racine retourné par `/actupub/feed`. Il décrit l'instance qui expose le flux et sert d'enveloppe pour la liste des références (items). Un client JSON Feed standard l'utilise pour afficher le nom du flux et accéder aux articles. Un client ActuPub l'utilise également pour localiser le point d'entrée de découverte de l'instance.

```json
{
  "version": "https://jsonfeed.org/version/1.1",
  "title": "Nom du blog ou de l'instance",
  "home_page_url": "https://example.com",
  "feed_url": "https://example.com/actupub/feed",
  "language": "fr",
  "_actupub": {
    "version": "1.0",
    "discovery_url": "https://example.com/.well-known/actupub"
  },
  "items": [ ]
}
```

| Champ | Description |
|---|---|
| `version` | Version de la spécification JSON Feed utilisée |
| `title` | Nom lisible de l'instance ou du blog |
| `home_page_url` | URL de la page d'accueil du blog ou de l'instance |
| `feed_url` | URL canonique de ce flux (self-referential). Peut inclure les paramètres de filtre appliqués pour cette réponse. |
| `language` | Langue principale du flux (code ISO 639-1) |
| `_actupub.version` | Version du protocole ActuPub |
| `_actupub.discovery_url` | Raccourci vers `/.well-known/actupub` de l'instance |
| `items` | Liste des références (objets Article, voir ci-dessous) |

### Objet Article (item / la référence)

Chaque item de la liste `items` est une référence ActuPub : un pointeur enrichi vers un article publié ailleurs (ou sur l'instance elle-même). Les étiquettes exposées dans une référence sont **toujours celles de l'instance qui expose le flux**, appliquées via son propre mapping. Une instance ne transmet jamais une référence avec des étiquettes qu'elle n'a pas définies elle-même.

```json
{
  "id": "https://example.com/mon-article",
  "url": "https://example.com/mon-article",
  "title": "Titre de l'article",
  "summary": "Résumé ou description de l'article en quelques phrases.",
  "image": "https://example.com/images/article-cover.jpg",
  "date_published": "2026-04-01T10:00:00Z",
  "date_modified": "2026-04-02T08:00:00Z",
  "language": "fr",
  "authors": [
    { "name": "Jean Dupont", "url": "https://example.com/auteur" }
  ],
  "tags": ["intelligence-artificielle", "deep-learning"],
  "_actupub": {
    "version": "1.0",
    "content_type": "article",
    "nsfw": false,
    "acturef_by_original_author": false,
    "source_domain": "example.com",
    "origin_instance": "https://instance-curatrice.example.com",
    "article_score": 0.5,
    "density_score": 3.7,
    "origin_tags": ["llm", "computer-vision"]
  }
}
```

### Champs JSON Feed standard

| Champ | Description |
|---|---|
| `id` | URL canonique nettoyée de l'article sur son site de publication d'origine. Identifiant universel immuable dans tout le réseau ActuPub. Les paramètres de tracking (UTM, etc.) doivent être retirés. |
| `url` | URL de l'article, identique à `id` dans la grande majorité des cas |
| `title` | Titre de l'article |
| `summary` | Résumé ou description de l'article. Champ natif JSON Feed 1.1. |
| `image` | URL de l'image principale ou de couverture de l'article. Champ natif JSON Feed 1.1. |
| `date_published` | Date de publication originale (ISO 8601) |
| `date_modified` | Date de dernière modification (ISO 8601) |
| `language` | Langue de l'article (code ISO 639-1) |
| `authors` | Auteurs de l'article. Champ natif JSON Feed 1.1. |
| `tags` | **Étiquettes locales de l'instance** qui expose ce flux. C'est cette liste que les clients utilisent pour le filtrage. Elle reflète la taxonomie de l'instance, pas nécessairement celle de la source. |

### Champs de l'extension `_actupub`

| Champ | Description |
|---|---|
| `content_type` | Type de contenu référencé. Valeurs définies par le protocole (voir liste exhaustive ci-dessous). |
| `nsfw` | Annotation NSFW obligatoire. `true` si le contenu n'est pas adapté à un contexte professionnel ou à un public mineur. |
| `acturef_by_original_author` | `true` si l'instance qui a créé cette référence est l'auteur de l'article. `false` si c'est une instance tierce qui a créé la référence originale. |
| `source_domain` | Domaine du site qui a publié l'article d'origine |
| `origin_instance` | Instance ActuPub qui a créé cette référence. Informatif, non utilisé pour la déduplication. |
| `article_score` | Note attribuée à cet article par l'opérateur de l'instance (0–1). Défaut : 0.5. Fixée manuellement. |
| `density_score` | Score de densité informationnelle de l'article (1–5). En l'absence d'un algorithme standardisé, cette valeur est fixée manuellement et subjectivement par l'opérateur. Défaut : `3` (valeur neutre au milieu de l'échelle). |
| `origin_tags` | Étiquettes telles que définies par l'instance source, avant mapping local. Conservées à titre informatif pour ne pas perdre les métadonnées d'origine. Absent si la source n'expose pas d'étiquettes. |

### Valeurs définies pour `content_type`

Le protocole définit la liste exhaustive suivante. Un client qui reçoit une valeur inconnue la traite comme `article` :

| Valeur | Description |
|---|---|
| `article` | Article de blog ou de presse (défaut) |
| `youtube-video` | Vidéo publiée sur YouTube |
| `instagram-post` | Publication Instagram |
| `x-post` | Publication sur X (ex-Twitter) |
| `podcast-episode` | Épisode de podcast |
| `newsletter` | Édition d'une newsletter |

---

## 5. Couche 1 : Découverte

### `GET /.well-known/actupub`

Point d'entrée universel de l'instance. Répond sans authentification.

```json
{
  "actupub_version": "1.0",
  "instance_name": "Nom du blog",
  "instance_url": "https://example.com",
  "feed_url": "https://example.com/actupub/feed",
  "tags_url": "https://example.com/actupub/tags",
  "stream_url": "https://example.com/actupub/stream",
  "capabilities": ["feed", "tags", "stream", "content", "embeddings"]
}
```

| Champ | Description |
|---|---|
| `actupub_version` | Version du protocole ActuPub implémentée |
| `instance_name` | Nom lisible de l'instance |
| `instance_url` | URL de base de l'instance |
| `feed_url` | Endpoint du flux principal |
| `tags_url` | Endpoint de la taxonomie d'étiquettes |
| `stream_url` | Endpoint SSE temps réel (si supporté) |
| `capabilities` | Fonctionnalités optionnelles supportées : `stream` (SSE), `content` (API de contenu), `embeddings` (endpoint embedding) |

### `GET /actupub/tags`

Expose la taxonomie complète de l'instance. Utile pour une instance consommatrice qui souhaite construire sa table de correspondance entre ses étiquettes locales et celles de la source. Les synonymes jouent un rôle clé dans ce mapping : si une étiquette distante correspond à un synonyme d'une étiquette locale, le mapping peut être établi automatiquement sans intervention humaine.

```json
{
  "tags": [
    {
      "id": "intelligence-artificielle",
      "name": "Intelligence artificielle",
      "synonyms": ["IA", "AI", "Artificial Intelligence"],
      "description": "Articles portant sur les systèmes d'intelligence artificielle...",
      "parent_id": null,
      "translations": {
        "en": {
          "name": "Artificial intelligence",
          "synonyms": ["AI", "machine intelligence"]
        },
        "de": {
          "name": "Künstliche Intelligenz",
          "synonyms": ["KI"]
        }
      }
    },
    {
      "id": "deep-learning",
      "name": "Deep learning",
      "synonyms": ["DL", "apprentissage profond"],
      "description": "Sous-ensemble du machine learning basé sur des réseaux de neurones profonds.",
      "parent_id": "intelligence-artificielle"
    }
  ]
}
```

| Champ | Description |
|---|---|
| `id` | Slug stable et immuable, local à l'instance |
| `name` | Nom principal affiché (dans la langue principale de l'instance) |
| `synonyms` | Noms alternatifs utilisés pour le mapping automatique lors de la fédération, ainsi que pour l'autocomplétion dans les interfaces utilisateur |
| `description` | Définition précise de ce que l'étiquette couvre et ne couvre pas |
| `parent_id` | Slug de l'étiquette parente (`null` si étiquette racine). La hiérarchie se reconstruit en remontant les `parent_id` côté client. |
| `translations` | Optionnel. Traductions de l'étiquette dans d'autres langues, indexées par code ISO 639-1. Chaque entrée contient `name` (nom traduit) et `synonyms` (liste de synonymes traduits). Particulièrement utile pour le mapping automatique inter-instances de langues différentes. |

Note : le champ `children` n'est pas exposé. La hiérarchie est toujours reconstruite à partir des `parent_id`, évitant toute redondance et incohérence dans les données.

Lorsqu'une instance consommatrice rencontre une étiquette distante inconnue, elle doit récupérer la liste complète et à jour des étiquettes de la source via cet endpoint (et non un endpoint par étiquette individuelle), car si une étiquette est inconnue, la taxonomie source a probablement évolué dans son ensemble.

---

## 6. Couche 2 : Flux et administration

### Endpoint public : `GET /actupub/feed`

Retourne la liste des références au format JSON Feed 1.1 + `_actupub`, filtrées par les paramètres passés en query string.

#### Paramètres de filtrage

| Paramètre | Type | Description | Exemple |
|---|---|---|---|
| `include_tags` | `string[]` | N'inclure que les articles portant au moins une de ces étiquettes locales | `include_tags=ia,deep-learning` |
| `exclude_tags` | `string[]` | Exclure les articles portant au moins une de ces étiquettes | `exclude_tags=politique,sport` |
| `lang` | `string` | Langue de l'article (code ISO 639-1) | `lang=fr` |
| `nsfw` | `boolean` | Inclure (`true`) ou exclure (`false`) les articles NSFW. Défaut : `false` | `nsfw=false` |
| `min_density` | `float` | Score de densité minimum (1–5) | `min_density=3` |
| `max_density` | `float` | Score de densité maximum (1–5) | `max_density=5` |
| `min_score` | `float` | Score d'article minimum (0–1) | `min_score=0.6` |
| `after` | `datetime` | N'inclure que les articles publiés après cette date (ISO 8601) | `after=2026-01-01T00:00:00Z` |
| `before` | `datetime` | N'inclure que les articles publiés avant cette date | `before=2026-04-01T00:00:00Z` |
| `acturef_by_original_author` | `boolean` | Si `true`, n'inclure que les références créées par l'auteur de l'article | `acturef_by_original_author=true` |
| `authored_only` | `boolean` | Si `true`, n'inclure que les articles dont l'auteur est le gestionnaire du blog hébergeant cette instance (articles propres, indépendamment de qui a créé la référence ActuPub) | `authored_only=true` |
| `content_type` | `string` | Filtrer par type de contenu | `content_type=article` |
| `limit` | `integer` | Nombre maximum de résultats par page. Défaut : 50, max : 200 | `limit=20` |
| `cursor` | `string` | Jeton de pagination retourné par la réponse précédente (voir note ci-dessous) | `cursor=eyJpZCI6NDJ9` |
| `include_embedding` | `boolean` | Inclure les embeddings inline dans chaque article. Défaut : `false` | `include_embedding=false` |

**Note sur la pagination** : le `cursor` est une valeur opaque retournée dans le champ `_actupub.next_cursor` de la réponse. Le client le transmet tel quel dans la requête suivante, sans l'interpréter ni le modifier. Il encode un état interne du serveur (par exemple, le dernier identifiant vu), garantissant la cohérence de la pagination même si de nouveaux articles sont ajoutés entre deux requêtes. Si `next_cursor` est absent ou `null`, il n'y a plus de page suivante.

**Note sur la confidentialité** : les paramètres de filtre étant passés en query string, ils sont visibles par le serveur. Un client soucieux de sa vie privée peut récupérer le flux sans paramètres et filtrer localement, au prix d'un traitement plus lourd côté client. Cette approche est entièrement compatible avec le protocole.

Exemple d'URL de flux :
```
https://example.com/actupub/feed?include_tags=ia,deep-learning&lang=fr&nsfw=false&min_density=3&limit=20
```

### Endpoint article individuel : `GET /actupub/article/{encoded-id}`

Retourne la référence complète d'un article spécifique, identifié par son `id` encodé en base64url. Utilisé notamment par le mécanisme de découverte de référence officielle (section 8).

```
GET /actupub/article/aHR0cHM6Ly9leGFtcGxlLmNvbS9tb24tYXJ0aWNsZQ==
```

### API de contenu : `GET /actupub/content/{encoded-id}`

Permet à un client de récupérer le contenu complet d'un article directement via ActuPub. Cet endpoint fait partie du protocole, mais son implémentation est **réservée aux instances hébergeant du contenu propre** (blogs, médias). Une instance curative qui ne fait que repartager des références n'a pas le droit de transmettre le contenu d'articles qu'elle ne publie pas. Elle peut fermer cet endpoint (il n'apparaît alors pas dans ses `capabilities`) ou retourner une redirection vers la page d'origine :

```json
{ "redirect": "https://url-originale.com/article" }
```

Quand un client souhaite lire le contenu d'un article, il s'adresse **directement au site qui a publié l'article** — identifiable via le `source_domain` et le `id` de la référence. L'instance intermédiaire ne relaie **jamais** le contenu.

**Requête pour un article public :**
```
GET /actupub/content/aHR0cHM6Ly9leGFtcGxlLmNvbS9tb24tYXJ0aWNsZQ==
```

**Requête pour un article payant (authentifiée) :**
```
GET /actupub/content/aHR0cHM6Ly9leGFtcGxlLmNvbS9tb24tYXJ0aWNsZQ==
Authorization: Bearer {jwt-token}
```

**Réponse :**
```json
{
  "id": "https://example.com/mon-article",
  "content_html": "<p>Contenu complet de l'article...</p>"
}
```

**Authentification pour les contenus payants : OAuth 2.0 + JWT Bearer**

Pour les articles sous paywall, le site publieur protège cet endpoint avec un JWT Bearer. L'utilisateur s'authentifie sur le site du média via son propre système OAuth 2.0 (Authorization Code Flow), obtient un JWT d'accès, et le transmet dans le header `Authorization` de la requête ActuPub. Le média valide son propre token selon ses propres règles. ActuPub ne prescrit pas le serveur d'autorisation ni la gestion des comptes, qui restent entièrement sous le contrôle du publieur.

Ce choix est motivé par le fait que les grands médias utilisent déjà OAuth 2.0 pour leurs systèmes d'abonnement. Implémenter cet endpoint ne leur demande que d'exposer une nouvelle route protégée par leur infrastructure d'authentification existante, sans nouvel acteur dans la chaîne.

### Endpoints d'administration (authentifiés)

Ces endpoints permettent à l'opérateur de gérer sa base de références. L'authentification utilise le même mécanisme que pour l'API de contenu : **OAuth 2.0 + JWT Bearer**. Le scope requis est `admin`, distinct du scope `read:content` utilisé pour la lecture d'articles payants.

**`POST /actupub/admin/articles`** : Ajouter une référence manuellement

Avant de remplir le formulaire de création, le client doit vérifier si une référence officielle existe déjà pour l'URL saisie (via le mécanisme de découverte décrit en section 8) et pré-remplir les champs disponibles. L'opérateur peut ensuite modifier ou compléter les métadonnées avant soumission.

Corps de la requête (tous les champs configurables) :

```json
{
  "url": "https://example.com/mon-article",
  "title": "Titre de l'article",
  "summary": "Résumé de l'article.",
  "image": "https://example.com/cover.jpg",
  "content_type": "article",
  "language": "fr",
  "authors": [{ "name": "Jean Dupont", "url": "https://example.com/auteur" }],
  "date_published": "2026-04-01T10:00:00Z",
  "tags": ["intelligence-artificielle", "deep-learning"],
  "nsfw": false,
  "article_score": 0.8,
  "acturef_by_original_author": false
}
```

| Champ | Requis | Description |
|---|---|---|
| `url` | Oui | URL canonique de l'article. L'instance en déduit l'`id` après nettoyage. |
| `title` | Non | Titre de l'article. Récupéré automatiquement depuis la page si absent. |
| `summary` | Non | Résumé ou description de l'article |
| `image` | Non | URL de l'image de couverture |
| `content_type` | Non | Type de contenu. Défaut : `article` |
| `language` | Non | Langue de l'article (ISO 639-1) |
| `authors` | Non | Auteurs de l'article |
| `date_published` | Non | Date de publication. Récupérée automatiquement si absente. |
| `tags` | Non | Étiquettes locales à associer |
| `nsfw` | Oui | Annotation NSFW obligatoire |
| `article_score` | Non | Note manuelle (0–1). Défaut : 0.5 |
| `acturef_by_original_author` | Non | Défaut : `false` |

Un article ajouté manuellement est publié directement dans le flux, sans passer par la file de validation.

**`PATCH /actupub/admin/articles/{encoded-id}`** : Modifier une référence existante

La requête suit le même format que `POST`, en n'incluant que les champs à modifier (patch partiel). Tous les champs sont optionnels.

```json
{
  "tags": ["intelligence-artificielle", "nlp"],
  "article_score": 0.9,
  "nsfw": false
}
```

**`DELETE /actupub/admin/articles/{encoded-id}`** : Supprimer une référence

Supprime la référence de la base locale. L'article n'apparaît plus dans le flux. Cet événement est propagé aux instances abonnées via SSE si la couche 3 est active.

---

## 7. Couche 3 : Flux temps réel (SSE, optionnel)

**Cette couche est optionnelle et désactivée par défaut.** Un pull régulier (toutes les 15 minutes à 1 heure selon la fréquence de publication de la source) est suffisant dans la grande majorité des cas d'usage. Elle n'est pas prioritaire pour la v1 et sera spécifiée en détail dans une version ultérieure.

Une instance qui l'implémente expose un endpoint SSE (Server-Sent Events) sur lequel les instances abonnées peuvent recevoir les événements `create`, `update` et `delete` en temps réel.

---

## 8. Déduplication et référence officielle

### Déduplication par identifiant

L'identifiant universel d'un article est son `id` : l'URL canonique nettoyée du site de publication d'origine. Si une instance reçoit deux références avec le même `id`, elle les traite comme la même. Le comportement de conservation par défaut est :
1. Si l'une des références est officielle (`acturef_by_original_author: true`) et l'autre curatée, conserver la référence officielle.
2. Sinon, conserver la plus récente ou la plus complète.

### Boucles de fédération

Si l'instance A suit l'instance B qui suit l'instance A, un article créé sur A arrive sur B, qui tente de le renvoyer à A. La déduplication par `id` arrête la boucle : A possède déjà cet article et l'ignore. Pour que ce mécanisme fonctionne, l'`id` doit toujours être l'URL d'origine de l'article sur son site de publication — jamais un identifiant interne à une instance.

### Mécanisme de découverte à la création d'une référence

Quand une instance crée une référence pour un article (URL `https://example.com/article`), elle doit d'abord vérifier :

1. Le site source expose-t-il `/.well-known/actupub` ? Si oui, chercher une référence existante pour cet `id` via `/actupub/article/{encoded-id}`. Si trouvée → récupérer la référence officielle plutôt qu'en créer une nouvelle.
2. Les instances connues ont-elles déjà une référence pour cet `id` ? → Même démarche.

Si aucune référence n'existe, l'instance crée la sienne localement. Elle ne la partage pas activement : elle sera découverte naturellement par les instances qui la demanderont.

### Mécanisme de correction

Si une instance découvre qu'elle possède une référence curatée alors qu'une référence officielle existe pour le même `id`, elle notifie l'instance à la source de la référence curatée de l'existence de la référence officielle :

```
POST /actupub/admin/corrections
```
```json
{
  "type": "reference_correction",
  "article_id": "https://example.com/article-original",
  "official_reference_url": "https://blog-auteur.com/actupub/article/aHR0cHM6Ly9ibG9n..."
}
```

L'instance réceptrice peut alors remplacer sa référence curatée par la référence officielle. Ce mécanisme est best-effort : une référence curatée qui circule quelque temps avant correction n'est pas un problème grave.

### Priorité des étiquettes

Les étiquettes exposées dans le flux d'une instance sont toujours celles définies par l'instance elle-même via son propre mapping. Les étiquettes de l'instance source sont conservées dans `origin_tags` à titre informatif, mais ne sont jamais utilisées directement par les clients pour le filtrage.

---

## 9. Confiance, validation humaine et publication

Le protocole ActuPub intègre un mécanisme de validation humaine au cœur de la fédération, cohérent avec la valeur fondatrice du projet : **le partage d'information doit rester un acte humain et délibéré**.

### Poids de confiance

Chaque source fédérée se configure avec les paramètres suivants, définis par l'opérateur de l'instance consommatrice dans un fichier de configuration local à l'instance :

**Poids de l'instance source** (`instance_trust`) : confiance globale accordée à une instance source, entre 0 et 1. Défaut : `0.5`.

**Poids par catégorie** (`weight` dans le mapping) : confiance accordée aux articles portant une étiquette donnée, entre 0 et 1. Défaut : `0.5`.

Le fichier suivant est un exemple de configuration locale pour une source fédérée :

```json
{
  "source_url": "https://assoc.example.com",
  "instance_trust": 0.8,
  "tag_mapping": [
    { "remote": "llm", "local": "modeles-de-langage", "weight": 0.9 },
    { "remote": "politique", "local": "politique", "weight": 0.3 },
    { "remote": "computer-vision", "local": "vision-par-ordinateur", "weight": 0.7 }
  ],
  "unmapped_behavior": "ignore",
  "auto_publish_threshold": 0.75,
  "review_timeout": 3600
}
```

| Champ | Description |
|---|---|
| `instance_trust` | Confiance globale dans la source (0–1, défaut 0.5) |
| `tag_mapping[].weight` | Confiance dans les articles de cette catégorie (0–1, défaut 0.5) |
| `unmapped_behavior` | Comportement pour étiquettes sans correspondance locale (voir section 10) |
| `auto_publish_threshold` | Seuil de publication automatique (0–1, défaut 0.75) |
| `review_timeout` | Délai en secondes avant suppression d'une référence non traitée (défaut : 3600 = 1h) |

### Formule de scoring

À chaque réception d'une nouvelle référence par fédération, l'instance consommatrice calcule un score :

```
matched_weights = poids des étiquettes locales matchées par le mapping
computed_score = (instance_trust + moyenne(matched_weights)) / 2
```

Si aucune étiquette ne matche (et que `unmapped_behavior != "ignore"`) :
```
computed_score = instance_trust
```

Avec les valeurs par défaut (instance_trust = 0.5, tous les tag weights à 0.5), `computed_score = 0.5`, inférieur au seuil par défaut (0.75). Le comportement par défaut est donc prudent : les nouvelles sources passent en file de validation jusqu'à ce que l'opérateur ajuste les poids.

### File de validation et publication automatique

```
Réception d'une référence fédérée
        ↓
computed_score >= auto_publish_threshold ?
        ↓ OUI                    ↓ NON
Publication immédiate     Entrée en file de validation
dans le flux                     ↓
                   L'opérateur peut : approuver,
                   rejeter, ou modifier des métadonnées
                                 ↓
                   review_timeout expiré sans action ?
                        ↓ OUI        ↓ NON
                   Suppression   Action de l'opérateur
                   référence     appliquée
                   locale
```

Les articles ajoutés **manuellement** par l'opérateur sont publiés directement sans passer par la file de validation.

### Note `article_score`

Le champ `article_score` exposé dans le flux est la note manuelle fixée par l'opérateur, indépendante du `computed_score` qui sert uniquement au mécanisme de validation interne. Le `computed_score` n'est pas exposé dans le flux. Les clients peuvent utiliser `article_score` pour trier ou filtrer les articles via le paramètre `min_score`.

---

## 10. Catégories

### Liste de base

Le protocole ActuPub fournit une liste de catégories standard avec leurs descriptions, dans un document séparé (`TAGS.md`). Cette liste est un point de départ, pas une contrainte. Les implémentations sont encouragées à l'importer comme base et à l'étendre selon leurs besoins.

### Partage inter-instances

Quand une instance rencontre une étiquette distante inconnue lors de la fédération, elle récupère la liste complète et à jour des étiquettes de la source via `GET /actupub/tags`, puis :

1. Tente un **mapping automatique par synonymes** : si le nom ou un synonyme de l'étiquette distante correspond à une étiquette locale, le mapping est établi.
2. Si les synonymes ne suffisent pas et que les embeddings sont activés : tente un **mapping automatique par similarité d'embedding** entre la description de l'étiquette distante et les descriptions des étiquettes locales, puis retente le mapping.
3. Sinon, applique le comportement défini par `unmapped_behavior`.

Le champ `unmapped_behavior` dans la configuration source définit le comportement par défaut pour les étiquettes sans correspondance :
- `ignore` : les articles non mappés ne sont pas ingérés
- `passthrough` : les articles sont ingérés sans étiquette locale
- `tag:{slug}` : les articles reçoivent une étiquette générique
- `ai` : mapping automatique par similarité d'embedding (nécessite que les embeddings soient activés)

Note : `unmapped_behavior` ne s'applique qu'aux articles ingérés par fédération automatique. Pour les articles ajoutés manuellement, l'opérateur vérifie et choisit lui-même les étiquettes.

### Recommandation : positions politiques

Le protocole **recommande fortement** que les articles exprimant une position politique identifiable soient étiquetés en conséquence (ex. `anarchisme`, `nationalisme`, `libéralisme`, `écologie-politique`, etc.). Cette transparence permet aux lecteurs de filtrer explicitement selon leurs préférences ou de s'exposer volontairement à des points de vue variés. Cette recommandation n'est pas contraignante, mais elle est inscrite dans l'esprit du protocole.

---

## 11. Embeddings (optionnel)

### Utilité

Les embeddings servent deux usages dans ActuPub :

1. **Étiquettes** : mesure de similarité sémantique entre étiquettes d'instances différentes, pour générer automatiquement des mappings lors de la fédération.
2. **Articles** : filtrage sémantique pour diversifier ou restreindre les articles, détecter les doublons sémantiques même si les `id` sont différents, regrouper des sujets connexes.

### Modèle standard

Le protocole spécifie un modèle de référence unique pour garantir l'interopérabilité des embeddings entre instances :

**`sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`**

- 384 dimensions (environ 1,5 Ko par vecteur en float32)
- Multilingue nativement (50+ langues)
- Optimisé pour l'inférence CPU (ne nécessite pas de GPU)
- Open source, déployable localement sans API externe

Les embeddings produits par ce modèle sont directement comparables entre instances, condition nécessaire au mapping automatique inter-instances.

### Endpoints

**`GET /actupub/article/{encoded-id}/embedding`** : Embedding d'un article

```json
{
  "id": "https://example.com/mon-article",
  "model": "paraphrase-multilingual-MiniLM-L12-v2",
  "dimensions": 384,
  "embedding": [0.021, -0.154, 0.302, "..."]
}
```

**`GET /actupub/tags/{tag-id}/embedding`** : Embedding d'une étiquette

```json
{
  "id": "intelligence-artificielle",
  "model": "paraphrase-multilingual-MiniLM-L12-v2",
  "dimensions": 384,
  "embedding": [0.045, 0.198, -0.087, "..."]
}
```

Les embeddings ne sont pas inclus inline dans les réponses du feed par défaut. L'endpoint séparé est l'approche recommandée. Ces capacités sont déclarées dans `/.well-known/actupub` sous `"capabilities": ["embeddings"]`.

---

## 12. La règle NSFW

Le champ `_actupub.nsfw` est **obligatoire** dans tout objet article. Une instance qui crée une référence doit l'annoter. Une instance qui reçoit un article avec `nsfw: true` doit répercuter cette valeur sans l'écraser, quelle que soit sa configuration locale.

---

## 13. Fondements techniques

| Aspect | Standard réutilisé | Usage dans ActuPub |
|---|---|---|
| Format de sérialisation | JSON Feed 1.1 | Base + extension `_actupub` |
| Mécanisme d'extension | Préfixe `_` de JSON Feed 1.1 | Champs `_actupub` ignorés par clients standard |
| Transport | HTTP/HTTPS | Universel |
| Flux temps réel | SSE -- W3C Server-Sent Events | Couche 3 (optionnelle, version future) |
| Découverte | RFC 8615 (`/.well-known/`) | Endpoint `/.well-known/actupub` |
| Identifiants globaux | URLs canoniques | `id` = URL nettoyée de l'article source |
| Auth contenu payant | OAuth 2.0 + JWT Bearer | Authorization Code Flow côté publieur |
| Embeddings | MiniLM-L12-v2 (sentence-transformers) | Similarité sémantique inter-instances |
| **Schéma `_actupub`** | **Aucun** | **Création propre au protocole** |

---

## 14. Ce qui reste à spécifier

**À traiter en priorité avant implémentation :**

- Format exact et règles de nettoyage de l'`id` (suppression des paramètres UTM, normalisation des URLs, gestion des redirections)
- Format du curseur de pagination et comportement en cas de curseur expiré
- Mécanisme d'authentification pour les endpoints d'administration (Bearer token, spécification complète)
- Format complet du document `TAGS.md` (liste de base de catégories avec descriptions)
- Algorithme de calcul du `density_score` : définir et inclure dans le protocole une méthode de calcul standardisée (richesse lexicale, densité conceptuelle, longueur, etc.) pour remplacer la saisie manuelle actuelle et rendre les scores comparables entre instances
- Étudier la possibilité de déduplication par comparaison d'embeddings, en complément de la déduplication par `id`, pour les cas où l'URL d'un article est mal encodée ou redirigée dans une référence

**À traiter en version future :**

- Spécification complète de la couche 3 (SSE)
- Versioning du protocole et négociation de version entre instances
