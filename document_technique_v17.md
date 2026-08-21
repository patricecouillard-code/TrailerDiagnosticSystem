# Document Technique et de Conception
## Testeur/Diagnostiqueur Universel Véhicule ↔ Remorque

**Version :** 17.0 — Bloc 3 (canal TAIL) détaillé au complet : sous-blocs VEHICLE/TRAILER/Bloc de Test, chaîne de mesure TRAILER indépendante (Option 2), logique à diodes du Bloc de Test avec références KiCad réelles, correction d'orientation des diodes des portes ET
**Date :** Juillet 2026

---

## 📐 RÈGLES DE CONCEPTION (à appliquer en tout temps)

> Ce bloc regroupe toutes les règles de conception établies au fil du projet. Chaque nouvelle règle décidée sera ajoutée ici, à cet endroit, dans l'ordre où elle a été établie. Ces règles ont préséance sur toute autre considération (coût, esthétique, simplicité) sauf mention contraire explicite dans la règle elle-même.

**R1 — 100 % analogique.** Aucun microcontrôleur, aucun composant programmable, aucun firmware. Toute la logique (commutation, détection, indication, gestion de charge) doit être réalisée avec des composants discrets et des circuits intégrés analogiques câblés.

**R2 — Robustesse avant élégance.** Entre deux solutions possibles, on retient toujours celle qui est la plus fiable, robuste et efficace en tension/courant/résistance — même si une autre option est plus simple, plus compacte ou moins chère — sauf si l'option la plus simple n'entraîne aucun compromis de fiabilité.

**R3 — Test d'utilité.** Si une idée surgit, il faut se poser la question : est-ce que la composante ou la fonctionnalité qu'on veut ajouter est utile? Si la réponse est non, on l'oublie.

**R4 — Qualité des pièces.** Sélectionner des pièces de qualité, robustes. À moins que le coût soit exorbitant, c'est ce qu'on choisit.

**R5 — Format de montage.** Composants traversants (« through-hole ») partout — plus faciles à souder et à réparer à la main, plus robustes mécaniquement. **Exception unique et documentée : le LM74610** (contrôleur diode idéale, §6.1/§9), qui n'existe nativement qu'en boîtier SMD chez le fabricant — conservé tel quel plutôt que remplacé par des diodes Schottky traversantes, la perte de tension supplémentaire (~0,3-0,5 V) de l'alternative traversante n'étant pas justifiée ici puisque le LM74610 ne demande aucune modification du reste du circuit.

**R6 — Précision du type d'étiquette (net label).** Chaque fois qu'une étiquette est proposée pour le schéma KiCad, son type doit être spécifié explicitement : **locale** (Label — valide seulement à l'intérieur de la feuille où elle est posée), **globale** (Global Label — se connecte automatiquement partout dans le projet, peu importe la feuille), ou **hiérarchique** (Hierarchical Label — crée une patte sur le rectangle parent, nécessite un fil tracé entre feuilles sur la page racine).

**R7 — Socket pour puces logiques/analogiques.** Toute puce logique ou analogique susceptible de nécessiter un remplacement ou un ajustement en cours de mise au point (comparateurs LM339/LM324, driver LM3914, etc.) doit être montée sur un **socket de qualité « turned-pin » / à broches tournées** plutôt que soudée directement — évite le risque de dommage au PCB lors d'un remplacement, sans sacrifier la fiabilité (contrairement à un socket bon marché à lame). **Exceptions :** MOSFET de puissance (TO-220, déjà remplaçables/boulonnables) et LM74610 (SMD, exception déjà documentée à R5).

---

## 💡 SUGGESTIONS ET IDÉES (registre de suivi)

> Toute idée — venant de l'utilisateur ou de Claude — est consignée ici avec son statut. Une idée reste « En attente » tant qu'elle n'a pas été explicitement tranchée.

| Idée | Proposée par | Statut |
|---|---|---|
| Confirmation lumineuse de la position des switchs MODE et SOURCE | Utilisateur | ❌ Refusée — trop de DEL supplémentaires pour une information déjà visible sur un switch mécanique |
| Jauge de batterie à échelle (5 DEL) plutôt qu'une seule DEL d'état | Utilisateur | ✅ Acceptée — voir §7 et §9 (LM3914 + bargraph 5 segments) |
| Support/pied pour positionner le testeur à angle | Utilisateur | ⏳ En attente — à valider à l'étape boîtier |
| Buzzer actif plutôt que passif (correction — choix final : actif) | Utilisateur | ✅ Acceptée — voir §6.5 et §9 |
| Test de résistance de masse (plutôt qu'une simple détection de coupure) pour le voyant MASSE | Claude | ✅ Acceptée — voir §6.5 pour l'explication détaillée et la distinction avec les canaux de test |
| Socket « turned-pin » pour les puces logiques/analogiques remplaçables (LM339, LM324, LM3914) | Utilisateur | ✅ Acceptée — voir R7 |
| Optocoupleur double (PC817-2, DIP-8) au lieu d'un PC817 simple par canal, pour isoler les 2 seuils (ouvert + court) sans perte d'information et sans agrandir le boîtier | Claude | ✅ Acceptée — voir §6.2, Bloc 3, et Annexe A.3 (mise à jour) |
| Chaîne de mesure de courant TRAILER indépendante de celle du VEHICLE (Option 2 — plus simple à câbler/dépanner, au prix de comparateurs/potentiomètre en double) plutôt qu'une chaîne partagée | Utilisateur | ✅ Acceptée — voir §7.2, Annexe A.3 (mise à jour) |
| Renommer le sous-bloc « Affichage » en « **Bloc de Test** » | Utilisateur | ✅ Acceptée — appliqué à partir de §7.2 |

---

## 1. Objectif du projet

Concevoir un appareil portable permettant de diagnostiquer le circuit électrique d'attelage, capable de fonctionner dans deux directions :

- **Mode « Véhicule »** : l'appareil se branche sur le connecteur du véhicule (celui qui accueille normalement la remorque) et **simule une remorque**, afin de vérifier que le véhicule envoie correctement chaque signal (feux de position, clignotants, stop, recul, charge auxiliaire, freins électriques).
- **Mode « Remorque »** : l'appareil se branche sur le connecteur de la remorque et **simule le véhicule**, afin de vérifier que le câblage et les ampoules/DEL de la remorque fonctionnent, détecter les circuits ouverts, courts-circuits ou masses défectueuses.

L'appareil doit supporter les trois familles de connecteurs nord-américains les plus courantes (4, 5 et 7 broches), fonctionner sur pile interne rechargeable ou sur une batterie 12 V externe, et intégrer des protections empêchant toute injection de courant indésirable vers le véhicule ainsi que des protections internes contre les défauts électriques.

Ce projet suit en tout temps les **Règles de conception** énoncées en début de document (R1-R4), notamment la contrainte fondamentale R1 (100 % analogique, sans microcontrôleur ni firmware) et R2 (robustesse avant élégance).

---

## 2. Exigences fonctionnelles (cahier des charges)

| # | Exigence | Détail |
|---|----------|--------|
| F1 | Sélecteur de mode | Interrupteur DPDT manuel (VEHICLE/TRAILER), décision fiable et non ambiguë ; détection de branchement conservée en simple vérification de cohérence (alerte si incohérence, §6.4) |
| F2 | Connecteurs supportés | 4 broches (plat, SAE), 5 broches (plat), 7 broches (rond/lame, RV/SAE J560), via câbles adaptateurs externes depuis les ports GX16-10 |
| F3 | Alimentation double | Batterie interne rechargeable (Li-ion/LiFePO4) **et** entrée batterie externe 12 V (pinces) |
| F4 | Basculement d'alimentation | Sélection automatique ou manuelle entre interne/externe, sans coupure lors du branchement |
| F5 | Protection du véhicule | Aucune injection de courant incontrôlée vers les modules électroniques du véhicule (BCM, module de remorque) ; la coupure en cas de défaillance doit être **ultra rapide** (commutation à semi-conducteurs, pas de relais mécanique sur ce chemin précis) |
| F6 | Auto-protection | Protection contre inversion de polarité, surtension, surintensité, court-circuit, transitoires (load dump) |
| F7 | Détection de défaut | Distinction : circuit ouvert / charge normale / court-circuit, par canal |
| F8 | Interface utilisateur | DEL RGB par fonction (masse, position, gauche, droite, stop, recul, frein élect./12V aux.) selon code couleur Bleu/Vert/Violet/Rouge (§5.0) — aucun écran piloté par logiciel, sauf module d'affichage V/A fermé |
| F9 | Robustesse | Boîtier résistant aux intempéries (utilisation en atelier/extérieur) |
| F10 | Conception analogique | Voir Règle de conception R1 |
| F11 | Robustesse | Voir Règle de conception R2 |

---

## 3. Brochages des connecteurs à supporter

> Ces brochages sont des standards de l'industrie automobile/RV nord-américaine ; certaines variantes existent selon les fabricants — l'appareil doit rester adaptable par simple changement de câble ou de petit commutateur mécanique pour s'ajuster aux variantes régionales (aucune configuration logicielle, conformément à la contrainte F10).

### 3.1 Connecteur plat 4 broches (SAE, standard voiture/utilitaire léger)

| Broche | Couleur usuelle | Fonction |
|--------|------------------|----------|
| 1 | Blanc | Masse (retour commun) |
| 2 | Brun | Feux de position / plaque |
| 3 | Jaune | Clignotant / stop gauche |
| 4 | Vert | Clignotant / stop droit |

### 3.2 Connecteur plat 5 broches

Identique au 4 broches, avec un 5ᵉ fil dont la fonction **varie selon le fabricant** : frein électrique, feu de recul, ou 12 V auxiliaire (charge batterie remorque). L'appareil doit permettre de **réassigner** ce canal via un petit commutateur mécanique (rotatif ou à glissière) qui reroute physiquement le canal interne concerné — aucune configuration logicielle.

| Broche | Couleur usuelle | Fonction |
|--------|------------------|----------|
| 1 | Blanc | Masse |
| 2 | Brun | Feux de position |
| 3 | Jaune | Gauche |
| 4 | Vert | Droit |
| 5 | Bleu / Noir / Violet | Configurable (frein élect. / recul / 12V aux.) |

### 3.3 Connecteur 7 broches (rond ou à lame « RV Blade », SAE J560)

| Broche | Couleur usuelle | Fonction |
|--------|------------------|----------|
| 1 | Blanc | Masse |
| 2 | Noir | 12 V auxiliaire (charge batterie remorque) |
| 3 | Bleu | Frein électrique |
| 4 | Jaune | Gauche |
| 5 | Vert | Droit |
| 6 | Brun | Feux de position |
| 7 | Violet / Rouge | Feu de recul |

**Choix d'implémentation retenu :** deux connecteurs physiques **GX16-10** (12 broches, robustes, à verrouillage par rotation) sur le panneau avant — un **« TRAILER » (femelle)** et un **« VEHICLE » (mâle)** — plutôt qu'un seul port partagé. Chaque canal électronique interne (masse, position, gauche, droite, stop, recul, frein élect./12V aux.) est câblé une fois vers ces deux connecteurs GX16-10 (la protection §6.2 étant commune aux deux, seul le sens du courant change selon le port utilisé). Des **câbles adaptateurs externes** GX16-10 → 4 broches plat, GX16-10 → 5 broches plat, et GX16-10 → 7 broches (rond/lame) permettent de couvrir les trois standards sans multiplier les prises exposées aux intempéries sur le boîtier lui-même. Ce choix élimine aussi le besoin d'un relais/commutateur interne pour inverser le sens d'un seul port — chaque port a un rôle fixe et non ambigu.

**Utilisation des broches libres :** sur les 10 broches du GX16-10, 8 sont utilisées (masse + 6 canaux + 1 broche de détection, voir §6.4 — vérification de cohérence), ce qui laisse **2 broches libres par connecteur** en marge pour un besoin futur (ex. un canal supplémentaire), sans devoir redessiner le connecteur ni la découpe de façade.

---

## 4. Architecture générale (100 % analogique)

Sans MCU, la « logique » de l'appareil est distribuée dans plusieurs blocs autonomes qui communiquent uniquement par tensions/courants analogiques. Il n'y a pas de « cerveau » central : un **switch de mode manuel (DPDT)** arme physiquement l'un ou l'autre des deux ports, et un petit circuit de **vérification de cohérence** (basé sur les broches de détection) surveille que le câble branché correspond bien au switch.

```
┌───────────────┐        ┌──────────────────────┐
│ Bloc            │        │ Switch MODE (DPDT)     │
│ Alimentation    │───────▶│ manuel : VEHICLE /      │
│ (interne+externe│        │ TRAILER — arme un       │
│  + OR-ing diode)│        │ seul côté à la fois      │
└───────────────┘        └──────────┬─────────────┘
                                        │
                    ┌───────────────────┴───────────────────┐
                    ▼                                         ▼
┌──────────────────────────────────┐              ┌──────────────────────────────────┐
│  Port VEHICLE (GX16-10, mâle)      │              │  Port TRAILER (GX16-10, femelle)   │
│  lecture seule : résistance de      │              │  source : alimentation commandée    │
│  charge + shunt + comparateur +      │              │  par bouton-poussoir à verrouillage  │
│  optocoupleur (isolement)             │              │  par canal + PTC + TVS               │
│  → DEL RGB (§5.0)                       │              │  → DEL RGB (§5.0)                      │
└──────────────────┬─────────────────┘              └─────────────────┬─────────────────┘
                     │                                                   │
                     │            Broche de détection (par port)          │
                     └───────────────────┐         ┌───────────────────┘
                                            ▼         ▼
                                  ┌──────────────────────────┐
                                  │  VÉRIFICATION DE COHÉRENCE   │
                                  │  (logique à diodes + comparateur)│
                                  │  Switch en position X, mais       │
                                  │  câble détecté côté Y → DEL        │
                                  │  d'alerte « INCOHÉRENCE »            │
                                  └──────────────────────────┘
```

**Principe clé :** le switch MODE décide, mécaniquement et de façon fiable, quel port est réellement armé — c'est la seule source de vérité pour "dans quel mode suis-je". La broche de détection de chaque port ne sert plus à décider quoi que ce soit : elle sert uniquement à vérifier, en arrière-plan, que le câble branché correspond au choix du switch, et à alerter si ce n'est pas le cas (ex. switch en position VEHICULE mais câble branché côté TRAILER). Chacun des 7 canaux possède son **propre circuit de protection/mesure analogique identique** (voir §6.2). Il n'y a aucune donnée numérique échangée entre les blocs — uniquement des niveaux de tension et de courant.

---

## 5. Fonctionnement par mode

### 5.0 Code couleur de référence (règle officielle du projet)

Chaque canal (position, gauche, droite, stop, recul, frein élect./12V aux.) possède **une seule DEL RGB**, mais chacune de ses trois couleurs internes (Rouge, Vert, Bleu) est allumée « à fond ou éteinte » par son propre petit interrupteur électronique (transistor), jamais à intensité variable. Ce choix garantit que les couleurs combinées (comme le violet) sont **toujours identiques et reproductibles**, sans dérive — c'est la solution retenue en application de la règle F11 (robustesse avant tout, sans sacrifier l'élégance quand elle ne coûte rien en fiabilité).

| État | Couleur affichée | Diodes allumées | Signification |
|---|---|---|---|
| Prêt / au repos | **Bleu** | Bleu seul | L'appareil est sous tension, le canal est prêt ; ne s'affiche qu'en mode Remorque, bouton relâché (OFF) |
| Fonctionnement normal | **Vert** | Vert seul | Courant mesuré dans la plage attendue — tout fonctionne |
| Circuit ouvert | **Violet** | Rouge + Bleu ensemble | Le canal est actif (bouton enclenché ou véhicule en train d'envoyer le signal) mais aucun courant ne circule — fil coupé, ampoule grillée, mauvais contact |
| Court-circuit | **Rouge** | Rouge seul | Courant excessif détecté — danger, priorité d'affichage absolue sur toutes les autres couleurs |

Le **rouge pur** est délibérément réservé au cas le plus dangereux (court-circuit), afin qu'un coup d'œil rapide au panneau suffise à repérer le problème le plus urgent parmi plusieurs canaux.

### 5.1 Mode « Véhicule » — le connecteur GX16-10 « VEHICLE » (appareil = charge simulée, entièrement passif)

L'appareil se branche sur le connecteur du véhicule via le port **VEHICLE**. Ce côté est **passif** : aucun bouton-poussoir n'est nécessaire, l'appareil ne fait qu'observer ce que le véhicule envoie, sur les mêmes DEL que le côté Remorque.

1. Chaque canal est en permanence connecté à une **résistance de charge fixe ou commutable** (simulant une ampoule incandescente ~2-3 Ω à froid, ou une valeur équivalente DEL réglable par petit commutateur), pour que le véhicule ne signale pas d'erreur « circuit ouvert » — un problème fréquent avec les véhicules modernes lorsqu'une remorque réelle utilise des feux DEL à faible consommation.
2. Un **shunt + comparateur analogique** (ampli opérationnel ou comparateur type LM339/LM324) mesure le courant réel dans la résistance de charge dès que le véhicule active la fonction (ex. appui sur la pédale de frein) et pilote directement la DEL du canal selon le code couleur du §5.0 : **Vert** si le courant est normal, **Violet** si le véhicule n'envoie rien (ou trop peu), **Rouge** si un défaut de câblage interne provoque un courant anormalement élevé.
3. La ligne de mesure passe par une **isolation galvanique passive** (optocoupleur simple, sans composant actif programmable), garantissant qu'aucun courant ne peut être renvoyé activement vers le véhicule — c'est la protection centrale de l'exigence F5.
4. La masse est vérifiée par un pont diviseur simple entre la masse de l'appareil et la broche masse du véhicule.

**Point clé de sécurité :** côté véhicule, l'appareil ne fournit **jamais** de tension active sur les lignes de signal — il ne fait que lire. Comme les deux ports (VEHICLE et TRAILER) sont des connecteurs physiquement distincts, il n'y a aucune ambiguïté possible sur le sens du courant : le port VEHICLE n'est câblé que pour recevoir, jamais pour émettre.

### 5.2 Mode « Remorque » — le connecteur GX16-10 « TRAILER » (appareil = source simulée, boutons à verrouillage)

L'appareil se branche sur le connecteur de la remorque via le port **TRAILER**. La batterie de l'appareil (interne ou externe, via l'étage de protection §6.1) alimente chaque canal à travers son étage de protection dédié (§6.2), commandée par un **bouton-poussoir à verrouillage** (type interrupteur à bascule ou bouton-poussoir « push-push » qui reste enclenché en position ON jusqu'à ce qu'on le rappuie) :

1. **Plusieurs canaux peuvent être activés en même temps** (ex. TAIL + BRAKE + LEFT TURN simultanément), puisque chaque bouton reste enclenché indépendamment des autres — utile pour observer les interactions entre circuits (ex. vérifier que le clignotant gauche continue de clignoter même quand les feux de stop sont actifs).
2. Une fois un bouton enclenché, le canal correspondant alimente la broche remorque à travers un **fusible réarmable (PTC)** dimensionné légèrement au-dessus du courant nominal attendu (ex. ~3-5 A par canal selon le nombre d'ampoules).
3. Le courant est surveillé par un **circuit à deux seuils** (un seuil bas, un seuil haut) qui pilote la DEL du canal selon le code couleur du §5.0 : **Bleu** tant que le bouton est relâché, **Vert** si le courant est dans la plage normale une fois le bouton enclenché, **Violet** si aucun courant ne circule malgré le bouton enclenché (circuit ouvert), **Rouge** si le courant dépasse le seuil haut (court-circuit) — le PTC coupe alors automatiquement l'alimentation du canal, protection physique et immédiate, indépendante de l'affichage.
4. La masse de la remorque est testée en continuité vers la masse de l'appareil.

---

## 6. Protections

C'est la partie critique du projet : protéger le véhicule (exigence F5) et protéger l'appareil lui-même (exigence F6).

### 6.1 Étage d'alimentation (protection de l'appareil)

- **Entrée batterie externe (pinces) :**
  - Fusible en ligne réarmable ou lame (10-15 A) au plus près des pinces.
  - Protection anti-inversion de polarité par MOSFET « diode idéale » (préférable à une simple diode, pour limiter les pertes) ou, à minima, diode Schottky + fusible sacrificiel.
  - Diode TVS (transil) bidirectionnelle en tête d'entrée pour absorber les transitoires du circuit véhicule (« load dump », coupures d'inductance selon ISO 7637-2), typiquement dimensionnée pour un pic > 40 V.
- **Batterie interne rechargeable :**
  - **Pack Li-ion 3 cellules 18650 en série (3S1P)**, ex. **Panasonic NCR18650B** (3350 mAh) — cellule de marque reconnue, très fiable, largement utilisée en industrie (outils électriques, etc.), format compact et standard. Trois cellules en série donnent ~11,1-12,6 V, cohérent avec le rail 12 V de l'appareil.
  - **Connecteur non-inversable** entre le pack et le circuit principal : **JST-PH 2 broches** (boîtier à clé/« keyed », empêche physiquement une insertion inversée) — standard répandu sur les packs Li-ion.
  - Module de protection de batterie (BMS) intégré, dimensionné pour 3S : coupure sur surcharge, décharge profonde, surintensité, court-circuit, et équilibrage des cellules.
  - Circuit de charge dédié (via port USB-C PD ou prise barrel), avec indicateur d'état de charge.
- **Sélection manuelle de source (interne/externe) — switch SOURCE :** un commutateur **DPDT à 3 positions (ON-OFF-ON) : INTERNE / OFF / EXTERNE**. La position centrale OFF coupe entièrement l'appareil (aucune source connectée), utile pour le rangement/transport sans user la batterie interne. Ce switch et le switch MODE (§6.3) sont tous les deux manuels et indépendants l'un de l'autre.
- **Anti-inversion de polarité (protection basse perte) :** un seul contrôleur « diode idéale » type **LM74610** (piloté avec son MOSFET externe, §9/Annexe A), placé **après** le switch SOURCE, sur la source déjà sélectionnée. *(Correction : pas de OR-ing entre 2 sources ici — le switch SW1 (ON-OFF-ON) garantit déjà qu'une seule source est connectée à la fois, ce qui rend un OR-ing entre deux LM74610 redondant. Le rôle du LM74610 restant est uniquement la protection anti-inversion, avec une perte de tension minime comparée à une diode classique.)*
- **Régulation :** régulateurs linéaires ou à découpage classiques (rails fixes, ex. 78xx/LM2596 ou équivalent) pour générer les tensions de référence nécessaires aux comparateurs, isolés du rail de puissance 12 V des canaux de test.

### 6.2 Étage de protection par canal (protection du véhicule ET de l'appareil)

Chacun des canaux (masse, position, gauche, droite, stop, recul, frein électrique, 12 V auxiliaire) traverse le même étage de protection générique, entièrement câblé :

1. **Isolement de direction (VEHICLE vs TRAILER) :** contrairement à un port unique partagé, cette isolation est ici **native au câblage** : le port **VEHICLE** n'est physiquement câblé que pour la lecture (résistance de charge + comparateur), et le port **TRAILER** n'est câblé que pour l'alimentation via bouton-poussoir. Il n'y a donc pas de relais à faire basculer entre deux sens — chaque port a un rôle fixe, ce qui supprime un point de défaillance possible. Sur le port VEHICLE, la ligne de mesure passe tout de même par un pont diviseur résistif de forte valeur suivi d'un **optocoupleur passif**, garantissant qu'**aucun courant significatif ne peut remonter vers le véhicule**, même en cas de défaillance d'un composant en aval.
2. **Fusible réarmable (PTC) :** limite le courant de sortie par canal en mode Remorque (protection contre court-circuit côté remorque) et protège l'appareil en toutes circonstances, par une propriété physique du composant (résistance qui augmente avec la température), sans aucune détection active nécessaire.
3. **Diode TVS par canal :** absorbe les surtensions transitoires provenant du véhicule ou de la remorque (débranchement en charge, étincelles de contact).
4. **Mesure de courant (shunt + comparateur analogique) :** un ampli opérationnel ou comparateur (ex. LM339/LM324) compare la tension aux bornes du shunt à des seuils de référence fixés par ponts diviseurs, pilotant directement les DEL d'indication décrites au §5 — aucune conversion analogique-numérique.
5. **Protection thermique :** simple thermistance (CTN) en série ou en surveillance sur le boîtier des composants de puissance, câblée pour couper l'alimentation (via un relais ou un thyristor de protection) en cas d'échauffement anormal — protection physique, sans logique programmée.

**Pourquoi deux couches de protection contre l'injection, alors que le risque est déjà très faible :** grâce aux connecteurs mâle (VEHICLE) / femelle (TRAILER) distincts, un câble d'un côté ne peut physiquement pas se brancher sur l'autre — ce qui élimine déjà la quasi-totalité du risque lié à une erreur humaine ou à un mauvais câblage externe. Cette architecture mâle/femelle est en fait la protection la plus fondamentale du système, purement structurelle. Le risque résiduel qui subsiste n'est pas lié au câblage externe, mais à une **défaillance interne d'un composant** (ex. un MOSFET ou l'optocoupleur du point 1 qui tombe en panne en mode passant) — un scénario rare, mais impossible à éliminer complètement pour un appareil électronique, peu importe la qualité du câblage. La détection active + coupure ultra rapide par MOSFET (§6.5) sert spécifiquement de filet de sécurité pour ce cas résiduel, en application du principe de « défense en profondeur » : ne jamais dépendre d'une seule barrière quand les conséquences d'un échec (endommager un module électronique de véhicule) sont sérieuses.

### 6.3 Switch de mode manuel (DPDT) — décision fiable et sans ambiguïté

**Décision retenue (après réflexion) : le mode est choisi manuellement par un interrupteur DPDT à 2 positions (VEHICLE / TRAILER), pas automatiquement.** C'est le choix le plus robuste : un interrupteur mécanique n'a pas de composant actif pouvant tomber en panne, et sa position répond **instantanément et sans ambiguïté** à la question « dans quel mode suis-je? » — il suffit de regarder le switch.

Ce switch coupe physiquement l'alimentation du côté non sélectionné : en position VEHICLE, seule l'électronique de lecture du port VEHICLE est alimentée (résistances de charge + comparateurs) ; en position TRAILER, seule l'alimentation vers les boutons-poussoirs du port TRAILER est active. Comme il porte un courant modeste (l'électronique de mesure, pas la pleine puissance des feux), un simple interrupteur à bascule ou DPDT de qualité suffit, sans relais intermédiaire — moins de pièces, plus robuste (conforme à la règle R2).

### 6.4 Vérification de cohérence (détection de branchement en renfort, pas en décision)

La broche de détection par port, introduite précédemment (§3.3), est conservée mais **change de rôle** : elle ne décide plus rien, elle sert uniquement de **garde-fou** pour repérer une erreur humaine (ex. switch en position VEHICLE alors qu'un câble est branché côté TRAILER, ou l'inverse).

1. Chaque port a sa broche de détection (reliée à la masse dans le câble adaptateur, avec résistance de tirage côté appareil — mécanisme identique à celui décrit précédemment).
2. Une **logique à diodes très simple** (l'équivalent d'un « ET » analogique, réalisable avec deux diodes et un transistor) compare l'état de détection de chaque port à la position du switch MODE (le switch fournit lui-même le signal de référence de sa position via son deuxième pôle).
3. En cas d'incohérence (câble détecté du côté opposé à la position du switch), une **DEL d'alerte dédiée** (« INCOHÉRENCE » ou réutilisation du concept « ALARM » du panneau) s'allume pour avertir l'utilisateur.
4. Cette vérification est purement informative — elle n'interrompt ni ne modifie l'alimentation. La décision reste toujours celle du switch ; la DEL d'alerte ne fait qu'inviter l'utilisateur à vérifier son branchement.

Ce découpage donne le meilleur des deux mondes : la fiabilité et la clarté d'un interrupteur mécanique pour la décision, et un filet de sécurité passif pour attraper une erreur d'inattention.

### 6.5 Avertissements critiques (rangée du bas du panneau)

Une rangée dédiée du panneau signale les anomalies graves — distinctes des DEL RGB par canal (§5.0), qui reflètent l'état normal d'un test. Un **buzzer actif unique et partagé** (intègre son propre oscillateur — aucun circuit de génération de ton nécessaire, juste un transistor pour l'allumer/l'éteindre) s'active en même temps que la DEL correspondante pour toute anomalie grave, **sauf batterie faible**. Choix simple et compact : moins de composants qu'un buzzer passif (pas besoin d'oscillateur externe), ce qui convient bien à la contrainte de compacité du boîtier. Le buzzer sonne en continu tant que l'anomalie est présente, et s'arrête automatiquement dès qu'elle disparaît (comportement le plus simple et le plus robuste — pas de circuit de mémorisation nécessaire).

| Avertissement | DEL | Buzzer | Coupure d'alimentation |
|---|---|---|---|
| **Inversion de polarité** | ✅ | ✅ | Totale — tout l'appareil |
| **Injection de courant vers le véhicule** | ✅ | ✅ | Partielle — port VEHICLE seulement ; le port TRAILER et le reste de l'appareil continuent de fonctionner |
| **Surtension** | ✅ | ✅ | Totale — tout l'appareil |
| **Court-circuit sur le bus d'alimentation principal** (distinct du court-circuit par canal déjà couvert en §5.0, qui est un résultat de test normal et ne déclenche pas cette rangée) | ✅ | ✅ | À déterminer |
| **Batterie faible** | Segment rouge de la jauge (§7, §9) | ❌ | Aucune — la jauge de batterie fait déjà office d'avertissement visuel, pas de DEL distincte ni de buzzer |
| **Masse (ground)** | ✅ (voir explication ci-dessous) | ❌ | Aucune — DEL seule, purement diagnostique |

**Détection d'injection vers le véhicule (protection ultra rapide, exigence F5) :** un capteur de courant dédié surveille en continu le port VEHICLE (en plus de l'isolement passif par optocoupleur du §6.2, qui reste la première ligne de défense puisqu'aucun chemin actif n'existe structurellement). Toute sortie de courant, même minime, est détectée par un comparateur qui pilote directement un **MOSFET de puissance** (commutateur à semi-conducteurs, sans pièce mobile) plutôt qu'un relais mécanique : la coupure se fait en quelques microsecondes, environ 1000 fois plus vite qu'un relais (qui prendrait plusieurs millisecondes le temps que son bras métallique bouge physiquement). Seul ce port est déconnecté, sans affecter le port TRAILER ni interrompre un test en cours de ce côté.

**Note :** le court-circuit par canal (rouge, §5.0) reste géré uniquement par le PTC du canal concerné, sans buzzer ni coupure globale — il s'agit souvent d'un résultat de test recherché (ex. détecter un fil qui touche la masse), pas d'une urgence pour l'appareil.

**À quoi sert vraiment le voyant MASSE — et pourquoi ce n'est pas la même chose que la masse des 6 canaux de test**

Ce voyant peut porter à confusion s'il n'est pas bien expliqué, parce que la masse est déjà impliquée dans chacun des 6 circuits de test (§5.0) — chaque canal a besoin d'un bon retour de masse pour fonctionner correctement. Voici la distinction :

- **Les 6 canaux de test** vérifient chacun *leur propre fonction* (tail, brake, gauche, droite, recul, aux 12V) — si un canal affiche Violet ou Rouge, le problème est spécifique à ce fil-là.
- **Le voyant MASSE**, lui, est un test **séparé et indépendant des boutons-poussoirs**, qui mesure en continu la **qualité de la connexion de masse elle-même** (pas juste "coupée ou pas", mais "à quel point elle est bonne"). Concrètement : un petit courant de test connu traverse la broche masse, et un comparateur mesure la chute de tension qui en résulte — une résistance de masse anormalement élevée (connexion corrodée, connecteur mal serré, fil trop mince) fait s'allumer ce voyant, **même si aucun fusible n'a sauté et même si les canaux individuels semblent fonctionner**.

**Pourquoi c'est utile malgré tout :** un problème de masse ne fait presque jamais sauter un fusible (ce n'est pas un court-circuit, c'est une résistance excessive) — mais c'est une cause très fréquente de symptômes déroutants sur une remorque : feux qui s'allument faiblement, ou un circuit qui "emprunte" le retour d'un autre par manque de masse propre (ex. le clignotant fait aussi clignoter faiblement le feu de stop). Si ce voyant s'allume, c'est un signal pour l'utilisateur de **vérifier la masse en premier**, avant de chercher l'anomalie ailleurs — ça peut expliquer d'un coup plusieurs symptômes qui semblaient être des problèmes distincts sur différents canaux.

**Comportement retenu** (cohérent avec la sévérité réelle du problème) : DEL seule, sans buzzer ni coupure d'alimentation — c'est un outil de diagnostic, pas une urgence pour l'appareil ou le véhicule, au même titre que la jauge de batterie faible.

---

## 7. Interface utilisateur (100 % câblée, sans écran piloté)

- **Une DEL RGB par canal** (masse, position, gauche, droite, stop, recul, frein élect./12V aux.), partagée entre les deux ports VEHICLE et TRAILER (le canal n'est actif que sur un seul port à la fois selon lequel est branché). Chaque couleur (R/V/B) est pilotée tout-ou-rien par son propre transistor, selon le code couleur officiel du §5.0 (Bleu = prêt, Vert = normal, Violet = circuit ouvert, Rouge = court-circuit).
- **Boutons-poussoirs à verrouillage** (« push-push » ou interrupteurs à bascule), un par fonction, actifs uniquement côté TRAILER — permettent de tester plusieurs circuits simultanément.
- **Aucun bouton nécessaire côté VEHICLE** : ce côté est purement passif, les DEL reflètent directement ce que le véhicule envoie.
- **Commutateur MODE (DPDT, 2 positions : VEHICLE / TRAILER)** — décision manuelle et fiable de quel port est armé ; sa position répond directement à la question « dans quel mode suis-je? » (hypothèse retenue : pas de position OFF centrale sur ce switch, puisque le switch SOURCE ci-dessous fournit déjà une coupure totale de l'appareil — à confirmer si tu préfères l'inverse).
- **DEL d'alerte « INCOHÉRENCE »** : s'allume si un câble est détecté du côté opposé à la position du switch MODE (§6.4) — purement informatif, n'affecte pas l'alimentation.
- **Voltmètre/ampèremètre numérique** (module d'affichage dédié, ex. type panel meter V/A) conservé tel quel sur le panneau — c'est un module d'affichage fermé, pas une logique de décision programmée, donc compatible avec l'esprit F10.
- **Commutateur SOURCE (DPDT, 3 positions ON-OFF-ON : INT./OFF/EXT.)** pour choisir manuellement la batterie interne ou l'alimentation externe, avec position centrale de coupure totale (rangement/transport). Un OR-ing de diodes reste en aval par sécurité (§6.1).
- **Jauge de batterie à 5 segments** (rangée du bas, avec les DEL d'anomalie) : bargraph LED 5 segments en un seul boîtier compact, couleurs **Rouge, Jaune, Jaune, Vert, Vert** (de bas à haut, mode « bar » — les segments s'allument en cascade selon le niveau de charge). Pilotée par une puce **LM3914** (driver analogique dédié, §9), avec un potentiomètre de calibration pour ajuster la plage de tension affichée à la chimie exacte du pack (§6.1).

---

### 7.1 Nomenclature des étiquettes de canal (schéma KiCad — Bloc 3)

Pour éviter toute ambiguïté dans le schéma, les 6 canaux fonctionnels portent des noms d'étiquette fixes, utilisés systématiquement dans KiCad et dans toute discussion technique à partir de maintenant :

| Canal (fonction) | Suffixe d'étiquette | Étiquette côté VEHICLE | Étiquette côté TRAILER |
|---|---|---|---|
| Feux de position | `TAIL` | `VEH_TAIL` | `TRAIL_TAIL` |
| Frein (feux de stop) | `BRAKE` | `VEH_BRAKE` | `TRAIL_BRAKE` |
| Clignotant gauche | `LEFT` | `VEH_LEFT` | `TRAIL_LEFT` |
| Clignotant droit | `RIGHT` | `VEH_RIGHT` | `TRAIL_RIGHT` |
| Recul | `BACKUP` | `VEH_BACKUP` | `TRAIL_BACKUP` |
| Frein élect./12V aux. | `AUX` | `VEH_AUX` | `TRAIL_AUX` |

**Décision d'architecture de feuille (Bloc 3) :** les 6 canaux sont regroupés sur **une seule feuille hiérarchique** « Canal_Test », reliée à la page racine par un **lien hiérarchique unique**, plutôt que 6 feuilles séparées ou une feuille générique instanciée ×6.

**Décision de câblage inter-feuilles :** les signaux `VEH_x` et `TRAIL_x` de chaque canal (les seuls signaux réellement propres à un canal) transitent de la feuille Canal_Test vers la page racine via un **bus hiérarchique** (un trait groupé par canal, plutôt que 12 étiquettes hiérarchiques individuelles) — choisi pour garder la page racine lisible. La masse (`GND`) et le rail de référence des comparateurs (`V_REF`) restent des **étiquettes globales** communes, partagées par tous les canaux — ce ne sont pas des membres du bus, puisqu'ils ne sont pas propres à un canal en particulier.

**Correction au composant d'isolement (§6.2, §9, Annexe A.3) :** chaque canal nécessite d'isoler **2 signaux distincts** venant des comparateurs côté VEHICLE (seuil bas = circuit ouvert, seuil haut = court-circuit), pas un seul. Un **PC817 simple** (1 canal d'isolement) ne suffit donc pas ; il est remplacé par un **PC817-2** (boîtier DIP-8 traversant, 2 canaux d'isolement dans le même format compact) — aucun agrandissement du boîtier, aucune perte d'information entre les deux défauts. Signaux isolés résultants par canal : `OUVERT_x` et `COURT_x` (étiquettes **locales**, valides à l'intérieur de la feuille Canal_Test, utilisées pour piloter le sous-bloc d'affichage RGB du même canal).

### 7.2 Canal TAIL — modèle détaillé (Bloc 3, référence pour les 5 autres canaux)

> Le canal TAIL sert de modèle complet. Les 5 autres canaux (BRAKE, LEFT, RIGHT, BACKUP, AUX) reprendront exactement la même structure — seule la référence change (ex. `_BRAKE` au lieu de `_TAIL`). **État au moment de cette mise à jour : le sous-bloc Bloc de Test est en cours de câblage (voir « Points en suspens » en fin de section) — pas encore complet.**

**Structure retenue :** le canal TAIL (et chacun des 5 autres) est divisé en **3 sous-blocs** sur la feuille « Canal_Test » :

1. **VEHICLE** — lecture passive (résistance de charge, shunt, comparateurs, isolement PC817-2)
2. **TRAILER** — source active (bouton-poussoir, PTC, MOSFET de sortie, comparateurs)
3. **Bloc de Test** *(renommé — anciennement « sous-bloc Affichage »)* — logique à diodes/transistors qui combine les signaux des deux côtés et pilote la DEL RGB

**Décision — mesure de courant côté TRAILER (Option 2 retenue) :** plutôt que de partager le même shunt/comparateurs entre VEHICLE et TRAILER (ce qui aurait suffi avec les 3 comparateurs déjà prévus à l'Annexe A, puisque les deux côtés ne sont jamais actifs en même temps), une **chaîne de mesure entièrement séparée et indépendante** est utilisée côté TRAILER — plus simple à câbler et à dépanner, au prix de composants supplémentaires (jugé acceptable, cf. directive de l'utilisateur : robustesse/simplicité de dépannage prioritaire tant que ça n'agrandit pas le boîtier).

**Conséquence sur les composants (voir aussi Annexe A.3, mise à jour) :** chaque canal a maintenant **5 comparateurs** (3 côté VEHICLE : ouvert, court, injection ; 2 côté TRAILER : ouvert, court) plutôt que 3, plus un 2ᵉ shunt et un 2ᵉ potentiomètre de calibration dédiés au côté TRAILER.

#### 7.2.1 Nomenclature des signaux internes (canal TAIL)

| Signal | Type d'étiquette | Sous-bloc | Rôle |
|---|---|---|---|
| `VEH_TAIL` | Hiérarchique (membre du bus, §7.1) | VEHICLE ↔ page racine | Fil vers le port physique VEHICLE |
| `TRAIL_TAIL` | Hiérarchique (membre du bus, §7.1) | TRAILER ↔ page racine | Fil vers le port physique TRAILER |
| `MODE_VEHICULE` | Globale (établie au Bloc 2) | Tous | Position du switch MODE = VEHICULE |
| `MODE_TRAILER` | Globale (établie au Bloc 2) | Tous | Position du switch MODE = TRAILER |
| `V_REF` | Globale (établie au Bloc 1) | Tous | Rail régulé de référence, toujours sous tension (12 V), alimente les comparateurs et les portes à diodes |
| `GND` | Globale (établie au Bloc 1) | Tous | Masse commune |
| `COURT_TAIL_Vehic` | Locale | VEHICLE → Bloc de Test | Sortie comparateur seuil haut, côté VEHICLE |
| `OUVERT_TAIL_Vehic` | Locale | VEHICLE → Bloc de Test | Sortie comparateur seuil bas, côté VEHICLE |
| `COURT_TAIL_Trailer` | Locale | TRAILER → Bloc de Test | Sortie comparateur seuil haut, côté TRAILER |
| `OUVERT_TAIL_Trailer` | Locale | TRAILER → Bloc de Test | Sortie comparateur seuil bas, côté TRAILER |
| `BTN_TAIL_ETAT` | Locale | TRAILER → Bloc de Test | État du bouton-poussoir (haut = enclenché) |
| `Court_V_Actif`, `Court_T_Actif` | Locale (nœuds internes) | Bloc de Test | Sorties des portes ET intermédiaires |
| `Ouvert_V_Actif`, `Ouvert_T_Actif` | Locale (nœuds internes) | Bloc de Test | Sorties des portes ET intermédiaires |
| `Court_Tail`, `Ouvert_Tail`, `Bleu_Tail` | Locale (nœuds finaux) | Bloc de Test | Signaux combinés finaux, avant pilotage DEL |
| `Not_Btn_Tail` | Locale (nœud interne) | Bloc de Test | Inversion de `BTN_TAIL_ETAT` |

#### 7.2.2 Composants — sous-bloc VEHICLE (canal TAIL)

| Réf. | Composant | Boîtier |
|---|---|---|
| RL_TAIL | Résistance de charge simulée, ~2-3 Ω | Traversant |
| RSH_TAIL | Résistance shunt, ~0,05-0,1 Ω | Traversant axial |
| U_TAIL_A/B/C | 3/4 de LM339 | DIP-14, socket turned-pin (R7) |
| OPTO_TAIL | PC817-2 | DIP-8 traversant |
| Q_TAIL_INJECT | IRLZ44N | TO-220 |
| POT_TAIL | Potentiomètre de calibration | Traversant, multi-tours |

#### 7.2.3 Composants — sous-bloc TRAILER (canal TAIL)

| Réf. | Composant | Boîtier |
|---|---|---|
| SW_TAIL | Bouton-poussoir push-push (électriquement : symbole `SW_SPST`) | Traversant, étanche |
| Q_TAIL_PWR | IRLZ44N | TO-220 |
| RSH2_TAIL | Résistance shunt dédiée TRAILER (nouvelle, Option 2) | Traversant axial |
| U_TAIL_D/E | 2 comparateurs (nouveau LM339, nouvelle puce) | DIP-14, socket turned-pin (R7) |
| POT2_TAIL | Potentiomètre de calibration dédié TRAILER (nouveau, Option 2) | Traversant, multi-tours |
| PTC_TAIL | Polyswitch | Traversant |
| TVS_TAIL | SMBJ33A | Traversant |

#### 7.2.4 Composants — Bloc de Test (logique d'affichage, canal TAIL)

> Références réelles telles que générées par KiCad (remplacent les références provisoires utilisées en cours de conception).

| Réf. KiCad | Composant | Rôle |
|---|---|---|
| D3, D4 | 1N4148 | Porte ET #1 → `Court_V_Actif` |
| D5, D6 | 1N4148 | Porte ET #2 → `Court_T_Actif` |
| D7, D8 | 1N4148 | Porte OU #1 → `Court_Tail` |
| D9, D10 | 1N4148 | Porte ET #3 → `Ouvert_V_Actif` |
| D11, D12, D13 | 1N4148 | Porte ET #4 (3 entrées) → `Ouvert_T_Actif` |
| D14, D15 | 1N4148 | Porte OU #2 → `Ouvert_Tail` |
| D16, D17 | 1N4148 | Porte ET #5 → `Bleu_Tail` |
| R5, R6 | 4,7 kΩ, pull-up vers `V_Ref` | Nœuds `Court_V_Actif`, `Court_T_Actif` |
| R8 | 4,7 kΩ, pull-down vers `GND` | Nœud `Court_Tail` |
| R9, R10 | 4,7 kΩ, pull-up vers `V_Ref` | Nœuds `Ouvert_V_Actif`, `Ouvert_T_Actif` |
| R11 | 4,7 kΩ, pull-down vers `GND` | Nœud `Ouvert_Tail` |
| R12 | 4,7 kΩ, pull-up vers `V_Ref` | Nœud `Not_Btn_Tail` (drain de Q5) |
| R13 | 4,7 kΩ, pull-down vers `GND` | Nœud `Bleu_Tail` |
| Q5 | 2N7000 | Inverseur de `BTN_TAIL_ETAT` → `Not_Btn_Tail` |
| Q6, Q7, Q8 | 2N7000 | Réservés — logique de priorité rouge + pilotage final R/V/B (câblage en cours) |

**Règle de câblage des portes à diodes (point de conception important, clarifié en cours de route) :**
- **Porte ET** (nœud tiré **haut** par une résistance vers `V_Ref`) : chaque diode a sa **cathode côté entrée**, son **anode côté nœud** — pour qu'une entrée basse puisse activement tirer le nœud vers le bas.
- **Porte OU** (nœud tiré **bas** par une résistance vers `GND`) : chaque diode a son **anode côté entrée**, sa **cathode côté nœud** — pour qu'une entrée haute puisse activement tirer le nœud vers le haut.

**Point en suspens (au moment de cette mise à jour) :** les diodes des 5 portes ET (D3-D6, D9-D10, D11-D13, D16-D17) ont été posées avec l'orientation d'une porte OU par erreur — **à réorienter (rotation 180°)** avant de poursuivre. La logique de priorité rouge (`Q_TAIL_NOT2` équivalent) et le pilotage final des 3 transistors R/V/B (Q_TAIL_R/V/B) restent aussi à câbler.

---

## 8. Boîtier et connectique physique

- Boîtier ABS/polycarbonate, indice de protection visé IP54 (usage atelier et extérieur, éclaboussures).
- Deux connecteurs **GX16-10** (VEHICLE mâle, TRAILER femelle) en face avant, avec capuchons de protection.
- Adaptateurs GX16-10 → 7/5/4 broches fournis en accessoires, câblés en dur (évite les erreurs de recâblage).
- Câble de pinces batterie externe de section suffisante (ex. AWG 14-16), avec porte-fusible en ligne visible.
- Port USB-C pour la recharge de la batterie interne.
- **Buzzer monté sous la façade**, avec un ou plusieurs petits trous percés pour laisser passer le son sans exposer le composant lui-même.
- **Rangée du bas (alarmes + jauge de batterie)** : prévoir un croquis d'agencement à l'étape PCB/composants pour confirmer l'espace exact, une fois les tailles réelles des DEL, du bargraph et du buzzer connues.
- **Un des côtés du boîtier** regroupe : le connecteur de la batterie externe (pinces), le port de recharge de la batterie interne, et le porte-fusible principal. Ce dernier doit être accessible de l'extérieur sans être encombrant ni accrocheur — un porte-fusible encastré/à fleur de boîtier (plutôt qu'un modèle en ligne qui dépasse) est recommandé, à valider à l'étape boîtier.

---

## 9. Suggestions de composants (à valider selon disponibilité et budget)

> **Note :** cette section a été la première ébauche de liste de composants. Elle est maintenue pour l'historique, mais l'**Annexe A** (fin de document) est désormais la liste de référence à jour, précise et complète — se fier à l'Annexe A en cas de divergence.

| Fonction | Exemple de composant |
|----------|----------------------|
| Comparateur/détection de courant par canal | Quadruple comparateur type LM339 ou quadruple ampli-op type LM324 (2 canaux logiques nécessaires par voie pour détection fenêtre ouvert/normal/court) |
| Fusible réarmable par canal | PTC (polyswitch) calibré selon le courant nominal du canal (ex. 2-5 A) |
| TVS entrée alimentation | SMBJ/SMCJ bidirectionnelle, tenue > 40 V |
| Anti-inversion de polarité | Contrôleur « diode idéale » analogique dédié (LM74610), un seul suffit — placé après le switch SOURCE, pas de OR-ing nécessaire (voir Annexe A) |
| Mesure de courant (shunt) | Résistance shunt de faible valeur (ex. 0,01-0,1 Ω) + ampli-op en montage soustracteur |
| BMS batterie interne | Module de protection Li-ion/LiFePO4 analogique dédié (ex. circuits type S8254A + double MOSFET, ou module de protection LiFePO4 tout-en-un) |
| Circuit de charge batterie interne | Circuit de charge dédié type TP4056 (Li-ion) ou circuit de charge CC/CV discret pour LiFePO4 |
| Isolement de mesure | Optocoupleurs passifs (ex. PC817) |
| Commutateurs mécaniques | Interrupteur MODE DPDT (VEHICLE/TRAILER) ; boutons-poussoirs à verrouillage (« push-push ») un par canal côté TRAILER ; commutateur SOURCE DPDT 3 positions (INT/OFF/EXT) |
| Vérification de cohérence (détection en renfort) | Résistance de tirage + diodes de logique « ET » + comparateur simple par port, pilotant une DEL d'alerte unique — n'agit pas sur l'alimentation |
| Pilotage DEL RGB tout-ou-rien | 3 transistors de commutation (un par couleur R/V/B) par canal, chacun saturé/bloqué (pas de zone linéaire) pour garantir un violet reproductible |
| Connecteurs principaux | 2× GX16-10 (10 broches), un mâle (VEHICLE) et un femelle (TRAILER), + câbles adaptateurs externes vers 4/5/7 broches |
| Thermistance de protection | CTN montée sur le dissipateur des composants de puissance |
| Cellules batterie interne | 3× 18650 Li-ion en série (3S1P), ex. Panasonic NCR18650B (3350 mAh) |
| Connecteur batterie interne | JST-PH 2 broches (boîtier à clé, non-inversable) |
| Buzzer d'alarme | Piézo actif (oscillateur intégré), ex. modèle compact type Kingstate KPEG ou équivalent — à valider disponibilité |
| Jauge de batterie | LM3914 (driver bar/dot analogique) + bargraph LED 5 segments (1 rouge, 2 jaune, 2 vert), potentiomètre de calibration |
| Test de résistance de masse (voyant MASSE) | Source de courant de test connu (résistance depuis le + régulé) + comparateur (ex. LM339) mesurant la chute de tension sur la broche masse, seuil calibré par potentiomètre |
| Coupure ultra rapide port VEHICLE (injection, exigence F5) | MOSFET de puissance (canal N, faible Rds(on)) piloté directement par le comparateur de courant — coupure de l'ordre de la microseconde, contre plusieurs millisecondes pour un relais mécanique |

---

## 10. Points à valider avant fabrication

1. Confirmer avec précision les brochages des variantes de connecteurs 5 broches présentes sur le marché visé (Amérique du Nord vs Europe), car ils ne sont pas universellement standardisés.
2. Déterminer le courant nominal cible par canal selon le marché (remorques légères type utilitaire vs remorques lourdes/RV avec freins électriques, qui consomment davantage).
3. Valider la tenue aux transitoires selon la norme ISO 7637-2 si une certification ou une robustesse « niveau industrie automobile » est visée.
4. Définir précisément la valeur de la charge simulée en mode Véhicule pour couvrir à la fois les ampoules incandescentes et les feux DEL modernes, ces derniers pouvant déclencher de fausses alertes « ampoule grillée » sur certains véhicules si la charge est trop faible.
5. Valider en pratique la fenêtre de seuils des comparateurs analogiques (ouvert / normal / court-circuit) pour chaque canal, car les courants nominaux varient sensiblement entre un feu de position DEL (quelques dizaines de mA) et un frein électrique de remorque lourde (plusieurs ampères) — un jeu de seuils fixes unique risque d'être trop large ou trop étroit selon le canal ; prévoir des seuils ajustables (potentiomètres de calibration) par canal ou par groupe de canaux.
6. Vérifier la fiabilité mécanique à long terme des commutateurs et boutons-poussoirs choisis, puisqu'ils portent désormais toute la responsabilité de la logique de sécurité (contacts de qualité automobile/industrielle recommandés, IP adéquat).

---

## Annexe A — Liste de composants (BOM)

> Cette liste regroupe toutes les pièces réelles retenues pour le projet, organisées par sous-système, avec quantité et rôle. Elle sert de base pour estimer l'encombrement des PCB et positionner la batterie interne dans le boîtier. Les valeurs exactes des résistances/potentiomètres seront calculées précisément à l'étape du schéma (KiCad) ; les quantités ci-dessous sont fiables pour l'estimation d'espace.

### A.1 Alimentation et gestion d'énergie

| Composant | Référence / modèle suggéré | Qté | Utilité |
|---|---|---|---|
| Cellule Li-ion 18650 | Panasonic NCR18650B (3350 mAh) | 3 | Pack interne 3S1P (~11,1-12,6 V) |
| Support/porte-cellules 18650 (×3) | Support rigide type « 3×18650 en ligne » avec contacts à ressort | 1 | Maintien mécanique du pack, facilite le remplacement |
| Module BMS 3S Li-ion | Module de protection 3S tout-en-un (coupure surcharge/décharge/surintensité + équilibrage) | 1 | Protection de la batterie interne (R1 : IC analogique, pas de MCU) |
| Module de charge 3S Li-ion | Module dédié 12,6 V (ex. à base de CN3791 ou équivalent commercial) | 1 | Charge via port USB-C PD ou prise barrel |
| Connecteur batterie interne | JST-PH 2 broches (paire mâle/femelle) | 1 paire | Connexion non-inversable pack ↔ circuit principal |
| Connecteur batterie externe (façade) | GX16-2 (cohérent avec VEHICLE/TRAILER), à valider ≥15 A par broche selon fabricant | 1 | Entrée alimentation externe 12 V, câble à pinces AWG 14-16 |
| Bornier de raccordement PCB (GX16-2 → PCB) | Bornier à vis 2 positions, pas 5,08 mm, calibré ≥15 A | 1 | Connexion robuste et réparable entre le connecteur en façade et le PCB (plus solide qu'un header à friction) |
| Fusible principal | Fusible lame automobile (ATO), 15 A | 1 | Protection globale de l'appareil |
| Porte-fusible étanche, encastré | Type « à fleur de boîtier », accessible de l'extérieur | 1 | Accessible sans être encombrant (§8) |
| Contrôleur diode idéale (anti-inversion) | LM74610QDGKRQ1, boîtier SOIC/SMD (exception documentée à R5 — aucune variante traversante existe, conservé tel quel) | 1 | Protection anti-inversion de polarité à faible perte, placé après le switch SOURCE sur la source déjà sélectionnée. *(Correction : un seul suffit — le OR-ing entre 2 sources était redondant, puisque SW1 (ON-OFF-ON) garantit déjà qu'une seule source est connectée à la fois ; l'ancien 2ᵉ LM74610 était pour un cas d'usage qu'on a abandonné.)* |
| MOSFET externe requis par le LM74610 (oubli corrigé) | IRLZ44N, boîtier TO-220 traversant (R5) — même référence que les autres MOSFET du projet | 1 | Le LM74610 est un contrôleur, pas l'interrupteur lui-même : Drain → nœud VPwr_Selection (même point qu'ANODE), Source → nœud V12_Rail (même point que CATHODE) — porte le courant réel de l'appareil (jusqu'à 15 A, limité par F1) |
| Condensateur de pompe de charge (VCAPH/VCAPL du LM74610) | Céramique 2,2 µF, ≥16 V, diélectrique X7R, traversant (radial) (R5) | 1 | Requis par le LM74610 pour générer la tension de gâchette qui ouvre pleinement le MOSFET — flottant, non connecté à la masse |
| TVS bidirectionnelle, entrée principale | SMCJ36CA (ou équivalent, tenue > 40 V) | 2 | Absorption des transitoires (load dump, ISO 7637-2) |
| Régulateur linéaire/à découpage | Module réglable type LM2596, ou 78xx fixe | 2 | Rails de référence pour comparateurs, isolés du rail de puissance |
| Switch MODE | DPDT ON-ON, qualité industrielle | 1 | Sélection VEHICLE/TRAILER (§6.3) |
| Switch SOURCE | DPDT ON-OFF-ON, qualité industrielle | 1 | Sélection INT/OFF/EXT (§6.1) |

### A.2 Connecteurs principaux

| Composant | Référence / modèle suggéré | Qté | Utilité |
|---|---|---|---|
| GX16-10 mâle (panneau) | GX16, 10 broches, mâle | 1 | Port VEHICLE |
| GX16-10 femelle (panneau) | GX16, 10 broches, femelle | 1 | Port TRAILER |
| Câble adaptateur GX16-10 → standard (VEHICLE) | 3 câbles (→4, →5, →7 broches), extrémité femelle GX16-10 | 3 | Interface vers connecteur du véhicule |
| Câble adaptateur GX16-10 → standard (TRAILER) | 3 câbles (→4, →5, →7 broches), extrémité mâle GX16-10 | 3 | Interface vers connecteur de la remorque |

### A.3 Par canal de test (TAIL, BRAKE, GAUCHE, DROITE, RECUL, AUX 12V) — ×6 canaux

| Composant | Référence / modèle suggéré | Qté (par canal) | Qté totale (×6) | Utilité |
|---|---|---|---|---|
| Bouton-poussoir à verrouillage (push-push) | Qualité industrielle, contact étanche | 1 | 6 | Armement du canal, actif côté TRAILER seulement |
| Fusible réarmable (PTC) | Polyswitch, calibré ~3-5 A (à ajuster par canal, §10.5) | 1 | 6 | Protection contre court-circuit côté remorque |
| Diode TVS | SMBJ33A (ou calibrée selon canal) | 1 | 6 | Absorption de transitoires par canal |
| Résistance shunt (côté VEHICLE) | ~0,05-0,1 Ω, traversant axial (1-2 W) (R5) | 1 | 6 | Mesure de courant, chaîne VEHICLE (base des 3 comparateurs) |
| Résistance shunt (côté TRAILER, dédiée) | ~0,05-0,1 Ω, traversant axial (1-2 W) (R5) | 1 | 6 | Mesure de courant, chaîne TRAILER indépendante (Option 2, §7.2) |
| Résistance de charge simulée (côté VEHICLE) | Fixe ou commutable, ~2-3 Ω à froid | 1 (bloc) | 6 | Simule une ampoule pour éviter une fausse alerte « circuit ouvert » côté véhicule |
| Optocoupleur d'isolement double | PC817-2 (ou PC817X2/EL817-2), boîtier DIP-8 traversant (R5) | 1 | 6 | Isolement des 2 seuils (ouvert + court) de la mesure côté VEHICLE, dans un seul boîtier (protection structurelle F5 ; correction Bloc 3, §7.1) |
| Comparateur quadruple LM339 | LM339N, boîtier DIP-14 traversant (R5), monté sur **socket turned-pin DIP-14** (R7) | 5/4 de puce (3 VEHICLE + 2 TRAILER) | 8 puces au total* | Détection ouvert/court par côté (VEHICLE ET TRAILER, chaînes indépendantes — Option 2, §7.2) + injection rapide (1 seuil, VEHICLE) par canal |
| MOSFET de puissance (sortie TRAILER) | IRLZ44N, boîtier TO-220 traversant (R5) | 1 | 6 | Le bouton pilote ce MOSFET plutôt que de porter lui-même le courant (durabilité du contact) |
| MOSFET de puissance (coupure injection VEHICLE) | IRLZ44N, boîtier TO-220 traversant (R5) | 1 | 6 | Coupure ultra rapide (µs) en cas d'injection détectée (F5) |
| DEL RGB (cathode commune) | Générique traversant 5 mm ou rectangulaire, RGB CC, montage direct en façade (R5) | 1 | 6 | Affichage Bleu/Vert/Violet/Rouge (§5.0) |
| Transistor de commutation (pilotage DEL R/V/B) | 2N7000, boîtier TO-92 traversant (R5) | 3 | 18 | Un par couleur (R/V/B), tout-ou-rien |
| Transistor inverseur/priorité (Bloc de Test) | 2N7000, boîtier TO-92 traversant (R5) | 2 | 12 | Inversion `BTN_TAIL_ETAT` + priorité rouge sur bleu (§7.2.4) |
| Diode logique 1N4148 (Bloc de Test) | 1N4148, traversant (R5) | 15 | 90 | Portes ET/OU à diodes, combine VEHICLE/TRAILER/bouton (§7.2.4) |
| Résistance de tirage (Bloc de Test) | 4,7 kΩ, traversant 1/4 W (R5) | 8 | 48 | Pull-up/pull-down des nœuds logiques des portes ET/OU (§7.2.4) |
| Potentiomètre de calibration de seuil (côté VEHICLE) | Multi-tours, petit format | 1 | 6 | Ajustement ouvert/court selon le courant nominal réel du canal, chaîne VEHICLE |
| Potentiomètre de calibration de seuil (côté TRAILER, dédié) | Multi-tours, petit format | 1 | 6 | Ajustement ouvert/court, chaîne TRAILER indépendante (Option 2, §7.2) |

*Total comparateurs requis : 5 par canal × 6 canaux = 30, + 1 pour la masse (§A.4) = 31 → **8× LM339** (32 comparateurs disponibles, 1 de marge).

### A.4 Test de masse (voyant MASSE)

| Composant | Référence / modèle suggéré | Qté | Utilité |
|---|---|---|---|
| Résistance source de courant de test | Valeur fixe depuis le rail régulé | 1 | Injecte un courant de test connu dans la broche masse |
| Potentiomètre de calibration | Multi-tours, petit format | 1 | Ajuste le seuil de résistance acceptable |
| Comparateur | Inclus dans le compte LM339 de l'A.3 | 1 | Décision « masse correcte / dégradée » |
| DEL d'alerte MASSE | DEL simple traversant, rouge, rectangulaire, montage direct en façade (R5) | 1 | Indication en rangée du bas (§6.5) |

### A.5 Alarmes critiques et buzzer (rangée du bas)

| Composant | Référence / modèle suggéré | Qté | Utilité |
|---|---|---|---|
| DEL d'alerte (inversion polarité, injection, surtension, court-circuit bus, incohérence) | DEL simple traversant, rouge, rectangulaire, montage direct en façade (R5) | 5 | Une par type d'anomalie grave (§6.5) |
| Buzzer actif | Modèle compact type Kingstate KPEG (à valider disponibilité) | 1 | Alarme sonore partagée |
| Transistor de commande buzzer | 2N3904 ou 2N7000, boîtier TO-92 traversant (R5) | 1 | Pilotage on/off du buzzer |
| Résistance de tirage + diodes logique « ET » (cohérence mode) | Génériques | ~4 | Comparaison position switch MODE vs broches de détection (§6.4) |

### A.6 Jauge de batterie (5 segments)

| Composant | Référence / modèle suggéré | Qté | Utilité |
|---|---|---|---|
| Driver bar/dot | LM3914N, boîtier DIP-18 traversant (R5), monté sur **socket turned-pin DIP-18** (R7) | 1 | Pilotage analogique de la jauge |
| Bargraph LED 5 segments | 1 rouge, 2 jaune, 2 vert, boîtier traversant unique, montage direct en façade (R5) | 1 | Affichage du niveau de charge |
| Potentiomètre de calibration | Multi-tours | 1 | Ajuste la plage de tension affichée à la chimie du pack |
| Résistance de référence (R_ref) | ~1,2 kΩ, traversant 1/4 W (R5) | 1 | Fixe le courant par DEL (~10 mA) |
| Condensateur de découplage | 0,1 µF, traversant (R5) | 1 | Stabilité, anti-scintillement |

### A.7 Divers et structure

| Composant | Référence / modèle suggéré | Qté | Utilité |
|---|---|---|---|
| Résistances diverses (diviseurs, seuils, tirage) | Assortiment traversant 1/4 W, valeurs à finaliser au schéma (R5) | ~40-50 | Polarisation et calibration des comparateurs/transistors |
| Condensateurs de découplage divers | 0,1 µF céramique, traversant (R5) | ~10-12 | Stabilité des puces (LM339 ×5, LM3914, régulateurs) |
| Sockets « turned-pin » (R7) | DIP-14 (LM339) et DIP-18 (LM3914) | 5× DIP-14 + 1× DIP-18 | Remplacement facile sans risque pour le PCB (R7) |
| PCB principal | À dimensionner selon le placement final | 1 | Porte l'ensemble des canaux, protections et alimentation |
| PCB dédié (optionnel) | Petit format, pour la jauge de batterie | 1 | Si séparé du PCB principal, monté directement derrière le bargraph (§7) |
| Boîtier | ABS/polycarbonate, IP54 | 1 | Enceinte complète (§8) |

**Note pour la planification du boîtier :** le pack de 3 cellules 18650 (Panasonic NCR18650B) occupe, avec son support, environ 65 mm × 60 mm × 20 mm — un volume distinct du PCB, à réserver comme espace mécanique dédié (pas monté en surface), généralement contre une paroi du boîtier. Les dimensions exactes des PCB dépendront du placement final des composants ci-dessus, mais cette liste donne un décompte fiable de pièces pour estimer une surface de PCB réaliste avant de passer à KiCad (mise à jour suite au détail complet du canal TAIL, §7.2 — chaîne TRAILER indépendante + logique du Bloc de Test) : **8 puces LM339** (31 comparateurs), **12 MOSFET de puissance**, **12 DEL au total** (6 DEL RGB tricolores, une par canal + 5 DEL d'alarme + 1 DEL masse — plus le bargraph 5 segments de la jauge), **30 transistors** de commutation (5 par canal : 3 pour le pilotage R/V/B de la DEL RGB + 2 pour la logique de priorité/inversion du Bloc de Test), et **90 diodes 1N4148** (15 par canal, portes ET/OU du Bloc de Test).

---

*Ce document constitue une base de conception. Une revue par un ingénieur électronicien qualifié est recommandée avant toute fabrication d'un prototype destiné à être connecté à un véhicule réel.*
