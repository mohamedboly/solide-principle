# Open/Closed Principle (OCP) – Principe ouvert/fermé

## Définition

Le **Open/Closed Principle** stipule qu’un composant logiciel (classe, fonction, module) doit être :

* **ouvert à l’extension** ✅
* **fermé à la modification** ❌

Autrement dit : on doit pouvoir **ajouter un nouveau comportement sans modifier le code existant**.

---

## Comment savoir qu'on doit appliquer ce principe ?

Voici les **signes concrets** d'une violation du OCP ou d'une opportunité pour l'appliquer :

### 1. Tu modifies souvent une classe stable pour ajouter un nouveau comportement

> Exemple : tu ouvres souvent la classe `EarningsCalculator` pour ajouter un nouveau `case` ou un `if`.

```java
if (type.equals("PDF")) {
  exportToPdf(data);
} else if (type.equals("CSV")) {
  exportToCsv(data);
} else if (type.equals("EXCEL")) {
  exportToExcel(data);
}
```

Dès qu’un nouveau type arrive, tu dois modifier ce code → ⚠️ OCP violé

---

### 2. Présence de `switch`, `if/else`, `instanceof`, ou `type.equals(...)`

> Ces structures signalent que **le comportement dépend du type** au lieu d’être délégué à des objets spécialisés.

Pense à utiliser **des interfaces et le polymorphisme**.

---

### 3. La classe change à chaque ajout de règle métier

> Par exemple : une classe qui calcule des tarifs change chaque fois qu’on ajoute un nouveau mode de paiement.

Mauvaise séparation des responsabilités. L’OCP permettrait d’**ajouter une nouvelle classe au lieu de modifier l’ancienne**.

---

### 4. Les tests cassent à chaque nouveau cas

> Tu ajoutes une fonctionnalité → plusieurs tests cassent → tu dois patcher le code partout.

Tes comportements sont **trop couplés**, pas assez isolés. C’est un signe clair qu’il faut appliquer le OCP.

---

### 5. Pas d’abstraction, pas de polymorphisme

> Si tu n’utilises pas d’**interfaces**, **héritage**, ou **injection**, tu risques de créer des blocs rigides et peu extensibles.

L’abstraction te permet d’**étendre sans modifier**.

---

### La question magique à se poser :

> « Est-ce que j’ajoute souvent des `if`, `switch`, ou je modifie du code existant pour gérer un nouveau cas ? »

Si oui → **Tu as une bonne opportunité d’appliquer le OCP**.

---

## Exemple AVANT application de l’OCP

```java
public class EarningsCalculator {

    public double calculateEarnings(Video video) {
        return switch (video.getCategory()) {
            case EDUCATIONAL -> calculateEducationalEarnings(video);
            case GAMING -> calculateGamingEarnings(video);
            case ENTERTAINMENT -> calculateEntertainmentEarnings(video);
        };
    }

    private double calculateEducationalEarnings(Video video) {
        return video.getLikes() * 0.013 + video.getViews() * 0.0013;
    }

    private double calculateGamingEarnings(Video video) {
        return video.getLikes() * 0.012 + video.getViews() * 0.0012;
    }

    private double calculateEntertainmentEarnings(Video video) {
        return video.getLikes() * 0.011 + video.getViews() * 0.0011;
    }
}
```

### Problème

Chaque fois qu’une nouvelle catégorie est ajoutée, on doit **modifier la méthode `calculateEarnings()`**.

Cela viole le principe ouvert/fermé car la classe n’est **pas extensible sans modification**.

---

## Exemple APRÈS application de l’OCP

### Interface `EarningsCalculator.java`

```java
public interface EarningsCalculator {
    double calculateEarnings(Video video);
}
```

### Implémentations par catégorie

```java
public class EducationalEarningsCalculator implements EarningsCalculator {
    public double calculateEarnings(Video video) {
        return video.getLikes() * 0.013 + video.getViews() * 0.0013;
    }
}

public class GamingEarningsCalculator implements EarningsCalculator {
    public double calculateEarnings(Video video) {
        return video.getLikes() * 0.012 + video.getViews() * 0.0012;
    }
}

public class EntertainmentEarningsCalculator implements EarningsCalculator {
    public double calculateEarnings(Video video) {
        return video.getLikes() * 0.011 + video.getViews() * 0.0011;
    }
}
```

### Utilisation (exemple d'injection par type ou contexte)

```java
EarningsCalculator calculator = new GamingEarningsCalculator();
double revenue = calculator.calculateEarnings(video);
```

---

## Bénéfices obtenus

* Plus besoin de **modifier `EarningsCalculator`** pour ajouter une nouvelle catégorie
* On respecte pleinement le **Open/Closed Principle**
* Chaque comportement est **isolé** et **facilement testable**
* **Facile à étendre** : il suffit d’ajouter une nouvelle classe implémentant l’interface


