
# Configuration du Panneau d'Administration

Ce document explique comment configurer le compte administrateur pour gérer les réservations.

## Vue d'ensemble

Le système d'administration permet à des utilisateurs spécifiques (administrateurs) de:
- Voir toutes les réservations
- Modifier les réservations existantes
- Supprimer des réservations
- Mettre à jour les statuts de réservation et de paiement

## Compte Administrateur

**Email administrateur:** khaireddinechouki@outlook.fr

## Installation

### Étape 1: Créer la table admin_users

Connectez-vous à votre tableau de bord Supabase et exécutez le SQL suivant dans l'éditeur SQL:

```sql
-- Créer la table admin_users
CREATE TABLE IF NOT EXISTS admin_users (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  email text UNIQUE NOT NULL,
  role text NOT NULL DEFAULT 'admin',
  created_at timestamptz DEFAULT now()
);

-- Activer RLS (Row Level Security)
ALTER TABLE admin_users ENABLE ROW LEVEL SECURITY;

-- Politique RLS pour permettre aux utilisateurs authentifiés de voir les admins
CREATE POLICY "Admins can view admin records"
ON admin_users
FOR SELECT
TO authenticated
USING (true);

-- Créer des index pour améliorer les performances
CREATE INDEX IF NOT EXISTS idx_admin_users_email 
ON admin_users(email);

-- Insérer le compte administrateur
INSERT INTO admin_users (email, role)
VALUES ('khaireddinechouki@outlook.fr', 'admin')
ON CONFLICT (email) DO NOTHING;
```

### Étape 2: Mettre à jour les politiques RLS des réservations

Exécutez le SQL suivant pour permettre aux administrateurs de gérer toutes les réservations:

```sql
-- Politique pour permettre aux admins de voir toutes les réservations
CREATE POLICY "Admins can view all bookings"
ON bookings
FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM admin_users
    WHERE admin_users.email = (
      SELECT email FROM auth.users
      WHERE id = auth.uid()
    )
  )
);

-- Politique pour permettre aux admins de modifier les réservations
CREATE POLICY "Admins can update all bookings"
ON bookings
FOR UPDATE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM admin_users
    WHERE admin_users.email = (
      SELECT email FROM auth.users
      WHERE id = auth.uid()
    )
  )
);

-- Politique pour permettre aux admins de supprimer les réservations
CREATE POLICY "Admins can delete all bookings"
ON bookings
FOR DELETE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM admin_users
    WHERE admin_users.email = (
      SELECT email FROM auth.users
      WHERE id = auth.uid()
    )
  )
);
```

## Utilisation

### Accès au panneau d'administration

1. **Créer un compte** avec l'email `khaireddinechouki@outlook.fr` via l'application
2. **Se connecter** avec ce compte
3. **Aller dans l'onglet Profil** - vous verrez un badge "Administrateur"
4. **Cliquer sur "Panneau d'administration"** pour accéder au panneau

### Fonctionnalités du panneau

#### Voir les réservations
- Toutes les réservations sont affichées avec leurs détails complets
- Statistiques en temps réel (total, confirmées, en attente)
- Bouton "Actualiser" pour recharger les données

#### Modifier une réservation
1. Cliquer sur le bouton "Modifier" d'une réservation
2. Modifier les champs souhaités:
   - Dates de début et fin
   - Heures de début et fin
   - Statut de réservation (Confirmé, En attente, Annulé, Terminé)
   - Statut de paiement (Payé, En attente, Remboursé)
   - Lieu de récupération
3. Cliquer sur "Enregistrer"

#### Supprimer une réservation
1. Cliquer sur le bouton "Supprimer" d'une réservation
2. Confirmer la suppression dans la boîte de dialogue

## Sécurité

- Seuls les utilisateurs listés dans la table `admin_users` peuvent accéder au panneau
- Les politiques RLS (Row Level Security) garantissent que seuls les admins peuvent modifier les réservations
- Toutes les actions sont enregistrées avec des timestamps

## Ajouter d'autres administrateurs

Pour ajouter un nouvel administrateur, exécutez:

```sql
INSERT INTO admin_users (email, role)
VALUES ('nouvel.admin@example.com', 'admin');
```

## Dépannage

### L'utilisateur ne peut pas accéder au panneau d'administration

1. Vérifiez que l'email est correctement enregistré dans la table `admin_users`
2. Vérifiez que l'utilisateur est connecté avec le bon compte
3. Vérifiez que les politiques RLS sont correctement configurées

### Les modifications ne sont pas enregistrées

1. Vérifiez que les politiques RLS UPDATE sont en place
2. Vérifiez les logs Supabase pour les erreurs
3. Assurez-vous que l'utilisateur est bien authentifié

## Support

Pour toute question ou problème, contactez le support technique.
