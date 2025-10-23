# Workflow GitHub Actions - Validation du Score de Mutation# Documentation : Workflow GitHub Actions - Validation du Score de Mutation# Documentation : Workflow GitHub Actions - Validation du Score de Mutation



## 📋 Objectif du Devoir



Modifier le workflow GitHub Actions de GraphHopper pour que **le processus de build échoue automatiquement si le score de mutation baisse après un commit**.## 📋 Objectif## 📋 Objectif



---



## 🏗️ Choix de Conception et Justifications**Modifier le workflow GitHub Actions pour que le build échoue si le score de mutation baisse.****Modifier le workflow GitHub Actions de GraphHopper pour que le build échoue automatiquement si le score de mutation baisse après un commit.**



### 1. Architecture : Intégration dans le workflow existant



**Choix retenu :** Modification de `.github/workflows/build.yml`------



**Justifications :**

- Exécution automatique à chaque push/PR

- Feedback immédiat aux développeurs## 🏗️ Choix de Conception## 🏗️ Choix de Conception

- Pas de duplication de configuration

- Simplicité de maintenance



**Alternative rejetée :** Workflow séparé### 1. Intégration dans workflow existant### 1. Architecture : Intégration dans workflow existant

- Risque d'oubli d'exécution

- Configuration dupliquée- **Fichier** : `.github/workflows/build.yml`**✅ Choix :** Ajout dans `.github/workflows/build.yml`  



### 2. Stockage du score baseline : Fichier versionné dans Git- **Raison** : Feedback immédiat, pas de duplication**Raison :** Feedback immédiat, pas de duplication, exécution automatique



**Choix retenu :** Fichier `baseline_mutation_score.txt` versionné



**Justifications :**### 2. Fichier baseline versionné### 2. Stockage du baseline : Fichier versionné Git

- Traçabilité complète dans l'historique Git

- Simple et transparent- **Fichier** : `baseline_mutation_score.txt` **✅ Choix :** `baseline_mutation_score.txt` dans le repo  

- Aucune dépendance externe (API, base de données)

- Fonctionne en mode offline- **Raison** : Traçabilité Git, simplicité**Raison :** Traçabilité, simplicité, aucune dépendance externe



**Alternative rejetée :** Cache GitHub Actions

- Peut être supprimé automatiquement

- Pas de traçabilité historique### 3. Tolérance zéro### 3. Règle de validation : Tolérance zéro



### 3. Règle de validation : Tolérance zéro- **Règle** : `if (score < baseline) → exit 1`**✅ Choix :** `if (score_actuel < baseline) → exit 1`  



**Choix retenu :** Build échoue si `score_actuel < baseline`- **Raison** : Empêche toute régression**Raison :** Prévient toute régression de qualité



**Justifications :**

- Empêche toute régression de qualité

- Force l'amélioration continue des tests### 4. Parser XML Pitest### 4. Extraction du score : Parser XML Pitest

- Évite la "dégradation progressive"

- **Méthode** : `grep 'mutationScore' mutations.xml`**✅ Choix :** `grep -o 'mutationScore="[0-9]*"' mutations.xml`  

**Alternative rejetée :** Tolérance de ±5%

- Permettrait une dégradation graduelle- **Raison** : Format stable et fiable**Raison :** Format stable, simple, fiable

- Moins strict sur la qualité



### 4. Extraction du score : Parser le XML de Pitest

------

**Choix retenu :** `grep -o 'mutationScore="[0-9]*"' mutations.xml`



**Justifications :**

- Format XML stable et standardisé par Pitest## 🔧 Implémentation## 🔧 Implémentation

- Simple et fiable

- Pas de dépendance supplémentaire



**Alternative rejetée :** Parser le rapport HTML### Étapes du Workflow### Workflow GitHub Actions (`.github/workflows/build.yml`)

- Format peut changer entre versions

- Plus complexe et fragile



---**1. Exécuter Pitest****Étape 1 : Exécution Pitest**



## 🔧 Implémentation Technique```yaml```yaml



### Structure du Workflow- name: Run Mutation Tests- name: Run Mutation Tests



Le workflow suit ces 4 étapes principales :  run: |  run: |



#### Étape 1 : Exécution des tests de mutation    cd core    cd core

```yaml

- name: Run Mutation Tests (Current)    mvn org.pitest:pitest-maven:mutationCoverage    mvn org.pitest:pitest-maven:mutationCoverage

  if: matrix.java-version == '24'

  run: |``````

    cd core

    mvn org.pitest:pitest-maven:mutationCoverage

    cd ..

```**2. Extraire le score****Étape 2 : Extraction du score**



#### Étape 2 : Extraction du score actuel```yaml```yaml

```yaml

- name: Extract Current Mutation Score- name: Extract Current Score- name: Extract Current Mutation Score

  if: matrix.java-version == '24'

  run: |  run: |  run: |

    MUTATIONS_FILE=$(find ./core -name "mutations.xml" -type f | head -1)

    CURRENT_SCORE=$(grep -o 'mutationScore="[0-9]*"' "$MUTATIONS_FILE" | grep -o '[0-9]*')    CURRENT_SCORE=$(grep -o 'mutationScore="[0-9]*"' mutations.xml)    CURRENT_SCORE=$(grep -o 'mutationScore="[0-9]*"' mutations.xml | grep -o '[0-9]*')

    echo "CURRENT_MUTATION_SCORE=$CURRENT_SCORE" >> $GITHUB_ENV

```    echo "CURRENT_MUTATION_SCORE=$CURRENT_SCORE" >> $GITHUB_ENV    echo "CURRENT_MUTATION_SCORE=$CURRENT_SCORE" >> $GITHUB_ENV



#### Étape 3 : Lecture du score baseline``````

```yaml

- name: Get Baseline Mutation Score

  if: matrix.java-version == '24'

  run: |**3. Lire le baseline****Étape 3 : Lecture baseline**

    BASELINE_SCORE=$(cat baseline_mutation_score.txt | grep -v '#' | tail -1)

    echo "BASELINE_MUTATION_SCORE=$BASELINE_SCORE" >> $GITHUB_ENV```yaml```yaml

```

- name: Get Baseline Score- name: Get Baseline Mutation Score

#### Étape 4 : Comparaison et validation

```yaml  run: |  run: |

- name: Compare Mutation Scores

  if: matrix.java-version == '24'    BASELINE_SCORE=$(cat baseline_mutation_score.txt | tail -1)    BASELINE_SCORE=$(cat baseline_mutation_score.txt | grep -v '#' | tail -1)

  run: |

    if [ $CURRENT_MUTATION_SCORE -lt $BASELINE_MUTATION_SCORE ]; then    echo "BASELINE_MUTATION_SCORE=$BASELINE_SCORE" >> $GITHUB_ENV    echo "BASELINE_MUTATION_SCORE=$BASELINE_SCORE" >> $GITHUB_ENV

      echo "❌ MUTATION SCORE REGRESSION DETECTED!"

      exit 1  # ← Le build échoue ici``````

    elif [ $CURRENT_MUTATION_SCORE -gt $BASELINE_MUTATION_SCORE ]; then

      echo "✅ MUTATION SCORE IMPROVED!"

      echo "$CURRENT_MUTATION_SCORE" > baseline_mutation_score.txt

    else**4. Comparer et valider****Étape 4 : Comparaison et validation**

      echo "✅ MUTATION SCORE MAINTAINED"

    fi```yaml```yaml

```

- name: Compare Scores- name: Compare Mutation Scores

### Configuration Pitest

  run: |  run: |

**Fichier :** `core/pom.xml`

    if [ $CURRENT_MUTATION_SCORE -lt $BASELINE_MUTATION_SCORE ]; then    if [ $CURRENT_MUTATION_SCORE -lt $BASELINE_MUTATION_SCORE ]; then

```xml

<plugin>      echo "❌ REGRESSION!"      echo "❌ MUTATION SCORE REGRESSION DETECTED!"

    <groupId>org.pitest</groupId>

    <artifactId>pitest-maven</artifactId>      exit 1  # ← Build échoue      exit 1  # ← Le build échoue ici

    <version>1.15.8</version>

    <configuration>    fi    fi

        <targetClasses>

            <param>com.graphhopper.util.shapes.Circle</param>``````

        </targetClasses>

        <targetTests>

            <param>com.graphhopper.util.shapes.CircleTest</param>

        </targetTests>### Configuration### Configuration Pitest (`core/pom.xml`)

        <mutators>

            <mutator>STRONGER</mutator>

        </mutators>

        <features>**Pitest (`core/pom.xml`):**```xml

            <feature>+CLASSLIMIT(limit[20])</feature>

        </features>```xml<plugin>

        <outputFormats>

            <outputFormat>HTML</outputFormat><plugin>    <groupId>org.pitest</groupId>

            <outputFormat>XML</outputFormat>

        </outputFormats>    <groupId>org.pitest</groupId>    <artifactId>pitest-maven</artifactId>

    </configuration>

</plugin>    <artifactId>pitest-maven</artifactId>    <version>1.15.8</version>

```

    <version>1.15.8</version>    <configuration>

### Fichier Baseline

    <configuration>        <targetClasses>

**Fichier :** `baseline_mutation_score.txt`

        <targetClasses>            <param>com.graphhopper.util.shapes.Circle</param>

```

# Baseline Mutation Score            <param>com.graphhopper.util.shapes.Circle</param>        </targetClasses>

# Score de référence pour les tests de mutation

# Le workflow GitHub Actions échoue si le score descend en dessous de cette valeur        </targetClasses>        <targetTests>

60

```    </configuration>            <param>com.graphhopper.util.shapes.CircleTest</param>



---</plugin>        </targetTests>



## ✅ Validation de l'Implémentation```        <features>



### Scénarios de Test            <feature>+CLASSLIMIT(limit[20])</feature>



| Baseline | Score Actuel | Résultat | Action Effectuée |**Baseline (`baseline_mutation_score.txt`):**        </features>

|----------|-------------|----------|------------------|

| 60% | 60% | ✅ Build SUCCESS | Aucune |```    </configuration>

| 60% | 70% | ✅ Build SUCCESS | Baseline mis à jour → 70% |

| 60% | 55% | ❌ Build FAILURE | Message d'erreur affiché |60</plugin>



### Méthode de Validation``````



**1. Test sur GitHub Actions**



Le workflow a été testé en pushant les modifications sur GitHub :---### Fichier Baseline (`baseline_mutation_score.txt`)



```bash

git add .

git commit -m "feat: add mutation score validation to CI/CD"## ✅ Validation```

git push

```60



**2. Vérification des résultats**### Scénarios de Test```



- Le workflow s'exécute automatiquement sur Java 24

- Les tests de mutation sont lancés sur le module `core`

- Le score est extrait du fichier `mutations.xml`| Baseline | Actuel | Résultat |---

- La comparaison est effectuée avec le baseline

- Le build échoue si régression détectée|----------|--------|----------|



**3. Artifacts générés**| 60% | 60% | ✅ Build OK |## ✅ Validation



Le workflow upload les artifacts suivants :| 60% | 70% | ✅ Build OK + Baseline → 70% |

- Rapports Pitest HTML/XML

- Fichier `current_mutation_score.txt`| 60% | 55% | ❌ Build FAIL |### Scénarios de Test

- Fichier `baseline_mutation_score.txt`



---

### Test Local| Scénario | Baseline | Actuel | Résultat | Action |

## 📊 Utilisation du Workflow

|----------|----------|--------|----------|--------|

### Processus Standard

```bash| **Maintenu** | 60% | 60% | ✅ Build OK | Aucune |

1. **Développeur fait des modifications**

2. **Commit et push sur GitHub**# Windows| **Amélioré** | 60% | 70% | ✅ Build OK | Baseline → 70% |

3. **Workflow s'exécute automatiquement**

4. **Pitest analyse les mutations**.\test-workflow.bat| **Régression** | 60% | 55% | ❌ Build FAIL | Message d'erreur |

5. **Score comparé au baseline**

6. **Build échoue si régression**



### Messages Affichés# Résultat attendu :### Test Local



**En cas de succès :**# - Pitest s'exécute

```

✅ MUTATION SCORE MAINTAINED# - Rapport généré dans core/target/pit-reports/**Commande :**

   Score remained at 60% - good!

```# - Browser s'ouvre avec le score```bash



**En cas d'amélioration :**```.\test-workflow.bat   # Windows

```

✅ MUTATION SCORE IMPROVED!./test-workflow.sh    # Linux/Mac

   Score increased by 10% - excellent work!

```### Test CI/CD```



**En cas de régression :**

```

❌ MUTATION SCORE REGRESSION DETECTED!1. **Push code** → Workflow démarre**Résultat attendu :**

   Current score (55%) is lower than baseline (60%)

   2. **Pitest exécuté** automatiquement1. Compilation OK

🔧 To fix this issue:

   1. Add more comprehensive tests for your new code3. **Score comparé** au baseline2. Pitest s'exécute

   2. Ensure your tests kill the mutants introduced by Pitest

   3. Review the Pitest HTML report for specific recommendations4. **Build échoue** si régression3. Rapport généré dans `core/target/pit-reports/index.html`

```



---

---### Test CI/CD

## 📝 Fichiers Modifiés



### Fichiers principaux

## 📊 Utilisation**Déclenchement :**

1. **`.github/workflows/build.yml`**

   - Ajout des étapes de mutation testing- Chaque `git push`

   - Extraction et comparaison du score

   - Gestion des artifacts**1. Tester localement avant commit :**- Chaque Pull Request



2. **`baseline_mutation_score.txt`**```bash

   - Score de référence : 60%

   - Versionné dans Git.\test-workflow.bat**Validation :**



3. **`core/pom.xml`**```1. Workflow s'exécute sur Java 24

   - Configuration Pitest

   - Classe cible : `Circle`2. Score extrait automatiquement

   - Tests : `CircleTest`

**2. Pousser sur GitHub :**3. Comparaison avec baseline

---

```bash4. Build échoue si régression

## 🎯 Conclusion

git push

### Objectif Atteint

```---

✅ Le workflow GitHub Actions échoue maintenant automatiquement si le score de mutation baisse après un commit.



### Avantages de la Solution

**3. Vérifier le résultat :**## 📊 Utilisation

1. **Automatique** : Pas d'intervention manuelle requise

2. **Transparent** : Baseline versionné dans Git- GitHub Actions tab

3. **Fiable** : Basé sur le format XML stable de Pitest

4. **Actionnable** : Messages d'erreur clairs et utiles- ✅ = Score OK## 🚀 Utilisation

5. **Maintenable** : Configuration simple et documentée

- ❌ = Régression détectée

### Points Clés

### 1. Avant de Commit (Recommandé)

- Score baseline actuel : **60%**

- Classe testée : `com.graphhopper.util.shapes.Circle`---```bash

- Workflow exécuté sur : **Java 24 (GitHub Actions)**

- Tolérance : **Zéro** (aucune régression acceptée)# Linux/Mac


## 📝 Conclusion./check-mutation-score.sh



**✅ Objectif atteint :** Build échoue si score baisse# Windows

check-mutation-score.bat

**Fichiers modifiés :**```

- `.github/workflows/build.yml` - Validation mutation

- `baseline_mutation_score.txt` - Score référence (60%)### 2. Exécution Manuelle de Pitest

- `core/pom.xml` - Config Pitest```bash

mvn org.pitest:pitest-maven:mutationCoverage

**Test rapide :**```

```bash

.\test-workflow.bat### 3. Visualisation des Résultats

```Ouvrir `target/pit-reports/index.html` dans votre navigateur.


## 📊 Workflow GitHub Actions

### Déclenchement
- ✅ Push sur n'importe quelle branche
- ✅ Pull Request

### Étapes
1. **Build & Test** - Tests unitaires classiques
2. **Mutation Testing** - Exécution de Pitest
3. **Score Extraction** - Analyse du rapport XML
4. **Comparaison** - Vérification vs baseline
5. **Validation** - ❌ Échec si régression détectée

### Résultats
- **✅ Score maintenu/amélioré** → Build réussi
- **❌ Score en régression** → Build échoué
- **📊 Rapport détaillé** → Artifacts téléchargeables
- **💬 Commentaire PR** → Score affiché automatiquement

## 🔍 Interprétation des Résultats

### Score de Mutation
- **70-80%** : Acceptable
- **80-90%** : Bon
- **90%+** : Excellent

### Types de Mutants
- **Tués** ✅ : Tests détectent la mutation
- **Survivants** ❌ : Tests n'échouent pas malgré la mutation
- **Non couverts** ⚠️ : Code non testé

## 🛠️ Résolution des Problèmes

### Score en Baisse
1. **Identifier les mutants survivants** dans le rapport HTML
2. **Ajouter des tests** pour tuer ces mutants
3. **Vérifier les assertions** dans les tests existants
4. **Tester les cas limites** et conditions de branchement

### Exemples de Tests Manquants
```java
// Mutant survivant : condition != devient ==
if (value != null) { ... }

// Test à ajouter :
@Test
public void testNullValue() {
    assertThrows(Exception.class, () -> method(null));
}
```

## 📈 Amélioration Continue

### Objectifs
- [ ] Augmenter progressivement le score de base
- [ ] Ajouter plus de classes à l'analyse
- [ ] Intégrer d'autres métriques de qualité

### Bonnes Pratiques
1. **Tests préventifs** : Écrire les tests avant de modifier le code
2. **Revue régulière** : Analyser les rapports Pitest
3. **Tests de régression** : Ajouter des tests pour chaque bug corrigé

## 🎓 Apprentissage

Cette configuration enseigne :
- **Qualité des tests** : Au-delà de la couverture de code
- **CI/CD avancé** : Métriques de qualité dans les pipelines
- **Tests de mutation** : Détection de tests insuffisants
- **DevOps** : Automatisation de la qualité

## 📚 Ressources

- [Documentation Pitest](http://pitest.org/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [Mutation Testing](https://en.wikipedia.org/wiki/Mutation_testing)

---

**💡 Conseil** : Utilisez `check-mutation-score.sh` avant chaque commit pour éviter les échecs CI !