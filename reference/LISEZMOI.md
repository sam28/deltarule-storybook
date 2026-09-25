# Références de la maquette

Ces captures tiennent lieu de **source de vérité** pour la comparaison de
Panorama, en attendant que les pages pointent vers de vraies URL Figma (voir
`A-FAIRE.md`).

## Le principe, et il n'est pas négociable

Le site audité vit dans `template/html/`. Il porte **six écarts volontaires**
injectés par `template/html/_build/injecte_ecarts_dashboard.py` — un par
catégorie, tous marqués `<!-- ECART VOLONTAIRE n -->` dans le HTML.

**La maquette est le site AVANT injection.** Le design fait foi, le code s'en
écarte : c'est le sens même du produit. Capturer la référence depuis le site
tel qu'il est déployé donnerait une maquette identique à la production, et
l'auto-diagnostic n'aurait plus rien à trouver.

## Comment les régénérer

1. Partir de `template/html/`, et produire une copie où les six écarts sont
   **inversés** : les écarts 1, 4, 5 et 6 sont de simples substitutions ; les
   écarts 2 (badge supprimé) et 3 (bouton remplacé par un badge) sont
   destructifs, leurs fragments d'origine se relisent dans le commit
   `54fc7a7^`.
   Contrôle : `grep -c "ECART VOLONTAIRE"` doit rendre **0**.
2. Servir cette copie (`python3 -m http.server`). **Ne pas capturer depuis
   `acme-template.pages.dev`** : ce domaine sert le site AVEC ses écarts.
3. Capturer en Playwright aux largeurs de `LARGEUR_APPAREIL`
   (`appareil-selecteur.tsx`) — **1440 / 768 / 375** — en
   `deviceScaleFactor: 2`, `fullPage: true`, attente `networkidle` + 450 ms.
4. Seule `mobile/dashboard-menu-deploye.png` est prise avec `?menu=open` :
   c'est la variante. Toutes les autres pages sont dans leur état par défaut.

**SI CES LARGEURS CHANGENT, ces captures sont à refaire.** Une référence rendue
à une autre largeur que la production simulée compare deux mises en page
différentes : un bloc passe sur deux lignes d'un côté et pas de l'autre, et le
diagnostic le signale comme un écart qui n'existe pas.

## Pourquoi elles sont servies par l'application

Elles vivaient sur `acme-reference.pages.dev`, un déploiement externe. Servies
depuis `public/`, elles sont **same-origin** : `reference-pixels.ts` lit leurs
pixels dans un `<canvas>` pour comparer les couleurs, et sur un hôte tiers sans
en-tête CORS la lecture lève une `SecurityError` qui désactive la règle
« couleur » **sans que rien ne le signale**.

Les anciennes captures de `template/captures/` ont été retirées : elles étaient
aux mauvaises largeurs (tablette 834 au lieu de 768, mobile 390 au lieu de 375)
et le menu overlay y était ouvert sur toutes les pages mobiles.
