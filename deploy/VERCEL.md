# Déploiement production sur Vercel — pas à pas

> Durée : ~15 minutes. Coût : **0 DT** pour démarrer (Vercel Hobby + Neon gratuit).
> ⚠️ Le plan **Hobby** de Vercel est réservé à un usage non-commercial : pendant le pilote c’est
> parfait, mais dès l’abonnement aux premiers barbiers, passez en **Pro (~20 $/mois)** ou sur le
> VPS (voir `deploy/PRODUCTION.md`). Les deux chemins utilisent exactement ce code.

---

## 1 · Prérequis

- Un compte [github.com](https://github.com) — le code est déjà sur `iyedomri/barber_shop`.
- Un compte [vercel.com](https://vercel.com) (connexion GitHub en 1 clic).
- 15 minutes.

## 2 · Importer le projet

1. Vercel → **Add New → Project** → importez `iyedomri/barber_shop`.
2. **Framework Preset : Other**. Aucune commande de build, aucun output directory.
3. **Root Directory** : racine du dépôt (laissez vide).
4. Ne cliquez pas encore sur Deploy — on met d’abord la base et les variables.

## 3 · La base de données (Postgres gratuit)

1. Vercel → **Storage → Create → Postgres** (Neon) → *Connect to a project* → `barber_shop`.
2. La variable **`POSTGRES_URL`** est injectée automatiquement. C’est toute la base dont le
   serveur a besoin : une seule ligne (`barbier_kv`) contient la plateforme entière.

## 4 · Les variables d’environnement

Vercel → **Settings → Environment Variables** :

| Variable | Valeur | Obligatoire ? |
|---|---|---|
| `POSTGRES_URL` | *(injectée par Storage)* | ✅ |
| `SUPER_ADMIN_PIN` | **un PIN secret que vous choisissez** (ex. 6–8 chiffres) | ✅ **changez-le** |
| `SEED_DEMO` | `false` | ✅ production vierge (la landing masque son bloc démo) |
| `D17_NUMBER` | votre numéro D17 (ex. `+216 XX XXX XXX`) | recommandé — affiché sur la page de pause |
| `PAYMENT_LINK` | lien Stripe Payment Link *(optionnel)* | — |
| `SESSION_SECRET` | `openssl rand -hex 32` | recommandé — signature des sessions |
| `TZ` | `Africa/Tunis` | recommandé (le code s’y ancre par défaut) |
| `BASE_URL` | `https://votre-domaine.tn` *(optionnel)* | — (les QR utilisent sinon l’Host de la requête) |

## 5 · Deploy

Cliquez **Deploy**. `vercel.json` envoie tout vers la fonction `api/index.js` ; les assets
statiques (css/js/img/super) passent par le CDN.

## 6 · Vérifications post-deploy (5 minutes)

1. `https://<votre-app>.vercel.app/` → la landing **sans** bloc démo (pas de liens vers les
   salons de démo, pas de PIN).
2. `https://<votre-app>.vercel.app/super` → connectez-vous avec votre `SUPER_ADMIN_PIN` →
   créez votre premier salon (nom → slug auto, PIN choisi). MRR = 0, tout est prêt.
3. Sur le téléphone : `/s/<slug>/` → créez une carte → `/s/<slug>/admin` (PIN du salon) →
   **Scanner la carte** → code manuel → +1 tampon. ✅
4. Pausez le salon depuis `/super` → la page de pause affiche votre numéro D17 →
   « +30 j payé » → tout revient. ✅
5. Testez la caméra QR sur un vrai téléphone (HTTPS requis — Vercel le fournit).

## 7 · Domaine (recommandé)

Vercel → **Settings → Domains** → ajoutez `cartebarber.tn` (ou votre domaine) → suivez les
enregistrements DNS proposés. Les certificats HTTPS sont automatiques.

## 8 · Sauvegarde

Une ligne = toute la plateforme :
- **Neon** : *Backups* intégrés (restore point-in-time sur le plan gratuit selon offre), ou
- export manuel : `pg_dump "$POSTGRES_URL" > backup-$(date +%F).sql` en cron mensuel.

## 9 · Ce qui est déjà géré par le code sur Vercel

- **Sessions admin/super signées HMAC (sans état)** — validables par toutes les instances lambda.
- **Rate-limit par IP réelle** (`X-Forwarded-For`) — pas de seau partagé entre clients.
- **Connexions Postgres réutilisées** par instance (pool `max: 1`).
- **Clés VAPID persistées** en base — les notifications push survivent aux redéploiements.
- **`views/` inclus dans le bundle** de la fonction (`vercel.json → includeFiles`).
- **Fuseau Tunis** pour les dates, créneaux et anniversaires.
- Pages modèles rendues **uniquement côté serveur** — aucun `{{PLACEHOLDER}}` ne peut fuiter.

## 10 · Montée en charge

Le modèle actuel (une ligne JSON) tient sans problème **plusieurs centaines de salons**.
Au-delà : sharding par salon (une ligne `barbier_kv:<tenantId>` — refactor léger) puis
facturation automatisée (Stripe Billing — voir *Roadmap P1* du README).

---

Support / commercial : **iyedomri922@gmail.com**
