# Dependency Inversion Principle (DIP) – Principe d’inversion des dépendances

## Définition

Le **Dependency Inversion Principle** stipule que :

> « Les modules de haut niveau ne doivent pas dépendre des modules de bas niveau. Les deux doivent dépendre d’abstractions. »
>
> « Les abstractions ne doivent pas dépendre des détails. Les détails doivent dépendre des abstractions. »

---

## Exemple AVANT (Violation du DIP)

```java
public class OrderService {
    private final MySQLOrderRepository repository = new MySQLOrderRepository();

    public void save(Order order) {
        repository.save(order);
    }
}
```

Ici, `OrderService` dépend **directement** d'une implémentation concrète `MySQLOrderRepository`. Il n’est pas flexible ni testable.

---

## Exemple APRÈS (Respect du DIP via injection de dépendance)

### Interface abstraite

```java
public interface OrderRepository {
    void save(Order order);
}
```

### Implémentation concrète

```java
public class MySQLOrderRepository implements OrderRepository {
    @Override
    public void save(Order order) {
        // implémentation réelle
    }
}
```

### Service découplé

```java
public class OrderService {
    private final OrderRepository repository;

    public OrderService(OrderRepository repository) {
        this.repository = repository;
    }

    public void save(Order order) {
        repository.save(order);
    }
}
```

Grâce à l’abstraction, on peut injecter n’importe quel `OrderRepository`, comme une version test ou une autre base de données.

---

## Comment savoir que DIP est violé ou doit être appliqué ?

* Une classe **instancie directement** ses dépendances (`new` dans la classe)
* Impossible de **tester la classe sans dépendances réelles**
* Difficile de changer d’implémentation sans modifier la classe cliente

---

## Faut-il toujours abstraire les bibliothèques externes ?

### Règle générale :

> **Tu ne dois pas dépendre directement d’une bibliothèque externe**, mais plutôt d’une **abstraction** que **ton application contrôle**.

### À isoler via abstraction :

* Bibliothèques de **communication externe** : HTTP clients, accès BDD, envoi mail, API tierces
* Composants que tu veux **tester / simuler** (mock)
* Libs susceptibles d’être **remplacées ou obsolètes**

### Utilisation directe acceptable :

* Libs **utilitaires** (Apache Commons, UUID, StringUtils, Math…)
* Libs **stables et génériques** (Jackson, DateTime, etc.)
* **API standard** du JDK

### Exemple d’abstraction sur lib externe :

```java
public interface MailSender {
    void sendEmail(String to, String content);
}

public class SendGridMailSender implements MailSender {
    public void sendEmail(String to, String content) {
        // Appel à la lib SendGrid
    }
}

public class UserService {
    private final MailSender mailSender;

    public UserService(MailSender mailSender) {
        this.mailSender = mailSender;
    }

    public void register(User user) {
        mailSender.sendEmail(user.getEmail(), "Bienvenue !");
    }
}
```

Facilement testable, et tu peux remplacer SendGrid par MailJet sans toucher à `UserService`

---

## Étapes pour appliquer le DIP

1. Identifier les **modules de haut niveau** (services, logique métier)
2. Identifier les **détails techniques** (base de données, fichiers, APIs externes)
3. Créer une **abstraction (interface)** pour isoler les détails
4. Faire dépendre les services **des abstractions, pas des classes concrètes**
5. Utiliser un framework (comme Spring) ou une injection manuelle

---

## Bénéfices obtenus

* Code **testable** (mock facile)
* Architecture **flexible** (changement de base ou implémentation facile)
* Meilleure séparation des responsabilités
* Respect du principe **Open/Closed** (on peut injecter de nouvelles variantes sans modifier le code client)


