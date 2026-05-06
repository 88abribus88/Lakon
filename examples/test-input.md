# Guide de configuration du système d'alertes

## Introduction

Ce document décrit **les étapes nécessaires** pour configurer correctement le système d'alertes dans votre environnement de production. Il est important de suivre ces instructions dans l'ordre indiqué, car certaines étapes dépendent des précédentes.

---

## Prérequis

Avant de commencer, vous devez vous assurer que les éléments suivants sont en place :

- **Serveur principal** : version 3.2 ou supérieure
- **Base de données** : PostgreSQL 14+
- **Accès réseau** : ports 8080, 8443 ouverts
- **Credentials** : fichier de configuration situé à `/etc/alerts/config.yaml`

---

## Règles de déclenchement

Le système applique les règles suivantes pour déterminer quand une alerte doit être émise :

**Règle 1** — Si le taux d'erreur dépasse 5% sur une fenêtre de 60 secondes, une alerte critique est déclenchée immédiatement. Cette règle ne peut pas être désactivée.

**Règle 2** — Si la latence moyenne dépasse 2000ms pendant plus de 3 minutes consécutives, une alerte de niveau warning est envoyée à l'équipe concernée.

**Règle 3** — Il ne faut pas modifier les seuils sans validation préalable de l'équipe infrastructure. Toute modification non autorisée sera annulée automatiquement.

---

## Procédure d'installation

Pour installer le système, il faut suivre ces étapes dans l'ordre :

1. Télécharger le paquet depuis le serveur interne : `wget https://internal.corp/alerts-3.2.tar.gz`
2. Décompresser l'archive et naviguer dans le répertoire
3. Exécuter le script d'installation avec les droits administrateur
4. Vérifier que le service démarre correctement en consultant les logs

Le script d'installation crée automatiquement les fichiers de configuration par défaut dans `/etc/alerts/`. *Il est recommandé de sauvegarder ces fichiers avant toute modification.*

---

## Gestion des erreurs

En cas de problème lors de l'installation, les erreurs les plus courantes sont les suivantes :

- **Erreur 403** : droits insuffisants — vérifier les permissions sur `/etc/alerts/`
- **Erreur connexion DB** : vérifier que PostgreSQL est bien démarré et accessible sur le port 5432
- **Service ne démarre pas** : consulter `/var/log/alerts/startup.log` pour le détail de l'erreur

Si aucune de ces solutions ne résout le problème, contacter l'équipe support en fournissant le contenu complet du fichier de log.
