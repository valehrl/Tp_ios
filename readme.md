# ehrle_fatmi_ios

Application SwiftUI multiplateforme qui affiche des citations de trois sources : **Kanye West**, **Chuck Norris** et **Breaking Bad**. Le projet contient une app iOS, une app watchOS autonome et une extension WidgetKit.

## Fonctionnalités

- Trois pages de citations, sélectionnables depuis le menu en haut à droite de l'app iOS.
- Chargement automatique d'une citation à l'ouverture d'une page, et rechargement manuel en touchant l'image du personnage.
- Widget iOS configurable (Kanye, Chuck Norris ou Breaking Bad), avec bouton pour changer de personnage et lien profond vers la page correspondante de l'app.
- App Apple Watch avec navigation verticale et rechargement par toucher.
- Deep links : `ehrlefatmi://kanye`, `ehrlefatmi://chuck`, `ehrlefatmi://breakingBad`.

## Prérequis

- macOS compatible avec **Xcode 26.3** ou plus récent (Xcode 26.3 a été utilisé pour les cibles Watch et Widget).
- Simulateur iOS **18.5+** (par exemple iPhone 17) ou appareil iOS avec un compte Apple Developer pour la signature automatique.
- Simulateur watchOS **11.5+** appairé si vous voulez lancer la cible Watch.
- Connexion Internet pour afficher de vraies citations. Les tests unitaires, eux, n'en ont pas besoin.
- Aucun gestionnaire de dépendances n'est utilisé : pas de `pod install` ni de `swift package resolve` à exécuter.

## Installation et lancement

```bash
git clone <url-du-depot>
cd ehrle_fatmi_ios
open ehrle_fatmi_proj.xcodeproj
```

Dans Xcode :

1. Sélectionner le schéma **ehrle_fatmi_proj** et un simulateur iPhone.
2. Lancer avec `Cmd+R` (ou le bouton ▶︎).

L'app s'ouvre directement sur la page **Kanye**.

### Lancer l'app Watch

Sélectionner le schéma **ehrle_fatmi_watch** et une montre simulée appairée, puis lancer avec `Cmd+R`. L'app Watch fonctionne de manière autonome (elle n'a pas besoin que l'app iPhone soit ouverte).

### Ajouter le widget

L'extension `ehrle_fatmi_widget` est intégrée à l'app iOS :

1. Lancer l'app iOS sur le simulateur ou l'appareil.
2. Revenir sur l'écran d'accueil, entrer en mode édition et ajouter le widget **Citation**.
3. Toucher le widget pour « Modifier le widget » et choisir la source des citations.
4. Formats disponibles : petit, moyen et grand.

### Alternative en ligne de commande

```bash
xcodebuild -project ehrle_fatmi_proj.xcodeproj \
  -scheme ehrle_fatmi_proj \
  -configuration Debug \
  -sdk iphonesimulator \
  -destination 'platform=iOS Simulator,name=iPhone 17' \
  build
```

Les schémas ne sont pas versionnés dans le dépôt : si un schéma n'apparaît pas en ligne de commande, ouvrir le projet une fois dans Xcode pour qu'il soit généré automatiquement.

## Utilisation

### App iOS

- Le menu en haut à droite (`Kanye`, `Chuck`, `Breaking Bad`) change de page.
- Toucher l'image du personnage recharge une citation ; un indicateur de chargement s'affiche pendant l'appel réseau.
- Si une API est indisponible, la page affiche `Erreur: …` et permet de réessayer.
- Pour ouvrir directement une page depuis le terminal avec un simulateur démarré :

```bash
xcrun simctl openurl booted ehrlefatmi://kanye
xcrun simctl openurl booted ehrlefatmi://chuck
xcrun simctl openurl booted ehrlefatmi://breakingBad
```

Les hôtes sont sensibles à la casse : `ehrlefatmi://breakingbad` n'est pas reconnu.

### Widget

- Toucher la citation (ou l'image sur les formats moyen et grand) recharge une citation.
- Toucher le bouton avec l'icône de personnage fait défiler la source : Kanye → Chuck Norris → Breaking Bad → Kanye.
- Le widget demande un rafraîchissement au système environ toutes les 30 minutes (fréquence indicative, décidée par iOS).
- Toucher le widget ouvre l'app iOS sur la page du personnage affiché.

### App Watch

- Faire défiler verticalement avec la Digital Crown pour passer de Kanye à Chuck Norris puis Breaking Bad.
- Toucher l'image du personnage pour charger une nouvelle citation.

## Tests

Dans Xcode, sélectionner le schéma **ehrle_fatmi_proj**, un simulateur iPhone, puis lancer `Cmd+U`.

En ligne de commande :

```bash
# Tous les tests (unitaires + interface)
xcodebuild test -project ehrle_fatmi_proj.xcodeproj \
  -scheme ehrle_fatmi_proj \
  -configuration Debug \
  -destination 'platform=iOS Simulator,name=iPhone 17'

# Uniquement les tests unitaires
xcodebuild test -project ehrle_fatmi_proj.xcodeproj \
  -scheme ehrle_fatmi_proj \
  -configuration Debug \
  -destination 'platform=iOS Simulator,name=iPhone 17' \
  -only-testing:ehrle_fatmi_projTests

# Uniquement les tests d'interface
xcodebuild test -project ehrle_fatmi_proj.xcodeproj \
  -scheme ehrle_fatmi_proj \
  -configuration Debug \
  -destination 'platform=iOS Simulator,name=iPhone 17' \
  -only-testing:ehrle_fatmi_projUITests
```

Le nom du simulateur peut être adapté avec `xcrun simctl list devices available`. Pour conserver les résultats, ajouter `-resultBundlePath ./Tests.xcresult` (les fichiers `.xcresult` sont déjà ignorés par Git).

### Suites de tests

| Cible | Suite | Ce qui est vérifié |
|---|---|---|
| `ehrle_fatmi_projTests` | `ModelDecodingTests` | Décodage/encodage JSON des modèles, notamment le `icon_url` en snake_case de Chuck Norris |
| `ehrle_fatmi_projTests` | `ServiceTests` | URLs appelées, codes HTTP, JSON invalide et erreurs réseau, via un `URLProtocol` factice |
| `ehrle_fatmi_projTests` | `AppPageTests` | Ouverture des pages via les deep links du widget |
| `ehrle_fatmi_projTests` | `WidgetTests` | Cycle des personnages, état mémorisé par widget et intents des boutons |
| `ehrle_fatmi_projTests` | `PersistenceTests` | Magasin Core Data en mémoire (création, lecture, suppression) |
| `ehrle_fatmi_projUITests` | `NavigationUITests` | Navigation réelle entre les trois pages via le menu de l'app |

Les tests unitaires simulent les appels réseau : ils fonctionnent hors ligne et ne dépendent pas de la disponibilité des API. Les tests d'interface lancent l'app dans le simulateur et sont nettement plus lents (le test de performance mesure plusieurs lancements). Un échec de lancement du runner sur un clone de simulateur peut être suivi d'un nouvel essai réussi.

## Organisation du projet

```text
ehrle_fatmi_proj/         App iOS : vues, modèles, services réseau, Core Data
ehrle_fatmi_watch/        App watchOS : pages verticales de citations
ehrle_fatmi_widget/       Extension WidgetKit : configuration, timeline et intents
ehrle_fatmi_projTests/    Tests unitaires (XCTest)
ehrle_fatmi_projUITests/  Tests d'interface (XCTest)
ehrle_fatmi_proj.xcodeproj/ Projet Xcode et ses 5 cibles
```

Les modèles et services de `ehrle_fatmi_proj/` sont partagés avec les cibles Watch et Widget. La pile Core Data est présente dans le projet et testée, mais elle n'est pas encore utilisée par les écrans de l'application.

## Dépannage

- **L'app affiche `Erreur: …`** : l'API correspondante est momentanément indisponible ou le réseau est coupé ; toucher à nouveau l'image pour réessayer.
- **Le deep link ne change pas de page** : vérifier la casse exacte de l'hôte (`breakingBad`).
- **Le widget n'apparaît pas** : lancer l'app iOS une fois, puis l'ajouter depuis l'écran d'accueil en mode édition.
- **Aucun simulateur ne correspond à la commande** : remplacer `iPhone 17` par le nom d'un simulateur installé (voir `xcrun simctl list devices available`).
- **Échec de signature sur appareil physique** : sélectionner une équipe de développement dans les réglages de signature du target concerné dans Xcode.
