---
nav_title: Aperçu et bonnes pratiques
article_title: Aperçu et bonnes pratiques de l’ingestion de données cloud
page_order: 0
page_type: reference
description: "Cet article de référence fournit un aperçu de l’ingestion de données cloud ainsi que des bonnes pratiques."

---

# Ingestion de Données Cloud dans Braze

## Qu’est-ce que l’ingestion de données Cloud ?

L’ingestion de données cloud dans Braze vous permet de configurer une connexion directe entre votre instance Snowflake et Braze pour synchroniser les attributs, les événements et les achats pertinents de l’utilisateur. Une fois synchronisées avec Braze, ces données peuvent être exploitées dans des cas d’utilisation tels que la personnalisation ou la segmentation.

{% alert important %}
L’ingestion de données cloud Braze est actuellement en accès anticipé. Contactez votre gestionnaire de compte Braze si vous souhaitez participer à l’accès anticipé.
{% endalert %}

### Fonctionnement

Grâce à l’ingestion de données cloud de Braze, vous pouvez mettre en place une intégration entre votre instance Snowflake et le groupe d’apps de Braze pour synchroniser des données de manière récurrente. Cette synchronisation se fait selon la planification que vous déterminez et chaque intégration peut disposer d’une planification différente. Les synchronisations peuvent avoir lieu d’une fois toutes les 15 minutes à une fois par mois. Les clients ayant besoin d’une synchronisation plus fréquente que toutes les 15 minutes peuvent contacter leur gestionnaire du succès des clients ou envisager d’utiliser les appels API REST pour une ingestion de données en temps réel.

Pendant qu’une synchronisation a lieu, Braze se connectera directement à votre instance Snowflake, récupérera toutes les nouvelles données d’un tableau donné et mettra à jour les profils utilisateurs correspondants sur votre tableau de bord de Braze. Chaque fois qu’une synchronisation a lieu, toutes les données mises à jour s’afficheront sur les profils utilisateurs.

### Qu’est-ce qui est synchronisé

Chaque fois qu’une synchronisation a lieu, Braze cherchera les lignes qui n’ont pas déjà été synchronisées. Nous le vérifions en utilisant la colonne `UPDATED_AT` dans votre table ou votre affichage. Chaque ligne dans laquelle `UPDATED_AT` est plus ancien que la dernière synchronisation, la ligne sera sélectionnée et transmise à Braze.

Dans Snowflake, vous ajoutez les utilisateurs et attributs suivants à votre table, en définissant la date `UPDATED_AT` sur la date à laquelle vous ajoutez ces données.

| UPDATED_AT | EXTERNAL_ID | PAYLOAD |
| --- | --- | --- |
| `2022-07-19 09:07:23` | `customer_1234` | {<br>&nbsp;&nbsp;&nbsp;&nbsp;"attribute_1":"abcdefg",<br>&nbsp;&nbsp;&nbsp;&nbsp;"attribute_2":42,<br>&nbsp;&nbsp;&nbsp;&nbsp;"attribute_3":"2019-07-16T19:20:30+1:00"<br>} |
| `2022-07-19 09:07:23` | `customer_3456` | {<br>&nbsp;&nbsp;&nbsp;&nbsp;"attribute_1":"abcdefg",<br>&nbsp;&nbsp;&nbsp;&nbsp;"attribute_2":42,<br>&nbsp;&nbsp;&nbsp;&nbsp;"attribute_3":"2019-07-16T19:20:30+1:00",<br>&nbsp;&nbsp;&nbsp;&nbsp;"attribute_5":"testing"<br>} |
| `2022-07-19 09:07:23` | `customer_5678` | {<br>&nbsp;&nbsp;&nbsp;&nbsp;"attribute_1":"abcdefg",<br>&nbsp;&nbsp;&nbsp;&nbsp;"attribute_4":true,<br>&nbsp;&nbsp;&nbsp;&nbsp;"attribute_5":"testing_123"<br>} |

Pendant la prochaine synchronisation planifiée, toutes les lignes possédant un timestamp `UPDATED_AT` plus ancien que le timestamp le plus récent seront synchronisées aux profils utilisateurs de Braze. Les champs seront mis à jour ou ajoutés et vous n’aurez donc pas besoin de synchroniser le profil utilisateur tout entier à chaque fois. Après la synchronisation, les utilisateurs afficheront les nouvelles mises à jour :

```json
{
  "external_id":"customer_1234",
  "email":"jane@exaple.com",
  "attribute_1":"abcdefg",
  "attribute_2":42,
  "attribute_3":"2019-07-16T19:20:30+1:00",
  "attribute_4":false,
  "attribute_5":"testing"
}
```
```json
{
  "external_id":"customer_3456",
  "email":"michael@exaple.com",
  "attribute_1":"abcdefg",
  "attribute_2":42,
  "attribute_3":"2019-07-16T19:20:30+1:00",
  "attribute_4":true,
  "attribute_5":"testing"
}
```
```json
{
  "external_id":"customer_5678",
  "email":"bob@exaple.com",
  "attribute_1":"abcdefg",
  "attribute_2":42,
  "attribute_3":"2017-08-10T09:20:30+1:00",
  "attribute_4":true,
  "attribute_5":"testing_123"
}
```

### Utilisation de points de données

Chaque attribut envoyé pour un utilisateur utilisera un point de données. Il dépend de vous de n’envoyer que les données requises. Le suivi de points de données pour l’ingestion de données pour le cloud est équivalent au suivi à l’aide de l’endpoint [`/users/track`]({{site.baseurl}}/api/endpoints/user_data/post_user_track#user-track). Consultez les [points de données]({{site.baseurl}}/user_guide/onboarding_with_braze/data_points/) pour plus d’informations.

## Recommandations de paramétrage de données

#### N’écrivez que les attributs nouveaux ou mis à jour pour limiter l’utilisation

Nous synchroniserons tous les attributs dans une ligne donnée, qu’elles soient ou non les mêmes que celles contenues actuellement dans le profil utilisateur. Voilà pourquoi nous recommandons de ne synchroniser que les attributs que vous voulez ajouter ou mettre à jour.

#### Utilisez un timestamp UTC pour la colonne UPDATED_AT

La colonne `UPDATED_AT` doit être en UTC pour éviter les problèmes liés aux heures d’été. Utilisez de préférence des fonctions uniquement en UTC, telles que `SYSDATE()` plutôt que `CURRENT_DATE()` lorsque c’est possible.

#### Séparer EXTERNAL_ID de la colonne PAYLOAD. 
L’objet PAYLOAD ne doit pas inclure d’identifiant externe ou d’autre type d’identifiant. 

#### Retirer un attribut

Vous pouvez le définir sur 'null' si vous désirez retirer complètement un attribut d’un profil utilisateur. Si vous désirez qu’un attribut reste inchangé, ne l’envoyez pas à Braze jusqu’à ce qu’il ait été mis à jour.

#### Créer une chaîne de caractères JSON depuis une autre table

Si vous préférez stocker de manière interne chaque attribut dans sa propre colonne, vous devez convertir ces colonnes en strings JSON pour renseigner la synchronisation avec Braze. Pour ce faire, vous pouvez utiliser une requête du type :
```json
CREATE TABLE "EXAMPLE_USER_DATA"
    (attribute_1 string,
     attribute_2 string,
     attribute_3 number,
     my_user_id string);

SELECT
    CURRENT_TIMESTAMP as UPDATED_AT,
    my_user_id as EXTERNAL_ID,
    TO_JSON(
        OBJECT_CONSTRUCT (
            'attribute_1',
            attribute_1,
            'attribute_2',
            attribute_2,
            'yet_another_attribute',
            attribute_3)
    )as PAYLOAD FROM "EXAMPLE_DATA";
```

#### Utilisation de l’horodatage UPDATED_AT

Nous utilisons l’horodatage `UPDATED_AT` pour suivre quelles données ont été synchronisées avec succès dans Braze. Si de nombreuses rangées sont écrites avec le même horodatage pendant qu’une synchronisation est en cours, ceci peut entraîner la synchronisation de données en double dans Braze. Voici quelques conseils pour éviter des données en double :
- Si vous mettez en place une synchronisation pour une `VIEW`, n’utilisez pas `CURRENT_TIMESTAMP` comme valeur par défaut. Cela entrainera la synchronisation de toutes les données chaque fois qu’elle s’effectue, car le champ `UPDATED_AT` sera mis à jour à chaque exécution de nos requêtes. 
- Si vous avez des flux de données à très longue durée d’exécution ou des requêtes écrivant des données dans votre table source, évitez de les exécuter en même temps que la synchronisation ou évitez d’utiliser le même horodatage pour chaque ligne insérée.
- Utilisez une transaction pour écrire toutes les rangées qui ont le même horodatage.

#### Exemple de configuration de table

Nous avons un [GitHub dépôt](https://github.com/braze-inc/braze-examples/tree/main/data-ingestion) public où les clients peuvent partager leurs bonnes pratiques ou leurs extraits de code. Créez une requête d’extraction pour contribuer avec vos propres extraits de code !

## Limites du produit

| Limitations | Description |
| --- | --- |
| Nombre d’intégrations | Le nombre d’intégrations que vous pouvez définir n’est pas limité. Cependant, vous ne pourrez définir qu’une seule intégration par table ou par affichage.
| Nombre de lignes | Le nombre de lignes que vous pouvez synchroniser n’est pas limité. Chaque ligne ne sera synchronisée qu’une fois, en fonction de la colonne `UPDATED_AT`. |
| Attributs par ligne | Chaque ligne doit contenir un seul ID utilisateur et un seul objet JSON contenant jusqu’à 50 attributs. Chaque clé dans l’objet JSON compte comme un seul attribut (c.-à-d., un tableau compte comme un attribut). |
| Type de données | Vous pouvez synchroniser les attributs utilisateurs via l’ingestion de données cloud. |
| Région Braze | Ce produit est disponible dans toutes les régions Braze. Toutes les régions Braze peuvent se connecter à toutes les régions Snowflake |
| Région Snowflake | Vous pouvez connecter votre instance Snowflake à Braze dans toutes les régions et tous les clouds en utilisant ce produit. |
{: .reset-td-br-1 .reset-td-br-2}

<br><br>