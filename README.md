# Projet — **Songo Beafrika**

*Plate-forme de promotion & publicité pour connecter l’économie locale de la République Centrafricaine au monde*

Super projet — je te propose ci-dessous une fiche projet complète (objectifs, fonctionnalités, MVP, architecture technique, modèles de données, monétisation, design & branding). Tu peux prendre ce plan tel quel ou me dire ce que tu veux que je développe en priorité (ex : modèles Django, API DRF, maquette Tailwind/Vue, ou logo).

---

# 1. Résumé / vision

Songo Beafrika est une plateforme de communication ouverte qui donne aux entreprises locales (PME, artisans, prestataires, événements) un espace professionnel unique pour publier des annonces multimédias (image, infographie, audio, vidéo / publi-reportage) et toucher des clients au niveau national et international. Objectif : augmenter la visibilité, faciliter les partenariats et stimuler les échanges commerciaux depuis la RCA vers l'extérieur.

---

# 2. Objectifs principaux

* Offrir un espace simple et accessible pour créer des annonces multimédias.
* Permettre la recherche/ciblage par secteur, localisation et audience.
* Fournir des outils analytiques pour mesurer les performances des annonces.
* Monétiser via packages publicitaires et options de mise en avant.
* Assurer modération, sécurité et bonne qualité média (transcodage, CDN).

---

# 3. Fonctionnalités clés (par bloc)

## Découverte & catalogue

* Page d’accueil / feed d’annonces (triées par popularité, récentes, géo).
* Recherche avancée (mots-clés, catégorie, ville/région, distance).
* Filtres par type d’annonce, budget, langue.
* Pages profil entreprises / portfolio.

## Création et gestion d’annonces

* Assistants de création d’annonce (wizard) : titre, description, catégorie, média (image/audio/vidéo), call-to-action (numéro, site, message).
* Téléversement multiple et gestion des médias (thumbnails, extrait audio).
* Options de mise en avant : promotion en home, bannière, push, zone premium.
* Gestion de campagne (durée, budget, ciblage géographique).

## Formats supportés

* Image / infographie (formats JPG/PNG, WebP).
* Vidéo (MP4 — avec transcodage/resizing + poster).
* Audio (MP3) — ex : jingles/publicités radio.
* Publi-reportage (article + multimédia incorporé).

## Monétisation & paiement

* Forfaits : Gratuit (listing basique), Standard (payant), Premium (featured).
* Paiement en ligne (Stripe/PayPal) + options locales (mobile money selon besoin).
* Facturation & historique paiements.

## Analytics & reporting

* Vues, clics, CTR, interactions, conversions (messages/contacts).
* Rapports téléchargeables (.csv, PDF).

## Communauté / Communication

* Messagerie entre annonceur et client (inbox).
* Avis & évaluations (modération).
* Partage social / intégration réseaux.

## Back-office / Admin

* Modération d’annonces (validation, refus, signalements).
* Dashboard finance / utilisateurs.
* Outils anti-fraude et vérification d’identité optionnelle.

---

# 4. MVP (Minimum Viable Product) — ce qu’on livre en priorité

* Authentification (utilisateur + entreprise).
* Création d’annonce image + listing public.
* Recherche simple & filtres par catégorie + localisation.
* Tableau de bord annonceur (liste annonces, statistiques basiques : vues, contacts).
* Paiement en ligne (un moyen de paiement).
* Modération admin simple.

---

# 5. Architecture technique recommandée

(je sais que tu utilises Django/Tailwind — je me base là-dessus)

### Stack recommandé

* Backend : **Django** + **Django REST Framework** (APIs).
* Frontend : **Vue 3 (Composition API)** + **Tailwind CSS** (+ Flowbite si voulu).
* Base de données : **PostgreSQL**.
* Stockage média : **AWS S3** ou **Cloudinary** (recommandé pour gestion vidéos + transformations).
* Traitement vidéo/audio : **FFmpeg** (workers Celery) ou services Cloud (Elastic Transcoder).
* Tâches asynchrones : **Celery + Redis**.
* Cache & sessions : **Redis**.
* Recherche / filtrage avancé : **Elasticsearch** ou **Postgres full-text** (démarrer par Postgres).
* Auth & paiement : JWT/DRF token + **Stripe** / fournisseurs locaux si besoin.
* CDN pour médias (CloudFront / Cloudflare).

---

# 6. Modèles de données (exemples Django — schéma simplifié)

```python
from django.db import models
from django.contrib.auth import get_user_model

User = get_user_model()

class Business(models.Model):
    owner = models.ForeignKey(User, on_delete=models.CASCADE)
    name = models.CharField(max_length=200)
    slug = models.SlugField(unique=True)
    description = models.TextField(blank=True)
    city = models.CharField(max_length=100, blank=True)
    country = models.CharField(max_length=100, default='Central African Republic')
    verified = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)

class Category(models.Model):
    name = models.CharField(max_length=100)

class Ad(models.Model):
    TYPE_CHOICES = [
        ('image','Image'),
        ('video','Video'), 
        ('audio','Audio'), 
        ('publi','Publi-reportage')
    ]

    business = models.ForeignKey(Business, on_delete=models.CASCADE)
    title = models.CharField(max_length=255)
    slug = models.SlugField(unique=True)
    ad_type = models.CharField(max_length=10, choices=TYPE_CHOICES)
    category = models.ForeignKey(Category, on_delete=models.SET_NULL, null=True)
    description = models.TextField()
    budget = models.DecimalField(max_digits=10, decimal_places=2, null=True, blank=True)
    is_featured = models.BooleanField(default=False)
    created_at = models.DateTimeField(auto_now_add=True)
    expires_at = models.DateTimeField(null=True, blank=True)

class MediaAsset(models.Model):
    ad = models.ForeignKey(Ad, related_name='media', on_delete=models.CASCADE)
    file = models.FileField(upload_to='ads/')
    kind = models.CharField(max_length=10)  # thumbnail, video, audio, image
    duration = models.IntegerField(null=True, blank=True)  # seconds for audio/video
    created_at = models.DateTimeField(auto_now_add=True)

class AdAnalytics(models.Model):
    ad = models.ForeignKey(Ad, on_delete=models.CASCADE)
    date = models.DateField()
    views = models.IntegerField(default=0)
    clicks = models.IntegerField(default=0)
```

Si tu veux, je peux détailler 10 endpoints DRF basiques pour ces modèles.

---

# 7. UX / Wireframes (pages clés)

* Landing (value proposition, CTA pour publier une annonce).
* Catalogue / listing (cards d’annonces avec preview).
* Page annonce (player vidéo/image, contact, bouton “Contacter”, statistiques).
* Création d’annonce (wizard en étapes : infos → média → ciblage → paiement).
* Dashboard annonceur (campagnes + analytics).
* Dashboard admin (modération).

Je peux livrer un prototype Tailwind/Vue pour la page d’accueil et la page de création d’annonce.

---

# 8. Branding — couleurs, logo, nom

* Nom : **Songo Beafrika** (déjà choisi — très bien).
* Couleurs : bleu / blanc. Proposition de palette :

  * Bleu principal : `#0B64D3` (bleu profond)
  * Bleu secondaire : `#1E90FF` (accent clair)
  * Fond / blanc : `#FFFFFF`
  * Gris clair (UI) : `#F3F4F6`
* Typographie : Poppins / Inter (moderne & lisible).
* Tagline possibles :

  * « Visibilité locale. Impact global. »
  * « Promouvoir. Connecter. Grandir. »
* Concepts de logo (3 idées) :

  1. Lettre “S” stylisée formant une silhouette de carte d’Afrique (marque + symbole).
  2. Icône « mégaphone » + motif de vague sonore en bleu + texte “SONGO BEAFRIKA”.
  3. Emblème circulaire avec pin de localisation + motif africain minimaliste — texte à droite.
     Je peux générer 3 propositions graphiques si tu veux.

---

# 9. Types d’offres publicitaires (exemples — 3 packages)

1. **Listing Gratuit** — publication texte + 1 image, visibilité standard.
2. **Standard** — images illimitées, 1 vidéo courte, boost local (algorithme) + analytics.
3. **Premium / Publi-reportage** — vidéo longue, mise en avant homepage, bannière, rapport détaillé + assistance créative.

(Chaque package peut être tarifé au forfait ou au CPM/CPM+flat fee.)

---

# 10. Modération & conformité

* Workflow de validation (auto/manuel).
* Outils anti-abuse (rate limit, detection spams).
* Règles de contenu (politique publicitaire) et formulaire d’appel.
* Option KYC pour annonceurs vérifiés.

---

# 11. KPIs & indicateurs de succès

* Nombre d’annonceurs actifs.
* Vues totales / CTR moyen.
* Revenu mensuel récurrent (MRR).
* Taux de conversion annonce → contact.
* Taux de rétention annonceurs (renewals).

---

# 12. Recommandations opérationnelles

* Démarrer MVP sur un seul pays / langue (français) puis ajouter l’anglais/portuguais.
* Prioriser la qualité média : vidéos compressées + previews, et décharger via CDN.
* Prévoir contrats / mentions légales et politique de confidentialité (surtout pour paiements).
* Penser partenariats locaux (chambres de commerce, radios) pour adoption.

---

# 13. Livrables que je peux te fournir tout de suite (choisis une ou plusieurs options)

1. Arborescence détaillée des modèles Django + migrations (code prêt).
2. 10 endpoints DRF (CRUD + search + analytics).
3. Maquette responsive (Tailwind + Vue) de la landing page + page création annonce.
4. 3 concepts de logo (descriptions + fichiers SVG si tu veux que je génère).
5. Plan marketing & onboarding annonceurs (email templates, messages WhatsApp, pitch).

Dis-moi ce que tu veux que je fasse en premier (par ex. « donne-moi les modèles Django et endpoints DRF ») — je te les fournis dans le prochain message.

---

Si tu veux, je commence tout de suite par **les modèles Django + endpoints DRF** (schema + exemples de serializers et vues) ou par **une maquette Tailwind** — dis-moi lequel tu préfères et je le fournis immédiatement.
