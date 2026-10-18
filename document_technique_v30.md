# Document Technique et de Conception
## Testeur/Diagnostiqueur Universel Véhicule ↔ Remorque

**Version :** 30.0 — **Chantiers 6 (connecteurs) et 7 (alimentation) intégrés.**
- **Alimentation refaite** (§6.1) : anomalies **A7** (Q64 côté haut à grille 5 V, l'appareil ne pouvait pas démarrer) et **A8** (C1 relié par des bus) **corrigées**. `V12_PostShunt` = rail permanent ; `V12_Rail` = côté TRAILER seulement. Fusible **F8** sur la branche INT.
- **Bloc « BATT. EXT. »** (sous-feuille `Limite_Pack_Int`) : sur pile interne, coupe le côté TRAILER au-delà de ~12 A et allume LED2 (ambre).
- **Mémoire du défaut de court-circuit du bus** (D180, C11, R219 = 100 kΩ).
- **Pack interne** : 3S1P à cellules **≥ 20 A**, BMS **≥ 30-40 A** (spécifications).
- **Mesure de masse refaite** (§6.5.1) : pince au châssis + **comparateur à fenêtre** sur U15 ; Kelvin broche 9 abandonné ; **D151 retirée** (D158 seule) ; **buzzer sur défaut de masse** (D149 dans le hub) ; inversion de polarité de `Masse_Fault` corrigée.
- **SW9** : positions 9 (PINCE VEH, R308) et 10 (PINCE TRAIL, R309). Correction documentaire des positions 6/7.
- **PTC figés** : F2-F5 = **MF-R700**, F7 = **MF-R300**.
- **Connecteurs J3/J4** : GX16-10 jugé **insuffisant** ; cahier des charges fixé (≥ 10 contacts, 15 A), modèle **en suspens**.
- **Anomalie A9** (vérification de cohérence inopérante) relevée, **en suspens** (§6.4).
- Empreintes : 14 LM339 → DIP-14 socket ; U17 → DIP-18 socket. TVS P6KE39A (R5).
- **ERC 0/0** (12 oct., commit `e2f94fe`). Annexe A ré-extraite : **652 composants**, 31 feuilles.

**Version :** 29.0 — Chantier 5 : A1, A2, A6 fermées sur les 5 canaux d'éclairage ; mesure TRAILER par capteur PNP (Chantier 2) ; 620 composants.

**Version :** 28.0 — **Intégration des chantiers 1, 3 et 4.**
- **Chantier 1** : conflit RL_x (§10.1) **tranché**, avec une valeur fixe de ~1 A sur les 5 canaux d'éclairage.
- **Chantier 3** : voltmètre/ampèremètre de façade câblé (R242, PS1, J1, SW9). BRAKE devient le **12 V auxiliaire** et AUX le **frein électrique**. Correction : P1 est côté batterie externe.
- **Chantier 4** : canal AUX entièrement refait et vérifié.
- **Trois anomalies transversales** que l'ERC ne détecte pas : **A1** (optocoupleurs VEHICLE inopérants), **A2** (MOSFET d'injection en suiveur de source) et **A6** (sortie TRAILER plafonnée à ~2-3 V). Les trois sont corrigées sur AUX et **restent ouvertes sur les 5 autres canaux** (§7.6).
- **ERC : 0 erreur, 0 avertissement** (29 sept., 4 oct. et 7 oct.).
- Décomptes de l'Annexe A ré-extraits du dépôt le 7 oct. (commit `b119a28`).

**Version :** 27.0 — Rattrapage documentaire majeur : Bloc_Masse, détail Alarmes/Buzzer, assainissement des bibliothèques KiCad, corrections de comptage, R9 proposée.
**Version :** 26.0 — Jauge de batterie fermée ; conflit §10.1 identifié.
**Version :** 23.0 — VEHICLE_LEFT validé avec références réelles.
**Version :** 22.0 — Canal_TAIL fermé et validé (résolution de l'oscillation de Q11).
**Date :** 12 octobre 2026

> **Note sur la numérotation des versions.** Il n'existe **aucune version 24 ni 25** de ce document (erreur de numérotation dans une conversation de travail).

> **Légende des marqueurs**
> - `✅ VÉRIFIÉ KICAD` : donnée extraite directement des fichiers `.kicad_sch` du dépôt GitHub `patricecouillard-code/trailerDiagnosticSystem`.
> - `⚠️ RECONSTRUIT` : section rédigée a posteriori (v27). Les valeurs viennent des fichiers, le raisonnement est reconstitué.
> - `🔵 PROPOSÉ` : en attente d'arbitrage de l'utilisateur.

> **Documents de chantier** (base de connaissances du projet) : `Chantier_1_VEHICLE_RL`, `Chantier_2_TRAILER_mesure` (appliqué en v29), `Chantier_3_Voltmetre_Amperemetre`, `Chantier_4_AUX_Frein_Electrique`, `Chantier_5_Corrections_5_canaux` (tableau complet des références par canal), `Chantier_6_Connecteurs`, `Chantier_7_Alimentation_Thermique`. Ils contiennent le détail des décisions ; ce document en garde la synthèse.

---

## 📐 RÈGLES DE CONCEPTION (à appliquer en tout temps)

> Ces règles ont préséance sur toute autre considération (coût, esthétique, simplicité), sauf mention contraire explicite dans la règle elle-même.

**R1 — 100 % analogique.** Aucun microcontrôleur, aucun composant programmable, aucun firmware. Toute la logique (commutation, détection, indication, gestion de charge) est réalisée avec des composants discrets et des circuits intégrés analogiques câblés. *Exceptions documentées : modules fermés sans logique de décision (module voltmètre/ampèremètre de façade et convertisseur isolé PS1, §7.4).*

**R2 — Robustesse avant élégance.** Entre deux solutions, on retient la plus fiable, robuste et efficace en tension/courant/résistance, même si une autre est plus simple, plus compacte ou moins chère, sauf si l'option la plus simple n'entraîne aucun compromis de fiabilité.

**R3 — Test d'utilité.** Avant d'ajouter une composante ou une fonctionnalité, on se demande si elle est utile. Si non, on l'oublie.

**R4 — Qualité des pièces.** Pièces de qualité, robustes, sauf coût exorbitant. *Précision (oct. 2026) : on retient la meilleure pièce même si elle est temporairement hors stock ; rien n'est commandé avant la fin de la conception.*

**R5 — Format de montage.** Composants traversants (« through-hole ») partout. **Exception unique et documentée : le LM74610** (contrôleur diode idéale, §6.1), qui n'existe qu'en boîtier SMD.

**R6 — Précision du type d'étiquette (net label).** Toute étiquette proposée précise son type :
- **locale** : valide seulement sur sa feuille ;
- **globale** : connectée partout dans le projet ;
- **hiérarchique** : crée une patte sur le rectangle parent.

**R7 — Socket pour puces logiques/analogiques.** LM339, LM3914, etc. sur socket **turned-pin**. Exceptions : MOSFET de puissance TO-220 et LM74610.

**R8 — Revalidation systématique des composants dimensionnés empiriquement.** Toute valeur fixée par une plage indicative est revalidée par calcul réel (courant nominal, P = I²×R, marge) **avant fabrication**, canal par canal. *État : RL_x et shunts VEHICLE revalidés (Chantier 1, §10.1) ; shunts TRAILER fixés à 12 Ω / 1 W écrêtés (Chantier 5) ; PTC d'éclairage fixés (Chantier 6, §10.1).*

**R9 — Aucune fonction de sécurité ne repose sur un appariement d'étiquettes.** 🔵 PROPOSÉ (inchangé depuis la v27). Une liaison qui participe à une protection doit être tracée en fil, ou reposer sur une étiquette dont le retrait déclencherait une erreur ERC visible.
*Origine : incident Q54 du 19 sept. (§7.2.5).*
*Nuance d'octobre 2026 : l'utilisateur accepte les étiquettes locales à la place des fils pour la lisibilité, notamment `Injection_Aux_Fault` sur Q66.3. La règle reste donc à arbitrer.*

---

## 💡 SUGGESTIONS ET IDÉES (registre de suivi)

| Idée | Proposée par | Statut |
|---|---|---|
| Confirmation lumineuse de la position des switchs MODE et SOURCE | Utilisateur | ❌ Refusée |
| Jauge de batterie à échelle (10 segments) | Utilisateur | ✅ Acceptée — §7, §A.6 |
| Support/pied pour positionner le testeur à angle | Utilisateur | ⏳ En attente — étape boîtier |
| Buzzer actif | Utilisateur | ✅ Acceptée |
| Test de résistance de masse (Kelvin, broche 9) | Claude | ❌ Abandonné (v30) — remplacé par la pince au châssis, §6.5.1 |
| Socket turned-pin | Utilisateur | ✅ R7 |
| PC817-2 double au lieu de 2× PC817 | Claude | ⚠️ Non appliquée — 2× PC817 DIP-4 par canal |
| Chaîne de mesure TRAILER indépendante (Option 2) | Utilisateur | ✅ Acceptée |
| Renommer « Affichage » en « Bloc de Test » | Utilisateur | ✅ Acceptée |
| 6 feuilles par canal, sous-feuilles hiérarchiques | Utilisateur | ✅ Acceptée |
| Revalidation par calcul réel (R8) | Utilisateur | ✅ R8 |
| PTC dédié côté VEHICLE | Claude | ❌ Refusée (R3) |
| Support AUX triple essieu (~20-24 A de charge simulée) | Utilisateur / Claude | ✅ **Résolue autrement (v28).** On ne simule plus un freinage : on mesure la résistance des aimants à ~1 A, ce qui rend le 3 essieux testable (§7.5.3) |
| Buzzer sur `Masse_Fault` | Utilisateur | ✅ Acceptée |
| Symbole 2N7000 propre au projet | Claude | ⏳ En attente, moins urgent : le mismatch Q54 est corrigé (29 sept.) |
| Empreintes projet en dossiers `.pretty` séparés | Claude | ⏳ En attente |
| **RL_x fixe ~1 A** (remplace les approches A et B) | Claude / Utilisateur | ✅ **Acceptée le 21 sept., appliquée le 30 sept.** — §10.1 |
| **Voltmètre : commutateur rotatif SW9 (option V2)** | Utilisateur | ✅ Acceptée le 25 sept. — §7.4 |
| **BRAKE = 12 V auxiliaire (option A)** | Utilisateur | ✅ Acceptée le 26 sept. — §7.1 |
| **Étireur d'impulsions sur AUX** (contrôleurs PWM) | Claude | ✅ Construit le 4 oct. — §7.5.2 |
| **Protection thermique côté TRAILER AUX** | Utilisateur | ✅ Construite le 6 oct. (SW11) — §7.5.3 |
| **SW9 position 8 = TRAIL AUX** (lecture de la résistance des aimants) | Claude | ✅ Construite le 7 oct. (R252) — §7.5.4 |
| Noms de rôle dans le champ « Etiquette » des composants | Utilisateur | ✅ Acceptée — AUX : liste au §A.9 |
| Étireur d'impulsions pour les clignotants (LEFT/RIGHT, ~1-2 Hz) | Claude | ⏳ En attente — A1 corrigée, l'étireur reste à étudier (§7.6) |
| **Mesure TRAILER par capteur PNP + diode d'écrêtage MUR1520G** (Chantier 2) | Claude | ✅ Construite le 9 oct. sur les 5 canaux d'éclairage — §7.2.3 |
| **Corrections des 4 derniers canaux générées par script** à partir de TAIL, puis vérifiées dans les fichiers | Claude / Utilisateur | ✅ Appliquée le 9 oct. — voir `Chantier_5` |
| **Bloc « BATT. EXT. »** : coupure côté TRAILER + DEL si le pack interne est trop sollicité | Utilisateur | ✅ Construit le 12 oct. — §6.1 |
| **Mémoire du défaut de court-circuit du bus** (supprime le « hoquet ») | Claude | ✅ Construite le 12 oct. — §6.6 |
| **Pince au châssis + comparateur à fenêtre** (mesure de masse, option 1+2) | Claude / Utilisateur | ✅ Construite le 12 oct. — §6.5.1 |
| **Buzzer sur défaut de masse** | Utilisateur | ✅ Câblé le 12 oct. (D149) — §6.6 |
| Ventilateur dans le boîtier | Utilisateur | ❌ Déconseillé (IP54, pièce mobile, consommation) — dissipation par la paroi, §8 |
| Câbles fixes sortant du boîtier (au lieu de connecteurs en façade) | Claude | ❌ Refusée par l'utilisateur |

---

## 1. Objectif du projet

Concevoir un appareil portable permettant de diagnostiquer le circuit électrique d'attelage, dans deux directions :

- **Mode « Véhicule »** : l'appareil se branche sur la prise du véhicule et **simule une remorque**, afin de vérifier que le véhicule envoie correctement chaque signal (position, clignotants, recul, 12 V auxiliaire, frein électrique).
- **Mode « Remorque »** : l'appareil se branche sur la prise de la remorque et **simule le véhicule**. Il vérifie le câblage et les feux de la remorque, la résistance des électroaimants de frein, les circuits ouverts, courts-circuits et masses défectueuses.

L'appareil supporte les connecteurs nord-américains 4, 5 et 7 broches. Il fonctionne sur pile interne rechargeable ou sur batterie 12 V externe. Il intègre des protections contre l'injection de courant vers le véhicule et contre les défauts électriques.

Ce projet suit en tout temps les **Règles de conception R1-R9**.

---

## 2. Exigences fonctionnelles (cahier des charges)

| # | Exigence | Détail |
|---|----------|--------|
| F1 | Sélecteur de mode | DPDT manuel (VEHICLE/TRAILER) ; détection de branchement en simple vérification de cohérence (§6.4) |
| F2 | Connecteurs supportés | 4, 5 et 7 broches (SAE J560) via câbles adaptateurs depuis deux connecteurs de façade (modèle en suspens, §3.3.1) |
| F3 | Alimentation double | Batterie interne 3S **et** batterie 12 V externe (pinces) |
| F4 | Basculement d'alimentation | Sans coupure lors du branchement |
| F5 | Protection du véhicule | Aucune injection de courant incontrôlée vers le véhicule ; coupure **ultra rapide** par semi-conducteurs |
| F6 | Auto-protection | Inversion de polarité, surtension, surintensité, court-circuit, transitoires |
| F7 | Détection de défaut | Ouvert / normal / court, par canal |
| F8 | Interface utilisateur | DEL RGB par canal selon §5.0 ; voltmètre/ampèremètre de façade (module fermé, §7.4) |
| F9 | Robustesse | Boîtier résistant aux intempéries |
| F10 | Conception analogique | R1 |
| F11 | Robustesse | R2 |

---

## 3. Brochages des connecteurs à supporter

### 3.1 Connecteur plat 4 broches

| Broche | Couleur | Fonction |
|---|---|---|
| 1 | Blanc | Masse |
| 2 | Brun | Position / plaque |
| 3 | Jaune | Clignotant / stop gauche |
| 4 | Vert | Clignotant / stop droit |

### 3.2 Connecteur plat 5 broches

Comme le 4 broches, plus un 5ᵉ fil dont la fonction varie selon le fabricant : frein électrique, recul ou 12 V auxiliaire. C'est le **câble adaptateur** qui route ce fil vers le bon canal.

### 3.3 Connecteur 7 broches (SAE J560) — correspondance avec les canaux ✅ (décidée les 25-26 sept.)

| Lame J560 | Couleur | Fonction | **Canal de l'appareil** |
|---|---|---|---|
| 1 | Blanc | Masse | GND |
| 2 | Noir | 12 V auxiliaire | **BRAKE** (option A, 26 sept.) |
| 3 | Bleu | Frein électrique | **AUX** |
| 4 | Jaune | Gauche (+ stop) | LEFT |
| 5 | Vert | Droit (+ stop) | RIGHT |
| 6 | Brun | Position | TAIL |
| 7 | Violet | Recul | BACKUP |

> **Il n'existe pas de lame « stop » séparée en J560** (vérifié sur une prise Mopar 68014986AB). Le stop passe par les lames gauche et droite, qui s'allument ensemble au freinage. Le nom de canal **BRAKE** est conservé dans KiCad (`Veh_Brake`, `Trail_Brake`), mais sa **sérigraphie sera « 12V AUX »**.

**Implémentation :** deux connecteurs en façade, **VEHICLE (mâle, J3)** et **TRAILER (femelle, J4)**, avec des câbles adaptateurs externes vers les prises 4, 5 et 7 broches.

### 3.3.1 Connecteurs J3 / J4 — ⏸ MODÈLE EN SUSPENS (Chantier 6)

**Le GX16-10 est insuffisant** : ~4 A par contact (valeur de vendeur), alors que la masse en TRAILER atteint ~14-18 A et un canal incandescent ~5 A.

**Cahier des charges retenu (12 oct.) :**

| Exigence | Valeur |
|---|---|
| Contacts | **≥ 10** |
| Capacité | **15 A par contact** |
| Masse | **2 contacts en parallèle** |
| Pince au châssis | **1 contact** ; le fil de pince est dans le câble adaptateur, sortie en Y près de la prise 7 broches |
| Montage | En façade, étanche ; câbles adaptateurs rangés à part (câbles fixes refusés) |
| Câble des adaptateurs | **SOOW 10 conducteurs** (14/10 existe ; déclassement à vérifier) |

Pistes : Deutsch DT / Amphenol AT (contact taille 16 ≈ 13 A, taille 12 ≈ 23 A), Neutrik speakON NL8. Choix par l'utilisateur.

**Affectation actuelle dans KiCad (`Conn_01x10`, numérotation provisoire) ✅ VÉRIFIÉ KICAD (12 oct.) :**

| Broche | J3 (VEHICLE) | J4 (TRAILER) | Note |
|---|---|---|---|
| 1 | `GND` | `GND` | Retour commun ; 2 contacts dans le futur connecteur |
| 2 | `Veh_Tail` | `Trail_Tail` | |
| 3 | `Veh_Brake` | `Trail_Brake` | = 12 V auxiliaire |
| 4 | `Veh_Left` | `Trail_Left` | |
| 5 | `Veh_Right` | `Trail_Right` | |
| 6 | `Veh_Backup` | `Trail_Backup` | |
| 7 | `Veh_Aux` | `Trail_Aux` | = frein électrique |
| 8 | `Detect_Vehicule` | `Detect_Trailer` | Cohérence (§6.4, A9) |
| 9 | `Pince_Veh` | `Pince_Trail` | **Pince au châssis** (v30) |
| 10 | libre | libre | Réserve |

Sur la page racine, ces étiquettes sont **locales** (R6).

**Prise de batterie externe P1 :** le GX16-2 actuel (~7 A) est aussi insuffisant pour 15 A. Spécifications : 2 pôles, ≥ 15 A, étanche, en façade, idéalement de la même famille que J3/J4 (piste : Deutsch DT04-4P à bride, 2 contacts par pôle). Câble des pinces ≥ AWG 14.

---

## 4. Architecture générale (100 % analogique)

Un **switch de mode manuel (DPDT)** arme physiquement l'un ou l'autre des deux ports. Un circuit de **vérification de cohérence** surveille que le câble branché correspond au switch. Il n'y a pas de cerveau central : les blocs communiquent uniquement par niveaux de tension et de courant.

```
 Bloc Alimentation ──▶ Switch MODE (DPDT) ──┬──▶ Port VEHICLE (lecture : charge + shunt + LM339 + PC817)
 (int./ext., anti-inv., │                     └──▶ Port TRAILER (source : bouton + MOSFET + PTC + TVS)
  shunt R242, PS1)      │
                        ├──▶ Vérification de cohérence ──▶ Bloc Alarmes/Buzzer ◀── Bloc_Masse (pince, fenêtre)
                        └──▶ Voltmètre/ampèremètre (SW9, 10 positions)
```

### 4.1 Hiérarchie des feuilles KiCad ✅ VÉRIFIÉ KICAD (12 oct.)

31 fichiers `.kicad_sch`, racine comprise :

```
TrailerDiagnosticSystem (racine)   ← J3, J4, SW9 + R235–R241, R252, R308, R309
├── Alimentation                   ← alim., anti-inversion, alarmes, jauge, shunt R242, PS1, J1, F8, R289
│   └── Limite_Pack_Int            ← bloc « BATT. EXT. » (v30)
├── Mode_Coherence
├── Detection_De_Masse             ← fichier MassDetection.kicad_sch
├── Alarmes_Buzzer_sch
├── Alertes
├── Canal_Tail    ── Vehicule_Tail / Trailer_Tail / Bloc_Test_Tail
├── Canal_Brake   ── Vehicule_Brake / Trailer_Brake / Bloc_Test_Brake
├── Canal_Left    ── Vehicule_Left / Trailer_Left / Bloc_Test_Left
├── Canal_Right   ── Vehicule_Right / Trailer_Right / Bloc_Test_Right
├── Canal_Backup  ── Vehicule_Backup / Trailer_Backup / Bloc_Test_Backup
└── Canal_Aux     ── Vehicule_Aux / Trailer_Aux / Bloc_Test_Aux
```

---

## 5. Fonctionnement par mode

### 5.0 Code couleur de référence

Une DEL RGB par canal ; chaque couleur est pilotée tout-ou-rien par son propre 2N7000.

| État | Couleur | Signification |
|---|---|---|
| Prêt | **Bleu** | Mode Remorque, bouton relâché |
| Normal | **Vert** | Courant dans la plage attendue |
| Ouvert | **Violet** (R+B) | Canal actif sans courant |
| Court-circuit | **Rouge** | Priorité d'affichage absolue |

### 5.1 Mode « Véhicule » — port VEHICLE (passif)

1. Chaque canal est chargé par une **résistance fixe** (RL_x, ~1 A pour l'éclairage ; 4,7 Ω pour AUX), pour que le véhicule détecte une remorque.
2. Un **shunt + LM339** mesure le courant et pilote la DEL.
3. La mesure passe par des **PC817** (isolement, F5). *Anomalie A1 corrigée sur les 6 canaux (§7.6).*
4. La masse est vérifiée par la pince au châssis et le Bloc_Masse (§6.5.1).

Côté véhicule, l'appareil ne fournit **jamais** de tension active.

### 5.2 Mode « Remorque » — port TRAILER (source, boutons à verrouillage)

1. Plusieurs canaux peuvent être actifs simultanément.
2. Le canal alimente la remorque à travers un **PTC**.
3. Deux seuils pilotent la DEL ; le PTC coupe en cas de court.
4. Masse testée par la pince au châssis (`Pince_Trail`, §6.5.1).

> ✅ **Anomalie A6 corrigée sur les 6 canaux (v29).** La sortie TRAILER passe par un MOSFET **P-canal IRF4905** côté haut : la remorque reçoit la pleine tension de la batterie, moins ~1 V d'écrêtage sur les canaux d'éclairage (§7.2.3).

> **Pile interne :** au-delà de ~12 A (test complet d'une remorque incandescente), le bloc « BATT. EXT. » coupe le côté TRAILER et allume LED2 : passer sur batterie externe (§6.1).

> **Procédure — canal BRAKE (12 V aux.) en mode Remorque :** n'enfoncer le bouton que brièvement. Il alimente la ligne de charge de la batterie de la remorque ; une batterie déchargée tire plusieurs ampères (limités par le PTC) et viderait le pack 3S. Une remorque sans batterie affichera probablement « ouvert » (violet) : c'est normal.

---

## 6. Protections

### 6.1 Étage d'alimentation ✅ VÉRIFIÉ KICAD (12 oct., commit `e2f94fe`)

> Refait au **Chantier 7** (anomalies A7 et A8 corrigées, §7.6). Détail et tableaux de câblage : `Chantier_7_Alimentation_Thermique`.

**Chaîne de puissance :**

| Tronçon | Pattes | Étiquette (R6) |
|---|---|---|
| Batterie externe | J2.1 → F1 → SW1.1 | `Batt_Externe_Pos` (locale) |
| Pack interne | BMS3s1.P+ → **F8** → `Batt_Interne_Pos` → **R289** (shunt INT) → SW1.3 | `Batt_Interne_Pos`, `Batt_Int_Shunt` (locales) |
| Sélection | SW1.2 → Q1 (diode idéale U1) | `VPwr_Selection` → `V12_Bus_Raw` |
| Shunt de bus | `V12_Bus_Raw` → R221 → `V12_PostShunt` | locale |
| **Rail permanent** | `V12_PostShunt` → U6, PS1, C2, C7, U17, R234, R214, R218 | `V12_PostShunt` (locale) |
| **Côté TRAILER** | `V12_PostShunt` → **Q64** (IRF4905, S→D) → `V12_Rail` → SW2 | `V12_Rail` (hiérarchique) |

**Commande de Q64 :** grille par R288 (1 kΩ) depuis Q65.3 (D) ; R230 (10 kΩ) et DZ7 (1N5242B) entre grille et source. Q65 est commandé par `Bus_CutOff_Gate` (Q65.2) ; il est bloqué par le défaut de court-circuit du bus (`Court_Bus_N`, via R231) ou par le bloc « BATT. EXT. » (Q81).

**Ce qu'une coupure de Q64 garde alimenté :** affichage, alarmes, jauge et voltmètre (tout ce qui est sur `V12_PostShunt`).

| Réf. | Composant | Valeur | Rôle |
|---|---|---|---|
| BT1-BT3 | 18650 | — | Pack interne 3S1P — **cellules ≥ 20 A continu** (ex. Molicel P28A, à vérifier) |
| BMS3s1 | BMS 3S | — | **≥ 30-40 A continu**, port commun, équilibrage ; seuil de court-circuit > ~80 A crête (appel des filaments froids) |
| **F8** | Fusible ~25 A + porte-fusible | — | **Nouveau.** Branche INT ; hors PCB, dans le fil du pack |
| **R289** | 0,05 Ω, 20 W, TO-220 | — | **Nouveau.** Shunt INT (sur la plaque, même famille que R221) |
| P1 | Conn_Plug_2P | — | Prise de façade de la batterie externe (en parallèle avec J2) ; modèle à remplacer (§3.3.1) |
| J2 | Bornier 2 pos., 3,50 mm | — | Arrivée des fils de la batterie externe |
| F1 | Fusible | 15 A | Branche EXT |
| SW1 | SW_DPDT_x2 | — | SOURCE (INT / OFF / EXT). 2ᵉ pôle : SW1.6 = `V_Int_Actif` (bloc « BATT. EXT. ») ; SW1.4 NC |
| U1 | LM74610QDGKRQ1 | — | Diode idéale (SMD, exception R5). **U1.8 (CATHODE) → `V12_Bus_Raw`** |
| Q1 | IRLZ44N | — | MOSFET piloté par U1 |
| C1 | 2,2 µF / 50 V | — | Pompe de charge : **C1.1 → U1.7, C1.2 → U1.1, par fils** (A8) |
| C2, C3, C4 | 0,33 µF / 10 µF / 0,1 µF | — | Découplage |
| U6 | LM7805CT | 5 V | Régulateur → `V_Ref` (alimenté par `V12_PostShunt`) |
| **Q64** | **IRF4905** (P) | TO-220 | **Coupure côté TRAILER** (A7) |
| R230 / DZ7 / R288 | 10 kΩ / 1N5242B / 1 kΩ | — | G-S / protection G-S / série grille de Q64 |
| R242 | Shunt `Device:R_Shunt` | 50 A / 75 mV | Dans le négatif des batteries : `Batt_Neg` (1) → `GND` (4) ; Kelvin 2/3 → J1 |
| PS1 | Traco TMR 2-1211WIN | 5 V isolé, 2 W | Alimentation isolée du module de façade |
| C7 / C8 | SR655C106KAR / FG26X7R1E106KRT06 | 10 µF | Entrée / sortie PS1 |
| J1 | Conn_01x05 | — | Module voltmètre/ampèremètre (§7.4) |

**Bloc « BATT. EXT. » (sous-feuille `Limite_Pack_Int`) ✅ VÉRIFIÉ KICAD (12 oct.) :**
- **But :** sur pile interne, si le courant dépasse ce que le pack peut fournir, couper le côté TRAILER, mémoriser et allumer **LED2 (ambre, « BATT. EXT. »)**.
- **Mesure :** Q79 (2N3906) sur la chute de R289 ; seuil ~**12 A**, réglable jusqu'à ~20 A par **VR2** (3296W 1 kΩ, en diviseur — pattes 2 et 3 **non** pontées, exception).
- **Délai** ~0,2-0,3 s (R298 100 kΩ / C10 4,7 µF WIMA MKS4) ; bascule mémorisée Q80 (2N3904) / Q82 (2N3906), alimentée par `V_Int_Actif`.
- **Sortie :** Q81 (2N7000) tire `Bus_CutOff_Gate` (Q65.2) au bas → Q64 coupe `V12_Rail`.
- **Réarmement :** SW1 sur OFF ou EXT. En EXT, bloc inactif.
- Pièces : Q79-Q82, VR2, R290-R300, C10, LED2.

**Étiquettes de la feuille Alimentation :**
- **locales** : `Batt_Interne_Pos`, `Batt_Int_Shunt`, `Batt_Externe_Pos`, `VPwr_Selection`, `V12_Bus_Raw`, `V12_PostShunt`, `Batt_Neg`, `V_Int_Actif`, `Bus_CutOff_Gate`, `Court_Bus_N`, `VM_Shunt_Batt`, `VM_Shunt_Gnd`, `VM_5V_Iso`, `VM_0V_Iso` ;
- **hiérarchiques** : `V12_Rail`, `VM_Mesure`, `Court_Bus_Fault` ;
- **globale** : `V_Ref`.

> **PWR_FLAG sur `GND` à R242.4 — ne pas supprimer.**

> ⛔ `VM_5V_Iso` et `VM_0V_Iso` ne doivent **jamais** toucher `GND`.

**Condition sur le futur chargeur :** son négatif va sur **P− (`Batt_Neg`)**, pas sur `GND`. Le chargeur n'est pas encore au schéma. Charge du pack : **10 à 40 °C**.

### 6.2 Étage de protection par canal

1. **Isolement de direction** natif au câblage ; côté VEHICLE, pont diviseur + **2 PC817** par canal.
2. **PTC** par canal (mode Remorque) : MF-R700 (éclairage), MF-R300 (BRAKE), MF-R160 (AUX).
3. **TVS P6KE39A** par canal (traversant, depuis le 10 oct.).
4. **Shunt + LM339** par canal.
5. **Coupure d'injection F5** : un MOSFET IRLZ44N par canal, **côté bas** (§7.2.5). *Anomalie A2 corrigée sur les 6 canaux.*

### 6.3 Switch de mode manuel (DPDT)

SW2 dans `mode_coherence.kicad_sch`. Il coupe physiquement l'alimentation du côté non sélectionné.

### 6.4 Vérification de cohérence — ⚠️ ANOMALIE A9 (inopérante), ⏸ EN SUSPENS

**But :** alarme « Incohérence » (DEL du hub + buzzer) si MODE = TRAILER avec un adaptateur au port VEHICLE, ou MODE = VEHICLE avec un adaptateur au port TRAILER. Purement informatif : une erreur de SW2 n'endommage rien (ports mâle / femelle distincts).

**État dans le fichier (`mode_coherence.kicad_sch`, vérifié le 12 oct.) :** logique à diodes D1, D2 + Q2, Q3, Q4 (2N3904) → `Alerte_Incoherence` (**hiérarchique**). **Elle ne peut jamais s'allumer** :
- D1/D2 montées à l'envers pour une porte ET : le nœud de base de Q2 est tiré haut par R3 → Q2 conduit en permanence ;
- `Alerte_Incoherence` (collecteurs ouverts) n'a **aucun pull-up**, alors que le hub (Q60, D167) attend un niveau haut ;
- ce que l'adaptateur met sur la broche 8 n'a jamais été défini.

**Correction étudiée (non appliquée, ~30-45 min) :** pont détection ↔ masse dans la fiche de chaque adaptateur (aucun conducteur de câble) ; `Detect_x` tiré à `V_Ref` ; par branche, 2N3904 qui tient un nœud bas, nœud alimenté par `Mode_x` via 10 kΩ, diode vers `Alerte_Incoherence` + rappel 100 kΩ. Garde D1, D2, Q2, Q4, R1-R5 ; retire Q3 ; ajoute 3 × 10 kΩ + 1 × 100 kΩ. Détail : `Chantier_6_Connecteurs` §2septies.

### 6.5 Avertissements critiques (rangée du bas)

| Avertissement | DEL | Buzzer | Coupure | Réf. DEL |
|---|---|---|---|---|
| Inversion de polarité | ✅ | ❌ (passif) | Totale | D152 |
| Injection vers le véhicule | ✅ | ✅ | Port VEHICLE du canal | D154-D158 |
| Surtension | ✅ | ✅ | Totale | D154-D158 |
| Court-circuit bus | ✅ | ✅ | `V12_Rail` (côté TRAILER), **mémorisé** jusqu'à SW1 = OFF | D154-D158 |
| Incohérence de mode | ✅ | ✅ | Aucune — **inopérante (A9)** | D154-D158 |
| Masse dégradée | ✅ | ✅ (D149, v30) | Aucune | **D158** (D151 retirée) |
| **Pile interne trop sollicitée (« BATT. EXT. »)** | ✅ ambre | ❌ | `V12_Rail` (côté TRAILER), mémorisé | **LED2** |
| **Surchauffe charge 12 V aux. (BRAKE)** | ✅ ambre | ❌ | Bloque Q51 | D147 |
| **Surchauffe charge frein élect. (AUX)** | ✅ ambre | ❌ | Bloque Q43 | **LED1** (nouveau, §7.5.2) |
| Batterie faible | Segments rouges de la jauge | ❌ | Aucune | D25 |

> Les surchauffes passent par le nœud `Injection_x_Fault`. Selon la polarité attendue côté hub, une surchauffe pourrait aussi déclencher ou masquer l'alarme d'injection. Ce point reste ouvert (§10 pt 5).

### 6.5.1 Bloc_Masse — pince au châssis + comparateur à fenêtre ✅ VÉRIFIÉ KICAD (12 oct., commit `e2f94fe`)

Feuille `Detection_De_Masse` (fichier `MassDetection.kicad_sch`). **Refait en v30** (Chantier 6) ; l'ancienne mesure Kelvin (broche 9 → D149/D150 → U15) est abandonnée : valeur négative en VEHICLE, ne mesurait que l'adaptateur en TRAILER, chute des diodes qui masquait les petits défauts.

**Principe :**
- Une **pince au châssis** (1 conducteur dans chaque câble adaptateur, sortie en Y près de la prise 7 broches) arrive sur `Pince_Veh` (J3.9) ou `Pince_Trail` (J4.9).
- On mesure la chute **pince ↔ `GND`** : **positive** en TRAILER (mauvaise masse de remorque), **négative** en VEHICLE (mauvaise masse du véhicule).
- Le signal est décalé autour de ~2,5 V (le LM339 n'accepte pas d'entrée négative), puis comparé à une **fenêtre** symétrique.
- **Seuil visé : 0,5 V** de chute (R210 ≈ 3,2 kΩ), réglable jusqu'à ~1,5 V. Gain d'entrée ≈ 0,094 ; ±14,4 V sur la pince reste entre ~1,15 et 3,9 V à l'entrée de U15.
- Pince non branchée : 0 V (rappel 10 kΩ), donc pas d'alarme.

| Net | Pattes |
|---|---|
| `Pince_Veh` (hiér.) | R208.1, R303.1 |
| `Pince_Trail` (hiér.) | R211.1, R307.1 |
| `Masse_Sense` (locale) | R303.2, R307.2, R304.2, R209.1, C12.1, U15.4 (−), U15.7 (+) |
| `Seuil_Haut` (locale) | R305.2, R210.2, R210.3, U15.5 (+) |
| `Seuil_Bas` (locale) | R210.1, R306.1, U15.6 (−), U15.9 (+) |
| `Fenetre_Ok` (locale) | U15.1, U15.2, R301.2, U15.8 (−) |
| `Masse_Fault` (hiér.) | U15.14, R302.2 — **haut = défaut** |
| `V_Ref` (globale) | R304.1, R305.1, R301.1, R302.1, U15.3, U15.10 |
| `GND` (globale) | R208.2, R211.2, R209.2, C12.2, R306.2, U15.11, U15.12 |
| No Connect | U15.13 |

| Réf. | Valeur | Rôle |
|---|---|---|
| R208, R211 | 10 kΩ | Rappel d'entrée |
| R303, R307 | 330 kΩ | Série d'entrée |
| R304 / R209 | 62 kΩ (1 %) / 100 kΩ | Décalage |
| C12 | 0,1 µF SR205E104MARAP1 | Filtre |
| R305, R306 | 82 kΩ | Chaîne des seuils |
| R210 | Bourns 3296W 10 kΩ (rhéostat, 2+3 pontées) | Largeur de la fenêtre |
| R301, R302 | 10 kΩ | Pull-ups `Fenetre_Ok` / `Masse_Fault` |
| U15 | LM339, DIP-14 socket | Pattes 4/5/2 : seuil haut ; 6/7/1 : seuil bas ; 8/9/14 : inverseur ; 10/11/13 : inutilisée |

**Voltmètre :** SW9 position 9 (PINCE VEH, R308) et 10 (PINCE TRAIL, R309), §7.4.

> ⚠️ À vérifier à la réception du module : affiche-t-il une tension **négative** (cas VEHICLE) ? L'alarme, elle, détecte les deux sens.

### 6.6 Bloc Alarmes / Buzzer ✅ VÉRIFIÉ KICAD

**Hub `Alarmes_Buzzer_sch.kicad_sch` :**

| Réf. | Rôle |
|---|---|
| D154-D158 | 5 DEL rouges (injection, surtension, court-bus, incohérence, masse = **D158**, seule DEL de masse depuis v30) |
| D159-D164 | Porte OU des 6 `Injection_x_Fault` → `Injection_Fault` (locale) |
| D165-D168, **D149** | Porte OU vers le buzzer (`Alarme_Actif`) ; **D149 = masse** (v30, référence réutilisée) |
| Q58-Q63 | 2N7000 : 5 DEL + buzzer (Q63) |
| R223-R227 | 1 kΩ, DEL |
| R228, R229 | 4,7 kΩ, tirage |
| BZ1 | Buzzer actif (⚠️ empreinte manquante) |

**Polarité :** toutes les entrées du hub sont **actives hautes** (grilles des 2N7000). `Masse_Fault` l'est depuis v30 (elle était inversée : D158 allumée en temps normal).

**Signaux entrants (tous hiérarchiques) :**
- `Injection_Tail_Fault`, `Injection_Brake_Fault`, `Injection_Left_Fault`, `Injection_Right_Fault`, `Injection_Backup_Fault`, `Injection_Aux_Fault` ;
- `Court_Bus_Fault`, `Surtension_Fault`, `Alerte_Incoherence`, `Masse_Fault`.

**Alarmes construites dans `Alimentation` :**

| Sous-bloc | Composants |
|---|---|
| Inversion (passif) | D152, D153, R212 (1 kΩ), R213 (100 Ω) |
| Surtension | U16 (LM339), R214 (100 kΩ), R215 (10 kΩ), R216 (pot 10 kΩ), R217 (4,7 kΩ), C5 |
| Court-circuit bus | R221 (0,03 Ω/20 W TO-220), Q55 (2N3906), Q56/Q57 (2N7000), R218/R220/R222 (4,7 kΩ), **R219 (100 kΩ)** ; **mémoire** : D180 (1N4148, `Court_Bus_Fault` → grille de Q56), C11 (0,1 µF) — réarmement par SW1 = OFF |
| Coupure `V12_Rail` | **Q64 (IRF4905, côté haut)**, Q65 (2N7000), R230 (10 kΩ), R231, R288 (1 kΩ), DZ7 (1N5242B) — A7 corrigée (§6.1) |

---

## 7. Interface utilisateur

- **6 DEL RGB** (une par canal), §5.0.
- **6 boutons à verrouillage** (SW3-SW8), côté TRAILER.
- **SW2** MODE ; **SW1** SOURCE.
- **Rangée du bas :** 5 DEL d'alarme (D154-D158, dont MASSE = D158), INVERSION (D152), **2 DEL SURCHAUFFE ambre** (D147 = 12 V aux. ; LED1 = frein électrique), **DEL « BATT. EXT. » ambre (LED2)**, bargraph 10 segments (D25), buzzer BZ1.
- **Voltmètre/ampèremètre** + **sélecteur rotatif SW9** (10 positions), §7.4.
- **Jauge de batterie** LM3914 (U17), §A.6.
- Sérigraphie : « **12V AUX** » sur le canal BRAKE ; « **FREIN ÉLECT.** » sur AUX.

### 7.1 Nomenclature des étiquettes de canal

| Canal (fonction) | Suffixe | Lame J560 | VEHICLE | TRAILER |
|---|---|---|---|---|
| Position | `TAIL` | 6 | `Veh_Tail` | `Trail_Tail` |
| **12 V auxiliaire** ⚠️ *corrigé v28* | `BRAKE` | 2 | `Veh_Brake` | `Trail_Brake` |
| Gauche | `LEFT` | 4 | `Veh_Left` | `Trail_Left` |
| Droit | `RIGHT` | 5 | `Veh_Right` | `Trail_Right` |
| Recul | `BACKUP` | 7 | `Veh_Backup` | `Trail_Backup` |
| **Frein électrique** | `AUX` | 3 | `Veh_Aux` | `Trail_Aux` |

Les signaux qui franchissent une frontière de feuille sont **hiérarchiques**. `GND`, `V_Ref`, `Mode_Vehicule`, `Mode_Trailer` et `V_Arm_Trailer` sont **globaux**.

Isolement : **2× PC817 par canal** : OC1/OC2 (Tail), OC3/OC4 (Left), OC5/OC6 (Right), OC7/OC8 (Backup), OC9/OC10 (Aux), OC11/OC12 (Brake).

### 7.2 Canal TAIL — modèle des canaux d'éclairage

> Canal de référence ✅ **re-vérifié le 9 oct.** (A1, A2, A6 corrigées). Il a servi de modèle aux 4 autres canaux d'éclairage. Correspondance des références : §7.3 et `Chantier_5_Corrections_5_canaux`.

**Structure :** VEHICLE (lecture), TRAILER (source), Bloc de Test. **5 comparateurs par canal**, donc 2 LM339.

#### 7.2.1 Signaux internes (TAIL)

| Signal | Type | Rôle |
|---|---|---|
| `Veh_Tail`, `Trail_Tail` | Hiérarchique | Vers les ports |
| `Court_Tail_Vehic`, `Ouvert_Tail_Vehic` | Hiérarchique | VEHICLE → Bloc_Test |
| `Court_Tail_Trailer`, `Ouvert_Tail_Trailer`, `Btn_Tail_Etat` | Hiérarchique | TRAILER → Bloc_Test |
| `Injection_Tail_Fault` | Hiérarchique | → Alarmes |
| `Court_V_Actif`, `Court_T_Actif`, `Ouvert_V_Actif`, `Ouvert_T_Actif`, `Court_Tail`, `Ouvert_Tail`, `Bleu_Tail`, `Not_Btn_Tail` | Locale | Nœuds logiques |

#### 7.2.2 Sous-bloc VEHICLE (TAIL) ✅ VÉRIFIÉ KICAD (9 oct.)

| Réf. | Composant | Valeur | Empreinte |
|---|---|---|---|
| **R21** (RL_TAIL) | Charge | **12 Ω / 25 W**, boîtier alu (Chantier 1) | Bornier Phoenix PT-1,5-2-3.5 (résistance sur plaque alu) |
| **R22** (RSH_TAIL) | Shunt | **0,2 Ω / 5 W** (Chantier 1) | `R_Axial_Power_L25.0mm_W9.0mm_P27.94mm` |
| U2 | LM339 | — | DIP-14 + socket |
| OC1, OC2 | PC817 — `OC1.2` → `U2.1` (ouvert), `OC2.2` → `U2.13` (court) : correction A1 | — | DIP-4 |
| Q11 | IRLZ44N (injection), **côté bas** : `Veh_Tail` → R21 → `Q11.2 (D)` ; `Q11.3 (S)` → R22 → `GND` (correction A2) | — | TO-220 |
| R23-R25 | Pots 3296W | — | `TRIM_3296W1103LF` |
| R26 | Pull-up injection | — | — |
| R37 / R38 | 100 kΩ / 10 kΩ | Diviseur anti-oscillation sur `Veh_Tail` | — |

Point ouvert (§10 pt 5) : polarité du comparateur d'injection (R37/R38 sur U2A pin 5 (+), seuil sur pin 4 (−)).

#### 7.2.3 Sous-bloc TRAILER (TAIL) ✅ VÉRIFIÉ KICAD (9 oct.)

**Puissance (correction A6) :** `V_Arm_Trailer` → F2 → `Q12.3 (S)` (IRF4905, P-canal), puis `Q12.2 (D)` → R33 ∥ D170 → `Trail_Tail`.

**Commande :** `Btn_Tail_Etat` → `Q69.2 (G)` (2N7000) ; `Q69.3 (D)` → R253 (1 kΩ) → `Q12.1 (G)`. R254 (10 kΩ) et DZ2 (1N5242B, `DZ2.1 (K)` côté source) entre G et S.

**Mesure (Chantier 2) :**
- **Ouvert, par le courant :**
  - Q70 (2N3906) mesure la chute dans R33 (12 Ω) : `Q70.1 (E)` en amont, `Q70.2 (B)` → R255 (1 kΩ) → `Trail_Tail`.
  - Dès ~50 mA, Q70 conduit : `Q70.3 (C)` → R256 (10 kΩ) → `I_Present_Tail` → R257 (2,7 kΩ) → `GND`, soit ≈ 1,9 à 2,9 V.
  - Sans courant : 0 V.
  - D170 (MUR1520G) écrête la chute dans R33 à ~0,8–1 V à fort courant.
- **Court, par la tension :** `Trail_Tail` → R258 (39 kΩ) → `Court_Sense_Tail` → R259 (10 kΩ) → `GND` (÷4,9).
- **Comparateurs :** `U3.6` ← `I_Present_Tail`, `U3.7` ← R31 ≈ 1,0 V ; `U3.10` ← `Court_Sense_Tail`, `U3.11` ← R32 ≈ 0,6 V. Toutes les entrées du LM339 restent sous 3 V.

| Réf. | Composant | Valeur |
|---|---|---|
| SW3 | Bouton push-push | — |
| Q12 | **IRF4905PBF** (P-canal) | TO-220 |
| Q69 | 2N7000 (commande de grille) | — |
| R253 / R254 | Grille / grille-source | 1 kΩ / 10 kΩ |
| DZ2 | 1N5242B | 12 V |
| R33 | Shunt TRAILER | **12 Ω / 1 W** (DIN0414) |
| D170 | **MUR1520G** (TO-220AC, languette = cathode → **isolant** sur la plaque) | 15 A / 200 V |
| Q70 | 2N3906 (capteur de courant) | — |
| R255 / R256 / R257 | Base / collecteur / vers la masse | 1 kΩ / 10 kΩ / 2,7 kΩ |
| R258 / R259 | Diviseur « court » | 39 kΩ / 10 kΩ |
| U3 | LM339 (sections A et D neutralisées, NC) | — |
| R31, R32 | Pots | ≈ 1,0 V / ≈ 0,6 V |
| F2 | Polyfuse | **MF-R700** (7 A / 14 A) |
| D148 | **P6KE39A** (`D148.1 (K)` sur `Trail_Tail`) | DO-15 traversant |
| R34 | 10 kΩ pull-down `Btn_Tail_Etat` | — |

> ⚠️ **À valider au banc :** avec une ampoule DEL (~100 mA), `I_Present_Tail` doit rester > 1,5 V. Repli : deux diodes en série à la place de D170 (~1,5 V de chute).
> **Coûts acceptés :** ~1 V de chute vers la remorque ; jusqu'à ~5 W dans D170 à 5 A, d'où le montage sur la plaque.

#### 7.2.4 Bloc de Test (logique d'affichage)

**23 × 1N4148** (+1 en v29), **13 × 4,7 kΩ**, 6 × 2N7000 et 3 × 330 Ω par canal. **D171** (TAIL) : `D171.1 (K)` → `Btn_Tail_Etat`, `D171.2 (A)` → `Court_T_Actif`. Elle bloque l'affichage « court » quand le bouton est relâché (`Trail_Tail` est alors à 0 V). Pour TAIL :
- étage 1 (portes ET/OU) : D3-D17, R5, R6, R8-R13, Q5 ;
- étage 2 (priorité rouge) : D18-D24, R14, R15, R18, R19, Q9, Q10 ;
- étage 3 (DEL) : Q6/Q7/Q8, R7/R16/R17 (330 Ω), D26 (RGB anode commune).

**Règle des portes à diodes :**
- **ET** : nœud tiré **haut**, **cathode côté entrée**.
- **OU** : nœud tiré **bas**, **anode côté entrée**.

#### 7.2.5 Coupure d'injection (F5)

Sur chaque canal, la sortie du comparateur d'injection, le pull-up, la grille du MOSFET d'injection et l'étiquette `Injection_x_Fault` sont sur le même nœud (corrigé le 19 sept.).

> ✅ **Anomalie A2 corrigée sur les 6 canaux (v29).** Le MOSFET d'injection est **côté bas** (entre RL et le shunt), avec une grille à 5 V. Q54 (BRAKE) et Q66 (AUX) tirent toujours le nœud `Injection_x_Fault` au bas en cas de surchauffe.

**Canal BRAKE — protection thermique :**
- TH1 (KSD9700) et R206 (100 kΩ) forment le nœud `Surchauffe_Brake`.
- Q54 (drain sur `Injection_Brake_Fault`) bloque Q51.
- Q53 allume D147 (ambre).
- *À 12 W (Chantier 1), cette protection est probablement superflue. Elle est conservée en attendant une décision.*

> Incident du 19 sept. (drain de Q54 isolé par le retrait d'une étiquette) → origine de R9.

### 7.3 Correspondance des références par canal ✅ VÉRIFIÉ KICAD (9 oct.)

| Rôle | TAIL | LEFT | RIGHT | BACKUP | AUX | BRAKE |
|---|---|---|---|---|---|---|
| RL (charge VEHICLE) | R21 | R64 | R89 | R122 | R155 (4,7 Ω/50 W) | R188 |
| Shunt VEHICLE | R22 | R65 | R90 | R123 | R156 (0,05 Ω) | R189 |
| LM339 VEHICLE | U2 | U4 | U7 | U9 | U11 | U13 |
| LM339 TRAILER | U3 | U5 | U8 | U10 | U12 | U14 |
| MOSFET injection (côté bas) | Q11 | Q19 | Q27 | Q35 | Q43 | Q51 |
| MOSFET sortie TRAILER (IRF4905) | Q12 | Q20 | Q28 | Q36 | Q44 | Q52 |
| Commande de grille (2N7000) | Q69 | Q73 | Q75 | Q77 | Q68 | Q71 |
| Grille 1 kΩ / G-S 10 kΩ | R253/R254 | R267/R268 | R274/R275 | R281/R282 | R247/R248 | R260/R261 |
| Zener G-S (1N5242B) | DZ2 | DZ4 | DZ5 | DZ6 | DZ1 | DZ3 |
| Shunt / charge TRAILER | R33 (12 Ω/1 W) | R67 | R102 | R135 | R168 (12 Ω/50 W) | R201 |
| Diode d'écrêtage (MUR1520G) | D170 | D174 | D176 | D178 | — | D172 |
| Capteur PNP (2N3906) | Q70 | Q74 | Q76 | Q78 | — | Q72 |
| Base / collecteur / masse | R255-R257 | R269-R271 | R276-R278 | R283-R285 | — | R262-R264 |
| Diviseur « court » 39k/10k | R258/R259 | R272/R273 | R279/R280 | R286/R287 | R249/R250 | R265/R266 |
| Diode Bloc_Test (court × bouton) | D171 | D175 | D177 | D179 | D169 | D173 |
| Optocoupleurs (ouvert / court) | OC1/OC2 | OC4/OC3 | OC6/OC5 | OC8/OC7 | OC10/OC9 | OC12/OC11 |
| Polyfuse | F2 (MF-R700) | F3 (MF-R700) | F4 (MF-R700) | F5 (MF-R700) | F6 (MF-R160) | F7 (MF-R300) |
| Bouton | SW3 | SW4 | SW5 | SW6 | SW7 | SW8 |

Sur les 5 canaux d'éclairage : RL = 12 Ω / 25 W et shunt VEHICLE = 0,2 Ω / 5 W (Chantier 1). AUX garde sa mesure propre (§7.5.3).

### 7.4 Voltmètre / ampèremètre de façade ✅ (Chantier 3, ERC 0/0 le 29 sept.)

**Module :** générique « 4 chiffres, 0-100 V / 50 A, courant positif et négatif ». Shunt externe 50 A / 75 mV inclus ; impédance d'entrée 400 kΩ ; alimentation **isolée** obligatoire. Critères d'achat : version **50 A** (et non 20 A), résolution 0,01 A, mention « isolated power supply ».

**Principe :**
- Le **courant** est mesuré par R242, dans le négatif des batteries. Il n'a de sens qu'en mode TRAILER.
- La **tension** est choisie par **SW9** (Lorlin **CKS1503** : 1 pôle, 12 positions, contacts non court-circuitants, plaqués or, IP67).
- Chaque entrée de SW9 passe par une résistance de **1 kΩ ½ W** (protection, F5). 10 positions utilisées sur 12.

| SW9 | Position | Résistance | Source (étiquette **locale**, page racine) |
|---|---|---|---|
| 1 | **VEH 12V** (défaut) | R241 | `Veh_Brake` (12 V aux. du véhicule, lu sous ~1 A) |
| 2 | BATT | R235 | `V12_Rail` |
| 3 | TAIL | R236 | `Veh_Tail` |
| 4 | LEFT | R237 | `Veh_Left` |
| 5 | RIGHT | R238 | `Veh_Right` |
| 6 | BACKUP | **R240** | `Veh_Backup` |
| 7 | AUX | **R239** | `Veh_Aux` |
| **8** | **TRAIL AUX** | **R252** | `Trail_Aux`. **Nouveau 7 oct.** Tension sur les aimants pour la mesure de résistance (§7.5.4) |
| **9** | **PINCE VEH** | **R308** | `Pince_Veh` (chute de masse côté véhicule, §6.5.1) — v30 |
| **10** | **PINCE TRAIL** | **R309** | `Pince_Trail` (chute de masse côté remorque) — v30 |
| 11-12 | — | — | No Connect |
| 13 | commun | — | `VM_Mesure` (**locale** sur la racine → patte du rectangle Alimentation → **hiérarchique** sur J1.3) |

> ⚠️ **Corrigé le 10 oct. :** R240 est en position 6 (BACKUP) et R239 en position 7 (AUX). Les versions précédentes de ce tableau inversaient ces deux résistances. Aucun changement au schéma : seule la documentation était fausse.

**J1 :**

| Patte | Signal |
|---|---|
| 1 | `VM_5V_Iso` |
| 2 | `VM_0V_Iso` |
| 3 | `VM_Mesure` |
| 4 | `VM_Shunt_Batt` |
| 5 | `VM_Shunt_Gnd` |

Couleurs des fils et côté du shunt du commun à confirmer **à la réception du module**.

**Règles de lecture :**
- 0 V en position 1, contact mis, sur une prise 7 broches = défaut entre la batterie du véhicule et la prise. (Un 0 V seul n'est pas un défaut : 4/5 broches ou relais de charge.)
- Courant d'un canal = lecture bouton enfoncé − lecture de base I₀ sans bouton (I₀ ≈ 0,1-0,3 A, consommation de l'appareil).
- En mode VEHICLE, l'affichage du courant n'a pas de sens.

### 7.5 Canal AUX — frein électrique ✅ VÉRIFIÉ KICAD (Chantier 4, 30 sept. → 7 oct., commit `b119a28`)

#### 7.5.1 Principes

- **VEHICLE** : le contrôleur de freins doit voir une remorque « connectée ». On lui présente **4,7 Ω**, à l'intérieur de la fenêtre « connecté » de tous les contrôleurs documentés (§7.5.5). La charge est protégée en température. Les impulsions PWM sont lissées par un étireur.
- **TRAILER** : on **ne simule pas un freinage**. On mesure la **résistance des électroaimants à ~1 A**. Résultat : ouvert / normal / court sur la DEL, et une valeur en ohms par le voltmètre (position 8) et l'ampèremètre. Le 3 essieux devient testable. L'usure mécanique reste hors de portée.

#### 7.5.2 `Vehicule_Aux` (vérifié le 4 oct., `e59e3f7`)

**Puissance :** `Veh_Aux` → **R155** (4,7 Ω / 50 W alu) → **Q43** D→S (IRLZ44N, **côté bas**, correction A2) → **R156** (0,05 Ω) → `GND`. La grille de Q43 est sur `Injection_Aux_Fault`.

**Protection thermique :**
- **SW10** (KSD9700 NC, sur le dissipateur de R155) et **R243** (100 kΩ) forment le nœud de surchauffe.
- Il commande **Q66** (2N7000), qui tire `Injection_Aux_Fault` au bas et bloque Q43.
- Il commande aussi **Q67** (2N7000), qui allume **LED1** (ambre 1,8 mm, K = patte 1, A = patte 2 → `V_Ref`) via **R244** (330 Ω).

**Étireur d'impulsions (pour les contrôleurs PWM) :**
- Sortie « normal » U11.1 → **R158** (330 kΩ) et **C9** (4,7 µF / 63 V film WIMA MKS4) → U11.9 (étiquette locale `To_C9_1`).
- **R245 / R246** (10 kΩ / 10 kΩ) fixent **2,5 V** sur U11.8.
- U11.14 → OC10.2 (étiquette locale `To_U11_14`).
- Maintien ≈ 0,69 × 330 kΩ × 4,7 µF ≈ **1,07 s**, ce qui couvre tout PWM ≥ 2 Hz et les impulsions de détection (Ram : 1 ms, 3×/s).

**Opto court (correction A1) :** OC9.2 → U11.13 / R159.1. R158.2 et R159.2 → `V_Ref`.

#### 7.5.3 `Trailer_Aux` (vérifié le 6 oct., `c120fb7`) + `Bloc_Test_Aux` (7 oct., `ee352dd`)

**Puissance (correction A6) :** `V_Arm_Trailer` → **F6** (Bourns MF-R160) → **Q44** S→D (**IRF4905PBF, P-canal**, TO-220) → **SW11** (KSD9700, même dissipateur) → **R168** (12 Ω / 50 W alu) → `Trail_Aux`. D122 (P6KE39A depuis le 10 oct.) est sur `Trail_Aux`.

**Commande de grille :**
- `Btn_Aux_Etat` → **Q68** (2N7000) G. Q68 D → **R247** (1 kΩ) → grille de Q44.
- **R248** (10 kΩ) entre grille et source : bloque Q44 au repos.
- **DZ1** (1N5242B, 12 V), K → source, A → grille : limite V_GS (max ±20 V).

**Mesure (un seul diviseur pour ouvert et court) :** `Trail_Aux` → **R249** (39 kΩ) → `Mesure_Aux_Trailer` (**locale**) → **R250** (10 kΩ) → `GND`. Rapport ÷ 4,9.
- **Ouvert :** U12.7 (+) ← mesure ; U12.6 (−) ← curseur de R169. Sortie haute si mesure > seuil.
- **Court :** U12.10 (−) ← mesure ; U12.11 (+) ← curseur de R170. R170 est alimenté par **R251** (100 kΩ) depuis `V_Ref`, d'où une plage de 0 à 0,45 V.

| Situation | `Trail_Aux` | Mesure |
|---|---|---|
| Ouvert | 9 à 13,8 V | 1,84 à 2,82 V |
| Normal (0,5 à 4 Ω) | 0,36 à 3,45 V | 0,073 à 0,70 V |
| Court (< ~0,1 Ω) | < 0,11 V | < 0,023 V |

**Calibration :** curseur de R169 ≈ **1,25 V** ; curseur de R170 ≈ **0,045 V**. Courant de test 0,72 à 1,15 A ; R168 dissipe ~12 W (15,9 W en court, limité par F6 et SW11).

**`Bloc_Test_Aux` :** **D169** (1N4148), K → `Btn_Aux_Etat`, A → `Court_T_Actif_Aux`. Le « court » n'est signalé que bouton enfoncé : au repos, `Trail_Aux` est à ~0 V et serait lu comme un court.

#### 7.5.4 Page racine et procédure « un aimant manquant »

**R252** (1 kΩ ½ W) relie `Trail_Aux` (**locale**, même net que J4.7) à **SW9.8**.

**Procédure :**
1. Mode TRAILER, aucun bouton : lire **I₀**.
2. Enfoncer AUX, SW9 en position 8 : lire **V** et **I**.
3. **R = V / (I − I₀)**.

| Essieux | Aimants | R attendue (aimant ≈ 3-4 Ω) | Avec **un aimant coupé** |
|---|---|---|---|
| 1 | 2 | 1,5-2,0 Ω | 3,0-4,0 Ω (+100 %) |
| 2 | 4 | 0,75-1,0 Ω | 1,0-1,33 Ω (+33 %) |
| 3 | 6 | 0,50-0,67 Ω | 0,60-0,80 Ω (+20 %) |

> La meilleure référence est **la même remorque mesurée en bon état** (noter la valeur). Un écart > ~15 % sur 3 essieux mérite une inspection. La résistance du faisceau s'ajoute à la mesure.

#### 7.5.5 Détection des contrôleurs de freins (recherche du 30 sept.)

| Contrôleur | Détection | Charge suffisante |
|---|---|---|
| Après-marché (Tekonsha, Curt…) | Faible courant | Une lampe-témoin suffit |
| Ram (intégré) | Impulsion 12 V, 1 ms, 3×/s ; PWM 250 Hz | 2-6 Ω = connecté ; ≤ 1,5 Ω = court |
| GM (intégré) | Impulsion | Seuil non publié |
| Ford (intégré) | Continuité ligne ↔ masse | Seuil non publié |

**Cible : tous les véhicules.** R155 = 4,7 Ω reste à **valider sur le terrain** (GM, Ford, Ram, après-marché).

### 7.6 Anomalies (A1-A9)

> **Aucune de ces anomalies n'est détectable par l'ERC** : le câblage est électriquement « valide », c'est la fonction qui est fausse. Elles ont été trouvées par analyse des nets dans les fichiers. **Leçon : un ERC à 0/0 ne prouve pas qu'un étage fonctionne.**

| # | Anomalie | Effet | Portée | État |
|---|---|---|---|---|
| **A1** | Cathode (patte 2) des PC817 sur `V_Ref` au lieu de la sortie du comparateur | L'opto ne s'allume jamais ; aucun défaut VEHICLE n'atteint le Bloc de Test | 6 canaux VEHICLE | ✅ 6 canaux (AUX 4 oct. ; autres 9 oct.) |
| **A2** | MOSFET d'injection côté haut, grille à 5 V | Suiveur de source : courant plafonné, la charge RL n'est pas vue par le véhicule | 6 canaux VEHICLE | ✅ 6 canaux (AUX 1er oct. ; autres 9 oct.) |
| **A6** | MOSFET de sortie TRAILER N-canal, grille = bouton à 5 V | La remorque ne reçoit que ~2-3 V | 6 canaux TRAILER (vérifié sur Q12, Q20, Q28, Q36, Q44, Q52) | ✅ 6 canaux (AUX 6 oct. ; autres 9 oct.) |
| ~~A3~~, ~~A5~~ | D122, D147 « non raccordées » | — | — | ❌ Fausses alertes (bogue de l'outil d'extraction, corrigé) |
| **A7** | Q64 (coupure du bus) N-canal côté haut, grille à 5 V ; U6 alimenté **après** Q64 | `V12_Rail` plafonné à ~3 V ; **l'appareil ne pouvait pas démarrer** ; Q64 coupait tout l'appareil | Alimentation | ✅ 11 oct. (IRF4905, `V12_PostShunt`) |
| **A8** | C1 (pompe de charge du LM74610) relié par des **bus** | Q1 jamais commandé, tout le courant dans sa diode interne (~12-15 W) | Alimentation | ✅ 11 oct. (fils) |
| **A9** | Vérification de cohérence : diodes à l'envers + aucun pull-up sur `Alerte_Incoherence` | L'alarme « Incohérence » ne s'allume jamais | Mode_Coherence | ⏸ En suspens (§6.4) |
| (v30) | `Masse_Fault` actif bas alors que Q62 attend un niveau haut | D158 allumée en temps normal, éteinte en défaut | MassDetection ↔ hub | ✅ 12 oct. (refonte §6.5.1) |

**Correction A1 appliquée (9 oct.).** opto.2 est relié au nœud de sortie du comparateur et du pull-up (étiquettes locales `Opto_Ouvert_x` / `Opto_Court_x` sur BRAKE, LEFT, RIGHT, BACKUP) :

| Canal | Ouvert | Court |
|---|---|---|
| TAIL | OC1.2 → U2.1 / R27.1 | OC2.2 → U2.13 / R28.1 |
| BRAKE | OC12.2 → U13.1 / R191.1 | OC11.2 → U13.13 / R192.1 |
| LEFT | OC4.2 → U4.1 / R55.1 | OC3.2 → U4.13 / R56.1 |
| RIGHT | OC6.2 → U7.1 / R92.1 | OC5.2 → U7.13 / R93.1 |
| BACKUP | OC8.2 → U9.1 / R125.1 | OC7.2 → U9.13 / R126.1 |

> Question ouverte : les clignotants (~1-2 Hz) feront clignoter « ouvert » en mode VEHICLE. Un étireur (comme sur AUX) reste à étudier.

**Correction A2 appliquée (9 oct.) :** même topologie que Q43 : `Veh_x` → RL → MOSFET D→S → shunt → `GND`, grille sur `Injection_x_Fault`. Étiquettes locales `Qn_Drain` et `x_Vehic_Shunt_Haut`. Le diviseur d'injection reste sur `Veh_x`, en amont de RL.

**Correction A6 appliquée (9 oct.) :** même topologie que Q44 (IRF4905 côté haut, 2N7000, 1 kΩ, 10 kΩ, zener 12 V), plus la mesure du Chantier 2 (§7.2.3).

**Méthode (9 oct.) :**
- TAIL a été câblé à la main, geste par geste, puis vérifié.
- BRAKE, LEFT, RIGHT et BACKUP ont été générés par script à partir de TAIL (Trailer_x), ou modifiés en place (Vehicule_x, Bloc_Test_x).
- Chaque feuille reçue a ensuite été comparée au fichier généré : partition des connexions et champs.

> **Incident du 9 oct. (envoi GitHub) :** une recherche Windows a mélangé des copies anciennes des sous-dossiers `.history`, `Backup` et `_restore_backup…`. L'envoi a remplacé 24 feuilles par leurs versions des 28 septembre et 29 août. Réparé le jour même. **Règle :** envoyer uniquement les fichiers du dossier principal du projet.

---

## 8. Boîtier et connectique physique

- Boîtier **IP54** au minimum.
- **Connecteurs J3 / J4** en façade : modèle en suspens (§3.3.1). Câbles adaptateurs 4/5/7 broches en **SOOW 10 conducteurs**, avec le **fil de pince** sorti en Y (renforcé) près de la prise 7 broches.
- **Prise de batterie externe P1** : à remplacer (§3.3.1). Pinces AWG 14 minimum, porte-fusible en ligne ; USB-C de recharge.
- **Plaque d'aluminium** pour les résistances à boîtier alu, raccordées au PCB par **borniers Phoenix PT 1,5/2-3,5** : 5 × 12 Ω/25 W (RL éclairage), R155 (4,7 Ω/50 W) et R168 (12 Ω/50 W). Aussi sur la plaque : R221, **R289** (shunts TO-220).
- **Diodes MUR1520G (D170, D172, D174, D176, D178) et IRF4905 (7, dont Q64) sur la plaque :** languette = cathode (MUR1520G) / drain (IRF4905) → **isolant** et rondelle isolante.
- **Thermostats KSD9700 en fils volants :** TH1 (BRAKE), SW10 (sur R155) et SW11 (dissipateur Q44 / R168).
- **F1 et F8** hors PCB (porte-fusibles).
- **Façade :** module voltmètre/ampèremètre (découpe 45,5 × 26 mm) et SW9 (axe 6 mm). Connecteur de câblage sur le PCB pour SW9 (10 fils + commun), à définir.
- **Rangée du bas :** 5 DEL d'alarme, INVERSION, 2 DEL SURCHAUFFE ambre, DEL BATT. EXT. ambre, bargraph, buzzer.
- Pack 3×18650 : ~65 × 60 × 20 mm, **dans un compartiment séparé de la plaque** (décharge < 60 °C).

**Thermique (Chantier 7, ⏳ attend les dimensions du boîtier) :**
- Chaleur dominante en mode VEHICLE (fournie par le véhicule) ; recalcul à 14,4 V : RL éclairage 17,3 W chacune, R155 44,1 W (marge mince sur 50 W) ; **pire cas ~131 W**.
- **Ventilateur déconseillé.** Dissipation **par la paroi** : boîtier tout aluminium à ailettes (A) ou boîtier plastique + plaque à ailettes en guise de fond (B). Température de surface > 60 °C = risque de brûlure → B plus défendable.
- **Trou de protection :** TAIL, LEFT, RIGHT, BACKUP n'ont pas de thermostat (seuls BRAKE, R155, Q44/R168 en ont).
- LM339N / LM3914N : 0-70 °C ; piste LM2901N (−40 à +125 °C) à confirmer.
- PTC éloignés des sources chaudes sur le PCB.

---

## 9. Suggestions de composants (historique — l'Annexe A fait foi)

| Fonction | Composant |
|---|---|
| Comparateur | LM339N, DIP-14, socket turned-pin |
| MOSFET N de puissance | IRLZ44N, TO-220 |
| **MOSFET P de sortie TRAILER** | **IRF4905PBF**, TO-220 (6 canaux) |
| **Diode d'écrêtage du capteur de courant** | onsemi **MUR1520G** (15 A, 200 V, TO-220AC, actif) |
| **Capteur de courant TRAILER** | **2N3906** (PNP) |
| **Zener de protection G-S** | **1N5242B** (12 V, DO-35) |
| Transistor logique | 2N7000, TO-92 |
| BJT | 2N3904 ×4 ; 2N3906 ×8 (dont capteurs TRAILER ×5, Q55, Q79, Q82) |
| Diode logique | 1N4148 |
| Isolement | PC817 ×2 par canal |
| TVS | **P6KE39A** par canal ; SMCJ36CA en entrée |
| Thermostat | KSD9700 NC ~90-100 °C |
| PTC | Bourns **MF-R700** (éclairage ×4), **MF-R300** (BRAKE), **MF-R160** (AUX) |
| Charges de puissance | 12 Ω/25 W alu (×5) ; 4,7 Ω/50 W alu (R155) ; 12 Ω/50 W alu TE **HSA5012RJ** (R168) |
| Condensateur d'étireur | WIMA **MKS4C044704D00KSSD** (4,7 µF / 63 V, ±10 %) |
| Sélecteur voltmètre | Lorlin **CKS1503** |
| Alim. isolée | Traco **TMR 2-1211WIN** |
| Jauge | LM3914N + OSX10201-GYR1 + Bourns 3296W-1-102LF |
| Potentiomètres | Bourns 3296W, pattes 2+3 pontées |

---

## 10. Points à valider avant fabrication

1. Brochages des variantes 5 broches du marché visé.
2. ~~Courant nominal par canal~~ → ✅ **tranché (Chantier 1)** pour VEHICLE ; côté TRAILER, mesure résolue (§10.3), calibre des PTC à fixer.
3. Tenue ISO 7637-2 si visée.
4. ~~Valeur de la charge simulée~~ → ✅ 12 Ω/25 W (éclairage), 4,7 Ω/50 W (AUX). **Essai sur un véhicule Ford à faire** (seuil non publié).
5. **Validation pratique des seuils**, canal par canal, en particulier :
   - polarité du comparateur d'injection (§7.2.2) ;
   - interaction surchauffe ↔ alarme d'injection (BRAKE et AUX) ;
   - marge de l'étireur AUX ;
   - R169 / R170.
6. Fiabilité mécanique des commutateurs.
7. ~~Retour de masse commun~~ → ✅ **recalculé (Chantier 6)** : pire cas réaliste TRAILER ~14-18 A → 2 contacts de 15 A en parallèle (§3.3.1).
8. **27 composants sans empreinte** (§A.8.4) — tous liés au Chantier 8.
9. Renumérotation globale de fin de projet (dont VR1 → R2xx).
10. ~~Report des corrections A1, A2 et A6 sur les 5 autres canaux~~ → ✅ **fait le 9 oct.** (§7.6).
11. ~~R5 : 6 TVS SMBJ33A en SMD~~ → ✅ **remplacées le 10 oct. par des Littelfuse P6KE39A** (600 W, VRWM 33,3 V, DO-15 traversant), symbole `PCM_Diode_TVS_AKL:P6KE39A`, empreinte `PCM_Diode_THT_AKL:D_DO-15_P10.16mm_Horizontal_Zener`. Cathode (patte 1) sur `Trail_x` vérifiée sur les 6 canaux (commit `5e470b7`).
12. **Valider au banc le capteur PNP** (§7.2.3).
13. **Mesure de masse :** régler R210 (~3,2 kΩ pour 0,5 V) ; vérifier l'affichage négatif du module (§6.5.1).
14. **Bloc « BATT. EXT. » :** régler VR2 (~12 A) au banc.
15. **Choisir le connecteur J3/J4 et la prise P1** (§3.3.1).

### 10.0 État ERC ✅

**0 erreur, 0 avertissement**, relevé le 29 sept., le 30 sept., les 4, 7, 9 et 10 oct., et le **12 oct.** après les chantiers 6 et 7 (commit `e2f94fe`).
- Le `lib_symbol_mismatch` de Q54 a été corrigé le 29 sept. : *Update Symbol from Library*, puis case « Show pin names » par symbole.
- L'exclusion `endpoint_off_grid` (page racine) est héritée.

> Rappel (§7.6) : 0/0 ne prouve pas le fonctionnement.

### 10.1 Revalidation (R8) — conflit RL_x ✅ RÉSOLU (Chantier 1)

**Décision du 21 sept., appliquée le 30 sept. :** valeur **fixe** d'environ **1 A** sur les 5 canaux d'éclairage. RL_x ne sert qu'à faire détecter une remorque au véhicule. Repères : seuil GM 55 mA (bulletin NHTSA PIT5747B) ; feux DEL DOT de 50 à 200 mA. Les approches A et B sont abandonnées.

| Composant | Valeur | Statut |
|---|---|---|
| RL_x éclairage (R21, R64, R89, R122, R188) | 12 Ω / 25 W alu | ✅ |
| RSH_x éclairage (R22, R65, R90, R123, R189) | 0,2 Ω / 5 W (200 mV à 1 A) | ✅ |
| RL_AUX (R155) / RSH_AUX (R156) | 4,7 Ω / 50 W ; 0,05 Ω | ✅ (validation terrain) |
| Shunts TRAILER (R33, R67, R102, R135, R201) | 12 Ω / 1 W, écrêtés par MUR1520G (P ≤ ~0,1 W) | ✅ (validation au banc) |
| PTC TRAILER éclairage (F2-F5) | **MF-R700** (7 / 14 A) — charge max ~5 A (TAIL), ~4,2 A (LEFT/RIGHT) | ✅ (Chantier 6) |
| PTC BRAKE (F7) | **MF-R300** (3 / 6 A) | ✅ |
| PTC AUX (F6) | MF-R160 | ✅ (empreinte ⏳) |
| TVS, RSH_BUS (R221), seuils R216 / R210 | — | ⏳ |

### 10.2 État des chantiers

| Chantier | Statut |
|---|---|
| Canal_TAIL | ✅ Validé et corrigé (9 oct.) |
| Bloc_Masse, Alarmes/Buzzer, Jauge, coupure `V12_Rail` | ✅ |
| **Chantier 1 — RL_x VEHICLE** | ✅ 30 sept. |
| **Chantier 2 — mesure TRAILER (capteur PNP)** | ✅ **Appliqué le 9 oct.** sur les 5 canaux d'éclairage (Chantier 5) |
| **Chantier 3 — Voltmètre/ampèremètre** | ✅ 29 sept. (+ position 8 le 7 oct.) — reste : vérification à la réception du module, empreintes |
| **Chantier 4 — Canal AUX** | ✅ **7 oct.**, ERC 0/0 — reste : noms de rôle (§A.9), empreinte de F6, essais terrain et banc |
| **Chantier 5 — A1 / A2 / A6 sur 5 canaux** | ✅ **9 oct.**, ERC 0/0 |
| **Chantier 6 — Connecteurs, PTC, mesure de masse** | ✅ PTC, mesure de masse (12 oct.) ; ⏸ modèle de connecteur, P1, A9 |
| **Chantier 7 — Alimentation et thermique** | ✅ Schéma (A7, A8, BATT. EXT., mémoire du bus, pack) ; ⏳ thermique (attend le boîtier) |
| **Chantier 6 — Connecteurs J3/J4** | 🔴 En cours (10 oct.) |
| Empreintes manquantes (27) | 🔴 §A.8.4 → Chantier 8 |
| Renumérotation globale | 📌 Fin de projet |

### 10.3 Étage de mesure TRAILER ✅ RÉSOLU (v29)

- **AUX :** mesure de tension sur la charge à travers R168 (§7.5.3).
- **Canaux d'éclairage :** capteur PNP + MUR1520G pour « ouvert », diviseur ÷4,9 pour « court » (§7.2.3). Toutes les entrées des LM339 restent sous 3 V, quel que soit l'état de la batterie (9 à 13,8 V). Le problème de la v27 (écart ouvert/normal noyé dans la variation du rail) disparaît.
- **Reste :** validation au banc (DEL ~100 mA, incandescent ~2 A).

---

## Annexe A — Liste de composants (BOM)

> ✅ **VÉRIFIÉ KICAD (12 oct. 2026, commit `e2f94fe`)** — 31 fichiers, **652 composants uniques**.

### A.1 Alimentation et gestion d'énergie

| Composant | Réf. | Qté | Note |
|---|---|---|---|
| 18650 haut débit (≥ 20 A) | BT1-BT3 | 3 | Pack 3S1P |
| BMS 3S (≥ 30-40 A) | BMS3s1 | 1 | Critères §6.1 |
| **Fusible ~25 A + porte-fusible** | **F8** | 1 | Branche INT, hors PCB |
| **Shunt 0,05 Ω / 20 W TO-220** | **R289** | 1 | Shunt INT |
| **Bloc « BATT. EXT. »** | Q79-Q82, VR2, R290-R300, C10, LED2 | — | §6.1 |
| Module de charge 3S | — | 1 | **Pas encore au schéma** ; négatif sur P− |
| Connecteur (côté batterie externe) | P1 | 1 | ⚠️ Corrigé v28 |
| Connecteur pack interne JST-PH | — | 1 | **Absent du schéma** |
| Bornier | J2 | 1 | |
| Fusible 15 A | F1 | 1 | |
| LM74610 + IRLZ44N + 2,2 µF | U1, Q1, C1 | 1 chacun | |
| LM7805CT | U6 | 1 | |
| Découplage | C2-C6 | 5 | |
| SMCJ36CA | — | 2 | |
| SW2 (MODE), SW1 (SOURCE) | | 2 | |
| **Shunt 50 A / 75 mV** | **R242** | 1 | Boulonné, sans empreinte |
| **Traco TMR 2-1211WIN** | **PS1** | 1 | SIP-8, symbole/empreinte SnapEDA (`TMR_2-1211WIN`) |
| **10 µF / 50 V X7R AVX SR655C106KAR** | **C7** | 1 | |
| **10 µF / 25 V X7R TDK FG26X7R1E106KRT06** | **C8** | 1 | |
| **Raccord du module** | **J1** | 1 | Empreinte ⏳ |

### A.2 Connecteurs principaux et façade

| Composant | Réf. | Qté |
|---|---|---|
| Connecteur de façade (≥ 10 contacts, 15 A) — **modèle en suspens** | J3 / J4 | 2 |
| Câbles adaptateurs | — | 6 |
| **Module voltmètre/ampèremètre 0-100 V / 50 A** | (via J1) | 1 |
| **Lorlin CKS1503** + bouton axe 6 mm | **SW9** | 1 |
| **1 kΩ ½ W film métallique** | **R235-R241, R252, R308, R309** | **10** |
| Câbles adaptateurs SOOW 10 conducteurs + pince alligator | — | 6 |

### A.3 Décomptes par famille ✅ (12 oct., commit `e2f94fe`)

**652 composants uniques** (620 en v29).

| Famille | v29 | **v30** | Détail des écarts |
|---|---|---|---|
| LM339 | 14 | 14 | DIP-14 socket |
| IRLZ44N | 8 | **7** | Q64 → IRF4905 |
| IRF4905 | 6 | **7** | + Q64 |
| 2N7000 | 55 | **56** | + Q81 |
| 2N3904 | 3 | **4** | + Q80 |
| 2N3906 | 6 | **8** | + Q79, Q82 |
| 1N4148 | 153 | **153** | − D150 ; D149 réutilisée (hub) ; + D180 |
| Zener 1N5242B | 6 | **7** | + DZ7 |
| MUR1520G | 5 | 5 | |
| PC817 | 12 | 12 | |
| TVS P6KE39A | 6 | 6 | |
| PTC | 6 | **6** | MF-R700 ×4, MF-R300, MF-R160 |
| DEL | 6 RGB + 7 rouges + 2 ambre + bargraph | **6 RGB + 6 rouges + 3 ambre** + bargraph | − D151 ; + LED2 |
| Condensateur film 4,7 µF | 1 | **2** | + C10 |
| 0,1 µF SR205E104MARAP1 | 3 | **5** | + C11, C12 |
| Shunts TO-220 | 1 | **2** | + R289 |
| Fusibles | 1 | **2** | + F8 |
| 1 kΩ ½ W (SW9) | 8 | **10** | + R308, R309 |

### A.4 Bloc_Masse

Voir §6.5.1 (U15, R208-R211, R301-R307, C12). D149 (ancienne), D150, D151 retirées.

### A.5 Alarmes et buzzer

Voir §6.6.

**Protections thermiques :**
- BRAKE : TH1, R206, Q53, Q54, D147.
- AUX : voir §A.9.

### A.6 Jauge de batterie (10 segments) — ✅ FERMÉ

| Réf. | Composant | Valeur |
|---|---|---|
| U17 | LM3914N | DIP-18 + socket — `Package_DIP:DIP-18_W7.62mm_Socket` (10 oct.) |
| D25 | OSX10201-GYR1 | 2 R / 3 J / 5 V |
| R36 / R53 | Réseau de référence | 1,5 kΩ / 3,9 kΩ |
| R232 / R233 | Diviseur | 18 kΩ / 10 kΩ |
| VR1 | Bourns 3296W-1-102LF | 1 kΩ |
| R234 | Bus LED | 68 Ω / 5 W |
| C6 | Découplage | 0,1 µF |

**Topologie :**
- RHI (6) ↔ REF OUT (7) ;
- RLO (4) ↔ REF ADJ (8) ;
- MODE (9) ↔ V+ (3) ;
- SIG (5) alimentée par le diviseur depuis `Batt_Interne_Pos`, plage 9,0-12,6 V.

Le mapping LM3914 → bargraph est inchangé depuis la v27 : LED1 (pin 1) → cathode 11 (rouge), …, LED10 (pin 10) → cathode 20 (vert).

### A.7 Divers

| Composant | Qté |
|---|---|
| Sockets turned-pin | 14× DIP-14 + 1× DIP-18 |
| Plaque d'aluminium (charges + Q44) | 1 |
| Dissipateurs (RL BRAKE / R155 / Q44+R168) | à définir |
| PCB, boîtier IP54 | 1 |

### A.8 Bibliothèques KiCad et empreintes

**A.8.1-A.8.3** : inchangées depuis la v27. Une table de symboles et d'empreintes **Project Specific** a été ajoutée : `TMR_2-1211WIN` (SnapEDA) et `SR655C106KAR`.
- Les bibliothèques **PCM_*** (AKL, SL) restent une dépendance externe à réinstaller via le Plugin & Content Manager sur toute nouvelle machine.
- Après toute modification d'une table : **redémarrage complet de KiCad**.

#### A.8.4 Composants sans empreinte — **27** ✅ (13 oct., commit `f8a3617`)

| Groupe | Réf. | Nb | Piste |
|---|---|---|---|
| DEL RGB | D26, D49, D73, D97, D121, D145 | 6 | Selon le modèle de façade |
| Boutons | SW3-SW8 | 6 | Selon le bouton |
| Batterie | BT1-BT3, BMS3s1, P1 | 5 | Hors PCB / bornier |
| Interrupteurs | SW1, SW2, SW9 | 3 | Façade ; SW9 → connecteur de câblage |
| Connecteurs | J1, J3, J4 | 3 | J1 bornier 5 pos. ; J3/J4 hors PCB (modèle en suspens) |
| Divers | BZ1, F1, F8, R242 | 4 | F1/F8 hors PCB ; R242 boulonné |

**13 oct. :** TH1, SW10, SW11 → `TerminalBlock_Phoenix_PT-1,5-2-3.5-H_1x02_P3.50mm_Horizontal` ; F2-F5 → `PTC_Bourns_MF-R:PTC_Bourns_MF-R_P10.20mm_W24.2mm`, F6-F7 → `PTC_Bourns_MF-R:PTC_Bourns_MF-R_P5.10mm_W12.0mm` (bibliothèque projet `PTC_Bourns_MF-R.pretty`, créée d'après la fiche Bourns MF-R : KiCad n'a pas d'empreinte MF-R). **Les 27 restantes relèvent du Chantier 8** (façade et pièces de puissance hors carte).

**10 oct. :** les 14 LM339 ont reçu `Package_DIP:DIP-14_W7.62mm_Socket` et U17 `Package_DIP:DIP-18_W7.62mm_Socket` (bibliothèque standard `Package_DIP`).

#### A.8.5 Incidents et leçons

Les incidents de la v27 (D25, Q55-Q65, R216, `multiple_net_names`) restent valables. Leçons d'octobre 2026 :
- **Fenêtre « Change Symbols » :** elle mémorise le dernier identifiant. Vérifier l'option « selected » **et** le champ avant de valider. Elle ne remplace pas un champ par un champ vide.
- **Symboles AKL :** ils portent un **deuxième style De Morgan**. Un outil d'extraction qui ne filtre pas le style lit de fausses positions de pattes, ce qui a causé les fausses alertes A3 et A5.
- **Toute référence de pièce est vérifiée chez un distributeur** avant d'être retenue (FG28X7R1H106KRT06, inventée, n'existait pas).
- **Un bus ne relie rien** (A8) : l'ERC ne signale pas forcément un composant relié seulement par des bus.
- **Renommer une étiquette hiérarchique** dans une sous-feuille exige de **synchroniser les pattes** du rectangle sur la feuille parente (*Sync Sheet Pins*), puis de renommer les étiquettes locales correspondantes.
- **Envoi GitHub :** vérifier que **toutes** les feuilles modifiées sont téléversées ; un ERC à 0/0 sur l'ordinateur ne garantit pas que le dépôt est à jour.
- **Ancien fil oublié** lors d'une refonte (MassDetection, 12 oct.) : il fusionnait deux nets. Supprimer les anciens fils **avant** de recâbler.

#### A.8.6 Champs « Value » à uniformiser (convention du Chantier 1) 🔵

Convention : point décimal (pas de virgule, qui coupe l'export CSV), jamais de point initial seul, pas de préfixe « R ».

| Réf. | Valeur actuelle | Proposée |
|---|---|---|
| R155 | `4,7 Ohm - 50W` | `4.7 Ohm - 50W` |
| R156 | `.05R Shunt` | `0.05 Ohm Shunt` |
| R243 | `R100kΩ` | `100k` |
| R244 | `R330 Ohm` | `330ohm` |
| R33 | `.1 Ohm - 2W` | `0.1 Ohm - 2W` |
| R67, R102, R135, R201 | `.1 ohm` | `0.1 Ohm` |
| R221 | `.03 Ohm 20W` | `0.03 Ohm - 20W` |

Cosmétique : R168 utilise le symbole `PCM_SL_Resistors:4.7k`, et R34 aussi.

### A.9 Canal AUX — références et noms de rôle (champ « Etiquette ») ✅

| Réf. | Nom de rôle | Pièce | Empreinte |
|---|---|---|---|
| **VEHICLE** | | | |
| R155 | RL_Aux | 4,7 Ω / 50 W alu (TE HSA50 4R7 J ou Vishay RH050 4R70) | Bornier Phoenix PT-1,5-2-3.5 |
| R156 | RSH_Aux | 0,05 Ω | R_Axial_Power |
| Q43 | Q_Aux_Inject | IRLZ44N | TO-220 |
| SW10 | TH_Aux | KSD9700 NC | fils volants |
| R243 | R_Surch_Aux | 100 kΩ | DIN0207 |
| Q66 | T_Surch_Aux | 2N7000 | TO-92_Inline_Wide_SGD |
| Q67 | T_LED_Surch_Aux | 2N7000 | TO-92_Inline_Wide_SGD |
| R244 | R_LED_Surch_Aux | 330 Ω | DIN0207 |
| LED1 | LED_Surch_Aux | DEL ambre 1,8 mm | LED_D1.8mm |
| R158 | R_Etire_Aux | 330 kΩ | DIN0207 |
| C9 | C_Etire_Aux | WIMA MKS4C044704D00KSSD | C_Rect_L19.0mm_W7.0mm_P15.00mm_MKS4 |
| R245 | R_Ref_H_Aux | 10 kΩ | DIN0207 |
| R246 | R_Ref_B_Aux | 10 kΩ | DIN0207 |
| U11 (sect. 8/9/14) | U_Etire_Aux | LM339 | DIP-14 |
| **TRAILER** | | | |
| Q44 | Q_Sortie_Aux_Trail | IRF4905PBF | TO-220-3_Vertical |
| Q68 | T_Cmd_Aux_Trail | 2N7000 | TO-92_Inline_Wide_SGD |
| R247 | R_Grille_Aux_Trail | 1 kΩ | DIN0207 |
| R248 | R_GS_Aux_Trail | 10 kΩ | DIN0207 |
| DZ1 | DZ_GS_Aux_Trail | 1N5242B | DO-35 |
| SW11 | TH_Aux_Trail | KSD9700 NC (vérifier le calibre CC) | fils volants |
| R168 | R_Test_Aux | 12 Ω / 50 W alu, TE HSA5012RJ | Bornier Phoenix |
| F6 | PTC_Aux_Trail | Bourns MF-R160 | ⏳ |
| R249 | R_Div_H_Aux_Trail | 39 kΩ | DIN0207 |
| R250 | R_Div_B_Aux_Trail | 10 kΩ | DIN0207 |
| R251 | R_Ech_Court_Aux | 100 kΩ | DIN0207 |
| D169 | D_Btn_Court_Aux | 1N4148 | DO-35 |
| R252 | R_SW9_Trail_Aux | 1 kΩ ½ W | DIN0309 |

---

## Journal des versions

| Version | Date | Contenu principal |
|---|---|---|
| 22.0 | Août 2026 | Canal_TAIL fermé |
| 23.0 | 16 août 2026 | VEHICLE_LEFT validé |
| *(24, 25)* | — | N'existent pas |
| 26.0 | 18 sept. 2026 | Jauge fermée ; conflit §10.1 identifié |
| 27.0 | 20 sept. 2026 | Bloc_Masse, Alarmes/Buzzer, F5, bibliothèques, R9 proposée |
| 28.0 | 7 oct. 2026 | Chantier 1 : RL_x fixe ~1 A (§10.1 résolu). Chantier 3 : voltmètre/ampèremètre, R242/PS1/J1/SW9, BRAKE = 12 V aux., AUX = frein électrique, correction P1. Chantier 4 : canal AUX refait (§7.5). Anomalies A1/A2/A6 (§7.6). Chantier 2 invalidé. Empreintes manquantes : 74 → 49. ERC 0/0. Annexe A ré-extraite (560 composants) |
| **29.0** | **9 oct. 2026** | **Chantier 5 : A1/A2/A6 fermées sur les 5 canaux d'éclairage. Mesure TRAILER du Chantier 2 appliquée (capteur 2N3906, shunt 12 Ω / 1 W, MUR1520G, diviseur 39k/10k). Diode de blocage « court » dans Bloc_Test. Vérification exhaustive des 30 feuilles. ERC 0/0. 620 composants** |
| 29.0 (rév.) | 10 oct. 2026 | TVS P6KE39A (R5). Correction documentaire SW9 positions 6/7. Empreintes DIP-14/DIP-18 socket. ERC 0/0 |
| **30.0** | **12 oct. 2026** | **Chantier 7 : A7, A8 corrigées ; bloc « BATT. EXT. » ; mémoire du défaut de bus ; pack haut débit. Chantier 6 : GX16-10 abandonné (cahier des charges, modèle en suspens) ; PTC MF-R700/MF-R300 ; mesure de masse par pince + fenêtre ; D151 retirée ; buzzer sur masse ; SW9 positions 9-10 ; A9 relevée. ERC 0/0. 652 composants** |

---

## Points ouverts — récapitulatif

| # | Point | Section | Criticité |
|---|---|---|---|
| 1 | **Choisir le connecteur J3/J4** (≥ 10 contacts, 15 A) **et la prise P1** | §3.3.1 | 🔴 Bloque la façade |
| 2 | **A9** : vérification de cohérence inopérante (corriger ou supprimer) | §6.4 | 🔵 Décision |
| 3 | Thermique : dimensions du boîtier, choix A/B, thermostat de plaque pour TAIL/LEFT/RIGHT/BACKUP, cible ~70 W vs 131 W | §8 | ⏳ Attend le boîtier |
| 4 | 27 empreintes manquantes (façade et puissance hors carte → Chantier 8) | §A.8.4 | 🔴 Bloque le PCB |
| 4b | F2 (TAIL) : MF-R700 à 93 % du maintien à 50 °C (déclassement Bourns) → passer au **MF-R800** ? (même empreinte) | §10.1 | 🔵 Décision |
| 5 | Valider au banc : capteur PNP, R210 (masse), VR2 (BATT. EXT.), étireur AUX, R169/R170, R31/R32, polarité injection, surchauffe ↔ alarme | §10 | ⏳ |
| 6 | Validation terrain R155 = 4,7 Ω (GM, Ford, Ram, après-marché) ; essai Ford pour RL éclairage | §7.5.5 | ⏳ |
| 7 | Module voltmètre : fils, commun du shunt, **affichage négatif** | §7.4, §6.5.1 | ⏳ À la réception |
| 8 | Cellules du pack et BMS : références à vérifier chez un distributeur | §6.1 | ⏳ Achat |
| 9 | Chargeur et JST-PH du pack interne absents du schéma | §6.1 | ⏳ |
| 10 | Étireur pour les clignotants | §7.6 | 🔵 À étudier |
| 11 | Protection thermique BRAKE probablement superflue à 12 W | §7.2.5 | 🔵 Décision |
| 12 | R9 à arbitrer | R9 | 🔵 Décision |
| 13 | Noms de rôle → champ Etiquette ; uniformisation des valeurs (§A.8.6) | §A.9 | 📌 Utilisateur |
| 14 | Déclassement du câble SOOW 14/10 (canal ~5 A, masse sur 2 conducteurs) | §3.3.1 | ⏳ |
| 15 | Renumérotation globale (VR1, VR2 → R2xx/R3xx) | §10 pt 9 | 📌 Fin de projet |

---

*Ce document constitue une base de conception. Une revue par un ingénieur électronicien qualifié est recommandée avant toute fabrication d'un prototype destiné à être connecté à un véhicule réel.*
