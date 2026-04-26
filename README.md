# ActuPub

ActuPub est un protocole de syndication d'articles conçu comme l'évolution naturelle de RSS, avec deux apports fondamentaux :

1. **Des métriques structurées** permettant de filtrer automatiquement les articles selon des critères fins et objectifs ;

2. **La fédération native** : la consommation et le repartage d'articles sont au cœur du protocole, pas une fonctionnalité.

> ⚠️ Brouillon : Ce document est en cours de conception et n'est pas encore prêt pour une implémentation. Des points restent à préciser. Si vous souhaitez contribuer à la suite (pour affiner la spec, implémenter une première version, ou simplement poser des questions) n'hésitez pas à me contacter ([mastodon](@cyberethiquefr@mastodon.tedomum.net), [Stoat (ex Revolt)](https://rvlt.gg/n4HQA97B)).

## Motivation

Voici les constats qui ont rendu ce projet nécessaire :

1. La dégradation sensible de la qualité de l'information sur les réseaux sociaux : la réaction et l'impulsion y priment sur la raison, la réflexion et l'argumentation. Le signal y est noyé dans le bruit.

2. La "merdification" des réseaux sociaux : les grandes plateformes sont conçues pour capter l'attention, rendre la création de compte inévitable, et rendre le partage vers l'extérieur aussi difficile que possible.

3. Le glissement vers la surveillance : utiliser les réseaux sociaux pour suivre l'actualité requiert aujourd'hui de s'identifier. Bon nombre d'état souhaite imposer la vérification d'identité. Suivre l'actualité ne devrait jamais exiger qu'on se déshabille.

4. Les réseaux sociaux sont massivement utilisés pour suivre l'actualité alors qu'ils sont architecturés autour de biais évidents : algorithmes de recommandation opaques, optimisation pour le temps d'attention, intérêts publicitaires et politiques.

5. Les flux RSS fonctionnent bien, mais passent mal à l'échelle. 250 articles par jour — cas courant pour un utilisateur quotidien — génèrent une charge qui peut décourager de nombreux utilisateurs. Seule une fraction de ces articles l'intéresseront vraiment.

6. La republication parasitaire : beaucoup de sites reformulent le même article à la marge pour contourner les droits d'auteur, générant des doublons sémantiques sans valeur ajoutée qui encombrent les flux.

7. La découvrabilité est absente du RSS : il n'existe aucun mécanisme natif pour trouver des articles intéressants publiés par des blogs qu'on ne suit pas encore, sauf s'ils sont explicitement mentionnés dans un article qu'on a déjà lu.

## Valeurs

Ce projet est conduit avec des convictions importantes :

- **Vie privée** : Aucun utilisateur ne devrait avoir à abandonner son intimité pour avoir le droit de suivre l'actualité. Lire un flux ne laisse aucune trace.

- **La qualité prime sur la quantité** : Un flux restreint d'articles diversifiés et choisis vaut mieux qu'un fil infini optimisé pour l'engagement. Nous préférons lire moins, et mieux.

- **Le lien humain au cœur du partage** : Les recommandations devraient venir d'êtres humains qui ont jugé une information par eux-mêmes et souhaitent la partager à ceux qui les suivent — pas d'un algorithme optimisant des intérêts opaques. La liberté de partager ne se résume pas à la possibilité technique de le faire : c'est aussi la possibilité de le faire vertueusement, pour des raisons qui tiennent à la qualité de l'information plutôt qu'à l'intérêt de la plateforme.

- **La connaissance doit circuler librement** : Le partage d'information est essentiel à la vie en société. Ce droit doit être durable et résilient face au capitalisme de l'information. Une information jugée de qualité doit pouvoir se repartager facilement. Toutes personnes compétentes doit pouvoir la repartager sans dépendre de plateformes fermées.

## Ambitions

ActuPub vise à répondre à ces problématiques à travers un protocole qui :

- **Filtre et ordonne les articles** selon des critères mesurables et transparents : thème, longueur, champ lexical, diversité sémantique. Ces mêmes métriques améliorent par ailleurs la détection de doublons. Ainsi **améliore-t-on la qualité des informations** en permettant de lire moins, mais mieux.

- **Fédère les serveurs entre eux** : chaque serveur ActuPub peut repartager les articles d'autres blogs sans les héberger. Le serveur devient lui-même client d'un autre serveur, automatiquement (abonnement filtré) ou manuellement (ajout d'un article au cas par cas).

- **Facilite la transition depuis RSS, Atom et JSON Feed** vers ActuPub, pour ne pas repartir de zéro et préserver l'écosystème existant.

- **Ne requiert aucune information personnelle** côté serveur pour suivre l'actualité.

Nous espérons ainsi :

- Rendre la veille informationnelle accessible malgré la surabondance d'information et la dégradation de sa qualité moyenne ;

- Favoriser des échanges entre blogs plutôt que des réactions impulsives sans argument sur les réseaux sociaux ;

- Réduire les biais algorithmiques et les bulles de filtre grâce à une sélection humaine et distribuée ;

- Permettre le suivi d'actualité sans identification ni surveillance ;

- Améliorer la visibilité des articles qui le méritent grâce à la sélection et au repartage communautaire.

## Le projet

Le protocole se structure autour de deux projets distincts :

| Projet | Type | Document |
|---|---|---|
| **ActuPub** | Le protocole, ses couches, son schéma de données | [ACTUPUB.md](./ACTUPUB.md) |
| **CuraFed** | Service web open source, implémentation de référence | [CURAFED.md](./CURAFED.md) |
