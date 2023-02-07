---
nav_title: Approuver des campagnes
article_title: Approuver des campagnes
alias: "/campaign_approval/"
page_order: 0
page_type: reference
description: "Cette page donne un aperçu du processus d’approbation d’une campagne."
tool: Campagnes
---

# Approuver des campagnes

L’approbation de la campagne ajoute un processus de révision à votre flux de travail avant de lancer une campagne. Disponible exclusivement pour les campagnes, cette fonctionnalité ajoute de nouveaux états disponibles dans l’étape du flux de travail de confirmation de la campagne. Vous pouvez maintenant vous assurer que chaque confirmation est approuvée afin de lancer la campagne.

## Activer l’approbation de campagne

Par défaut, le paramètre d’approbation de la campagne est désactivé. Pour activer cette fonctionnalité, allez dans **Manage Settings** > **Flux de travail d’approbation**.

{% alert note %}
Seuls les administrateurs et utilisateurs disposant d’autorisations pour gérer les paramètres d’approbation verront cette page.
{% endalert %}

## Utiliser les approbations

Dans l’étape **Résumé de la révision** du flux de travail de création de la campagne, il existe une option d’approbation pour approuver ou rejeter des composants clés de votre campagne : **messages**, **livraison**, **audience cible**, et **événements de conversion**. L’état par défaut d’approbation de la campagne est **Approbation en attente**. 

![][1]

Une fois que chaque section est approuvée, le bouton **Lancer** sera activé et vous pourrez lancer votre campagne ! 

{% alert important %}
L’approbation de campagne n’est pas prise en charge dans le flux de travail de création des Canvas ou des campagnes par API.
{% endalert %}

[1]: {% image_buster /assets/img_archive/campaign_approval_example.png %} 
