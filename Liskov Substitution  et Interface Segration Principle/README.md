# Liskov Substitution Principle (LSP) – Principe de substitution de Liskov

## 1. Définition

Le **Liskov Substitution Principle** stipule que :

> « Une sous-classe doit pouvoir être utilisée à la place de sa super-classe **sans altérer le comportement du programme**. »

Formulé par Barbara Liskov, ce principe impose qu’un objet d’une classe fille **puisse être substitué partout où un objet de la classe mère est attendu**, **sans effets de bord**.

---

## Remarque importante sur l’héritage (point clé du LSP)

Le LSP **nous met en garde contre un usage naïf de l’héritage**.

> ❌ On ne doit pas hériter **uniquement** parce qu’on a des **attributs communs**
> ✅ On doit aussi s’assurer que **le comportement du parent est souhaité et valable pour les enfants**

### À se demander avant d’hériter :

* Est-ce que la sous-classe **peut vraiment se comporter comme la classe mère** ?
* Est-ce qu’elle **accepte toutes les méthodes** de la super-classe sans les casser ou lever d’exception ?

### Exemple classique d’erreur :

```java
class Bird {
    void fly() { ... }
}

class Ostrich extends Bird {
    @Override
    void fly() {
        throw new UnsupportedOperationException();
    }
}
```

L’autruche a des attributs d’oiseau, mais **ne veut pas** du comportement `fly()` → ❌ LSP violé

> ✅ Solution : utiliser une interface `FlyingBird`, ou une composition

### Héritage ≠ partage d'attributs uniquement

Il ne suffit pas que deux classes partagent des champs ou propriétés communes pour les relier par héritage. Le LSP nous pousse à nous poser cette question essentielle :

> « Les enfants **veulent-ils** hériter du comportement du parent ? »

L'héritage doit donc être **comportemental**, pas uniquement structurel.

---

## 2. Exemple de violation : `Video` vs `PremiumVideo`

```java
public class Video {
    private String title;
    private int time;
    private int likes;
    private int views;

    public double getNumberOfHoursPlayed() {
        return (time / 3600.0) * views;
    }

    public void playRandomAd() throws Exception {
        // play an ad
    }
}
```

```java
public class PremiumVideo extends Video {
    private int premiumId;

    @Override
    public void playRandomAd() throws Exception {
        throw new Exception("No Ads play during Premium videos");
    }
}
```

Le code va planter si on utilise `PremiumVideo` là où un `Video` est attendu et qu’on appelle `playRandomAd()`

---

## 3. Pourquoi ce code viole LSP ?

* La classe `PremiumVideo` **remplace une méthode valide par une exception** : elle ne respecte plus le contrat de `Video`.
* On ne peut **plus utiliser `PremiumVideo` à la place de `Video`** sans casser le programme.

> Cela viole l’esprit de l’héritage : on **brise la substituabilité**.

---

## 4. Refactorisation correcte : plusieurs approches

### Refactorisation par composition (limite possible)

```java
public class VideoManager {
    private String title;
    private int time;
    private int likes;
    private int views;

    public double getNumberOfHours() {
        return (time / 3600.0) * views;
    }

    public void playRandomAd() {
        // play an ad
    }
}
```

```java
public class Video {
    private VideoManager manager;

    public double getNumberOfHoursPlayed() {
        return manager.getNumberOfHours();
    }

    public void playRandomAd() {
        manager.playRandomAd();
    }
}
```

```java
public class PremiumVideo {
    private int premiumId;
    private VideoManager manager;

    public double getNumberOfHoursPlayed() {
        return manager.getNumberOfHours();
    }
}
```

Cette solution est meilleure, mais ne permet pas la substitution polymorphique directe avec une hiérarchie (`Video`/`PremiumVideo`).

### Limite de la composition

La **composition** permet d’éviter le problème en contournant l’héritage, mais ne permet **pas** d’utiliser `PremiumVideo` comme un `Video` dans un système typé.

C’est pourquoi on préfère dans ce cas :

---

### Refactorisation avec Interface Segregation Principle (préférée)

### Motivation : Pourquoi faire ?

L'Interface Segregation Principle (ISP) nous dit que :

> « Une classe **ne doit jamais être forcée** d'implémenter une interface qu'elle **n'utilise pas**. »

> Cela évite à `PremiumVideo` d’être forcée d’implémenter une méthode `playRandomAd()` qu’elle ne souhaite pas supporter.

### Interfaces :

```java
public interface VideoActions {
    double getNumberOfHoursPlayed();
}

public interface AdsActions {
    void playRandomAd();
}
```

### Implémentations :

```java
public class Video implements VideoActions, AdsActions {
    @Override
    public double getNumberOfHoursPlayed() {
        ...
    }

    @Override
    public void playRandomAd() {
        ...
    }
}

public class PremiumVideo implements VideoActions {
    @Override
    public double getNumberOfHoursPlayed() {
        ...
    }
}
```

Ainsi, `PremiumVideo` **n'implémente pas** l'interface `AdsActions` qu’elle ne veut pas → **substituabilité préservée**, **pas de méthode cassée**, et **séparation des responsabilités**.

---

## 5. 📍 Comment savoir que LSP est violé ou doit être appliqué ?

### Signes typiques de violation :

* Une méthode héritée est **override pour lancer une exception**
* Tu dois **tester le type** avec `instanceof` avant d'appeler une méthode
* Une sous-classe **modifie le comportement attendu** (retourne null, fait rien, modifie une sémantique)
* Les tests plantent quand tu injectes une sous-classe à la place d'une super-classe

### Cas fréquents :

* Une classe "fille" annule une fonctionnalité (comme `playAd`) en disant "je ne fais pas ça"
* Un constructeur ou une API accepte une superclasse, mais crashe si on lui passe une sous-classe

---

## 6. Étapes pour appliquer correctement le LSP

1. **Identifier le contrat** de la classe mère (ce qu’on attend d’elle)
2. Si une sous-classe ne peut pas remplir ce contrat **sans restriction**, **ne pas hériter** !
3. Utiliser une **interface spécifique** ou la **composition**
4. Refactoriser les méthodes pour qu’elles soient cohérentes dans toutes les sous-classes
5. S’assurer par des **tests** que les sous-classes fonctionnent partout où la super-classe est utilisée

---

## 7. Bénéfices de respecter LSP

* Code **plus robuste** et **prévisible**
* Facilite le **polymorphisme sûr**
* Moins de `instanceof`, moins d'exceptions inattendues
* Meilleure **réutilisabilité** des classes
* Favorise une **architecture propre et découpée**


