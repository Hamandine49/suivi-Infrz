Inforezo — Tableau de bord Suivi Contrats
Tableau de bord de suivi des abonnements et contrats clients d'Inforezo (ALH Conseil).  
Déployé sur GitHub Pages, connecté à une base de données Supabase.
---
Ce que fait ce tableau de bord
Vue globale : KPIs (nombre de contrats, retards, échéances dans 30 jours, CA annuel estimé) + grille par produit + alertes
Par mois : calendrier de l'exercice comptable (avril → mars), total à facturer par mois
Contrats : liste complète avec recherche et filtre, modification en un clic
Clients : répertoire avec nombre de contrats actifs par client
Configuration : connexion Supabase, script SQL pour initialiser les tables
Produits suivis
Office 365 · Domaines · ESET MSP · Sauvegarde · Licences · Contrats EPP · Maintenance Serveur · Licences VPN
Logique de facturation
Chaque contrat a trois dates distinctes :
Champ	Rôle
Dernière facture émise	Date réelle de la dernière facture envoyée
Échéance contractuelle	Date anniversaire du contrat (ne change jamais)
Prochaine échéance (calculé)	Échéance contractuelle + périodicité
La prochaine échéance se calcule depuis l'échéance contractuelle, pas depuis la dernière facture.  
Ainsi, si tu factures en retard, le cycle reste calé sur la bonne date anniversaire.
---
Architecture technique
```
GitHub Pages          Supabase (PostgreSQL)
┌─────────────────┐   ┌──────────────────────┐
│  index.html     │──▶│  Table : clients      │
│  (1 seul        │   │  Table : contrats     │
│   fichier HTML) │   └──────────────────────┘
└─────────────────┘
```
Frontend : HTML + CSS + JavaScript vanilla (pas de framework)
Base de données : Supabase (PostgreSQL hébergé)
Hébergement : GitHub Pages (gratuit)
Connexion : librairie `@supabase/supabase-js` chargée via CDN
Authentification : clé publique `anon` Supabase, stockée dans le `localStorage` du navigateur
---
Structure de la base de données
```sql
-- Table clients
CREATE TABLE clients (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  nom text NOT NULL,
  type text,                  -- TPE, Artisan, Professionnel, Collectivité, Association, Doctolib, Particulier
  contact text,
  email text,
  telephone text,
  adresse text,
  departement text,
  notes text,
  created_at timestamptz DEFAULT now()
);

-- Table contrats
CREATE TABLE contrats (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  client_id uuid REFERENCES clients(id) ON DELETE CASCADE,
  produit text NOT NULL,               -- Office 365, Domaines, ESET MSP, etc.
  detail text,                         -- Description libre (nb licences, offre...)
  periodicite text NOT NULL DEFAULT 'Annuel',  -- Mensuel, Trimestriel, Semestriel, Annuel
  montant_ht numeric(10,2),
  derniere_facture date,
  echeance_contractuelle date,         -- Date anniversaire réelle du contrat
  facture_ebp text,                    -- Référence facture dans EBP
  notes text,
  actif boolean DEFAULT true,          -- false = contrat archivé (pas supprimé)
  created_at timestamptz DEFAULT now()
);
```
> Les contrats ne sont jamais supprimés, seulement archivés (`actif = false`).
---
Mise en place initiale (faite une seule fois)
1. Créer le repo GitHub
Créer un nouveau repo sur github.com
Déposer `index.html` à la racine
Aller dans Settings → Pages → Branch : main → Save
L'URL du tableau de bord sera : `https://[ton-pseudo].github.io/[nom-du-repo]`
2. Créer la base Supabase
Créer un projet sur supabase.com
Aller dans SQL Editor → New query
Coller le SQL ci-dessus (section "Structure de la base de données")
Cliquer Run with RLS (active la sécurité par défaut)
Vérifier que le résultat affiche `Success`
3. Connecter le tableau de bord à Supabase
Dans Supabase : Project Settings (icône engrenage) → API
Copier :
Project URL : `https://xxxxxxxxx.supabase.co`
anon / public key : `eyJhbGci…` (longue clé)
Ouvrir le tableau de bord via l'URL GitHub Pages
Aller dans Configuration
Coller l'URL et la clé → Enregistrer et tester
Le point en haut à droite doit passer au vert ✅
> Les clés sont sauvegardées dans le `localStorage` du navigateur.  
> Si tu changes de navigateur ou d'ordinateur, il faudra les re-saisir une fois.
---
Mettre à jour le fichier index.html
Quand une nouvelle version du fichier `index.html` est fournie, voici comment mettre à jour :
Méthode rapide (directement sur GitHub)
Aller sur le repo GitHub
Cliquer sur `index.html` dans la liste des fichiers
Cliquer sur l'icône crayon ✏️ en haut à droite du fichier
Faire Ctrl+A pour tout sélectionner dans l'éditeur
Supprimer tout le contenu sélectionné
Coller le nouveau contenu du fichier (`Ctrl+V`)
Cliquer Commit changes (bouton vert en haut à droite)
Attendre 1 à 2 minutes que GitHub Pages se mette à jour
Dans le navigateur, faire Ctrl+Maj+R (ou Cmd+Shift+R sur Mac) pour vider le cache
Rouvrir l'URL du tableau de bord
Méthode via upload
Aller sur le repo GitHub
Cliquer Add file → Upload files
Glisser-déposer le nouveau `index.html`
GitHub détecte que le fichier existe déjà et le remplace
Cliquer Commit changes
Attendre 1 à 2 minutes + vider le cache navigateur
> ⚠️ Toujours vider le cache (`Ctrl+Maj+R`) après une mise à jour.  
> Sans ça, le navigateur affiche l'ancienne version.
---
Utilisation au quotidien
Ajouter un client
Onglet Clients → + Nouveau client → remplir le formulaire → Enregistrer.
Ajouter un contrat
Bouton + Ajouter un contrat (en haut à droite, toujours visible) → choisir le client et le produit → renseigner :
La périodicité (Mensuel / Trimestriel / Semestriel / Annuel)
Le montant HT
La dernière facture émise (date réelle de ta dernière facture)
L'échéance contractuelle (date anniversaire du contrat — ne change pas même si tu factures en retard)
Le N° facture EBP une fois la facture émise
Consulter les échéances du mois
Onglet Par mois → cliquer sur le mois voulu dans le calendrier → voir la liste des contrats à facturer avec le total HT.
Marquer une facture comme émise
Cliquer Modifier sur le contrat → renseigner le N° Facture EBP → mettre à jour la Dernière facture émise → Enregistrer.
---
Exercice comptable
Le calendrier est calé sur un exercice du 1er avril au 31 mars.  
Il affiche toujours les 12 mois de l'exercice en cours, en commençant par avril.
---
Fichier Excel complémentaire
Un fichier Excel (`Inforezo_Suivi_v3.xlsx`) existe en parallèle avec les mêmes données.  
Il peut être utilisé hors connexion ou pour préparer les données avant import.  
Il contient également :
Un onglet Agenda & Projets pour les événements futurs (ex : remplacement serveur prévu en 2028)
Un onglet Prospects & Relances pour les clients à réactiver (calcul automatique de la date de relance à +2 ans)
---
Intégration Power Automate (optionnel)
Pour recevoir des rappels automatiques par email dans Outlook :
Déposer le fichier Excel sur OneDrive (pas sur le bureau local)
Aller sur flow.microsoft.com
Créer un flux avec :
Déclencheur : Récurrence → tous les jours à 8h
Action 1 : Excel Online → Lister les lignes du tableau de bord
Action 2 : Pour chaque ligne où "Jours restants" ≤ 30 → Envoyer un email avec client / service / montant / date échéance
> Prérequis : les données Excel doivent être dans un **Tableau formaté** (Insertion → Tableau dans Excel).
---
Contact et maintenance
Projet développé pour Inforezo / ALH Conseil — Cholet (49) et Pouzauges (85).  
En cas de question sur le code, conserver l'historique de conversation avec l'IA pour retrouver le contexte.
