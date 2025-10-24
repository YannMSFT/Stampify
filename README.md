# Stampify - Standalone Version 📄

Application web 100% client-side pour ajouter des filigranes personnalisables aux fichiers PDF.

## ✨ Version Standalone - Aucun serveur requis !

Cette version fonctionne **entièrement dans votre navigateur**. Aucune donnée n'est envoyée à un serveur.

## 🚀 Utilisation

### Option 1 : Ouvrir directement le fichier HTML
1. Double-cliquez sur `stampify-standalone.html`
2. L'application s'ouvrira dans votre navigateur par défaut
3. C'est tout ! Commencez à ajouter des filigranes

### Option 2 : Depuis la ligne de commande
```bash
open stampify-standalone.html
```

## 📋 Fonctionnalités

- ✅ Upload de fichiers PDF (max 50MB, 500 pages)
- ✅ Configuration complète du filigrane :
  - Texte personnalisable
  - Taille de police ajustable (10-100)
  - Rotation (-180° à 180°)
  - Opacité (10% à 100%)
  - Choix de la couleur
  - Option de répétition sur toute la page
- ✅ Aperçu en temps réel du filigrane
- ✅ Validation automatique des fichiers
- ✅ Traitement 100% local (aucune donnée envoyée)
- ✅ Téléchargement du PDF traité
- ✅ Interface minimaliste et intuitive

## 🔒 Sécurité et Confidentialité

- **100% Client-Side** : Tout le traitement se fait dans votre navigateur
- **Aucune connexion réseau** : Vos PDF ne quittent jamais votre ordinateur
- **Pas de serveur** : Aucune installation ou configuration requise
- **Open Source** : Code transparent et vérifiable

## 💻 Technologies utilisées

- **HTML5** : Structure de la page
- **CSS3** : Design minimaliste et responsive
- **JavaScript (Vanilla)** : Logique de l'application
- **PDF-lib.js** : Manipulation des PDF côté client

## 📦 Fichiers

```
stampify-standalone.html    # Fichier unique autonome (tout le code est inclus)
```

## 🌐 Compatibilité

Fonctionne sur tous les navigateurs modernes :
- ✅ Chrome / Edge
- ✅ Firefox
- ✅ Safari
- ✅ Opera

## 🎯 Avantages de la version standalone

1. **Aucune installation** : Pas de Python, Flask, ou dépendances
2. **Portable** : Copiez le fichier HTML sur n'importe quel ordinateur
3. **Sécurisé** : Vos documents restent privés
4. **Rapide** : Pas de communication réseau
5. **Simple** : Double-clic et c'est parti !

## 📝 Utilisation

1. **Ouvrir** le fichier `stampify-standalone.html`
2. **Sélectionner** votre fichier PDF
3. **Configurer** le filigrane (texte, taille, rotation, etc.)
4. **Prévisualiser** le rendu en temps réel
5. **Appliquer** le filigrane
6. **Télécharger** votre PDF modifié

## 🆚 Comparaison avec la version Flask

| Caractéristique | Version Flask | Version Standalone |
|----------------|---------------|-------------------|
| Installation | Python + dépendances | Aucune |
| Serveur requis | Oui (local) | Non |
| Sécurité | Données locales | Données locales |
| Portabilité | Moyenne | Excellente |
| Performance | Rapide | Rapide |
| Simplicité | Moyenne | Très simple |

## 🎨 Interface

Interface minimaliste avec :
- Zone de drag & drop pour l'upload
- Curseurs intuitifs pour les réglages
- Aperçu en temps réel
- Messages d'erreur clairs
- Design responsive (mobile-friendly)

## 📄 Limites

- Taille maximale : 50 MB
- Nombre de pages : 500 maximum
- Format : PDF uniquement

## 🔧 Personnalisation

Le fichier HTML contient tout le code (HTML, CSS, JavaScript). Vous pouvez facilement :
- Modifier les couleurs dans la section `:root`
- Ajuster les limites (taille, pages)
- Personnaliser le texte et les labels
- Ajouter des fonctionnalités

## 📞 Support

Pour toute question ou suggestion, créez une issue sur le repository.

---

**Stampify Standalone** - La solution la plus simple pour ajouter des filigranes à vos PDF ! 🎉
