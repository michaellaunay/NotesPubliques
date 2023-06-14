Ces principes sont des bonnes pratiques de conception orientée objet, établies pour rendre notre code plus compréhensible, flexible et maintenable.

1. **Principe de Responsabilité Unique (Single Responsibility Principle, SRP)** : Ce principe stipule qu'une classe doit avoir une seule raison de changer. En d'autres termes, chaque classe doit se concentrer sur une tâche spécifique ou une fonctionnalité. Cela augmente la cohésion, facilite la maintenance et favorise la lisibilité du code.

2. **Principe Ouvert/Fermé (Open/Closed Principle, OCP)** : Selon ce principe, les entités logicielles (classes, modules, fonctions, etc.) devraient être ouvertes à l'extension, mais fermées à la modification. C'est-à-dire qu'il devrait être possible d'ajouter des fonctionnalités ou de modifier le comportement d'une entité sans modifier son code source. Cela peut être réalisé en utilisant des interfaces, l'héritage et le polymorphisme.

3. **Principe de Substitution de Liskov (Liskov Substitution Principle, LSP)** : Ce principe dit qu'une sous-classe doit pouvoir se substituer à sa classe parent sans altérer le bon fonctionnement de notre programme. Cela assure que notre programme reste correct même lorsque nous introduisons des hiérarchies de classes et de l'héritage.

4. **Principe de Ségrégation des Interfaces (Interface Segregation Principle, ISP)** : Ce principe stipule que les clients ne devraient pas être forcés de dépendre des interfaces qu'ils n'utilisent pas. En d'autres termes, il est préférable d'avoir de nombreuses interfaces spécifiques à une tâche plutôt qu'une seule interface générale. Cela permet d'éviter que les modifications d'une partie du système n'affectent des parties qui ne l'utilisent pas.

5. **Principe d'Inversion des Dépendances (Dependency Inversion Principle, DIP)** : Selon ce dernier principe, les modules de haut niveau ne doivent pas dépendre des modules de bas niveau. Les deux doivent dépendre des abstractions. De plus, les abstractions ne doivent pas dépendre des détails ; les détails doivent dépendre des abstractions. Ce principe favorise le découplage entre les différents modules de notre application.
