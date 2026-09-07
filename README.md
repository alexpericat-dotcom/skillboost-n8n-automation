# 🚀 SkillBoost Academy — Système d'automatisation des inscriptions

Solution d'automatisation no-code / low-code pour la gestion de bout en bout des inscriptions, déduplication, gestion des files d'attente FIFO, rappels automatiques et émargement terrain.

---

## 🛠️ Stack Technique

* **Moteur d'orchestration :** n8n (Community Edition) VPS
* **Base de données relationnelle :** Baserow Cloud
* **Interfaces & Canaux :** Telegram Bot API (`@SkillBoost_Admin_Bot`), Gmail API
* **Résilience & Failover :** Gestionnaire d'erreurs global (alertes hors-bande OVH SMS)

---

## 🏗️ Architecture des Workflows n8n

* **WF-00 :** API Disponibilité Ateliers (catalogue en direct)
* **WF-01 :** Inscription aux Ateliers SkillsBoost
* **WF-02 :** Traitement Inscription DB (contrôle d'intégrité & idempotence)
* **WF-03 :** Gestion Annulation & Repêchage (FIFO automatique)
* **WF-04A :** Relances Automatiques J-1 (CRON matinal)
* **WF-04B :** Relances H-2 (CRON horaire)
* **WF-05A :** Génération Liste d'Appel (Telegram Bot `/appel`)
* **WF-05B :** Clôture Session & Émargement interactif
* **WF-ERR :** Gestionnaire d'Erreurs Globales & Circuit Breaker

---

## 🗄️ Schéma Relationnel des Données (Baserow)

```mermaid
erDiagram
    ATELIERS ||--o{ INSCRIPTIONS : "accueille (1:N)"
    ATELIERS ||--o{ LOGS : "trace (1:N)"

    ATELIERS {
        int id PK
        string ID_Atelier UK
        string Nom_de_latelier
        datetime Date_et_Heure
        int Capacite_Max
        string Lieu
        string Materiel
        int Inscrits_Confirmes
        int Places_Restantes
        string Taux_de_remplissage
    }

    INSCRIPTIONS {
        int id PK
        string Email
        string Atelier_Choisi FK
        string Statut
        string Cle_Deduplication UK
        int Check_Confirme
        datetime Date_Inscription
    }

    LOGS {
        int id PK
        datetime Timestamp
        string Type_Evenement
        string Email_Utilisateur
        string ID_Atelier FK
        string Message
        string WorkFlow
    }
```
---

## 🔒 Sécurité & Idempotence

* **Unicité :** Clé composite `email_idAtelier` calculée avant toute écriture.
* **Annulation dynamique :** Altération de la clé sous la forme `email_idAtelier_timestamp` pour autoriser une réinscription ultérieure.
* **Audit :** Journalisation de chaque transition d'état dans la table `Logs`.
