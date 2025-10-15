# Différents types de services systemd
NB: Recap fait par l'IA Claude

## Type=simple (défaut)

Le processus lancé par `ExecStart` reste au premier plan. Systemd considère le service comme démarré dès que le processus est lancé. C'est le type le plus courant pour les applications qui ne se daémonisent pas.

```ini
Type=simple
ExecStart=/usr/bin/mon-app
```

## Type=forking

Le processus lancé se daémonise (fork) et se met en arrière-plan. Le processus parent se termine, et systemd attend que le processus enfant soit lancé. C'est ce qu'utilise Tomcat car `startup.sh` lance le processus et revient.

```ini
Type=forking
ExecStart=/opt/apache-tomcat/bin/startup.sh
PIDFile=/opt/apache-tomcat/catalina.pid
```

## Type=oneshot

Le service exécute un script ou une commande unique et se termine. Utile pour les tâches ponctuelles (setup, configuration, nettoyage).

```ini
Type=oneshot
ExecStart=/usr/bin/mon-script.sh
RemainAfterExit=yes  # Conserve l'état "active" après exécution
```

## Type=notify

Le processus signale à systemd quand il est prêt, via `systemd-notify`. Plus fiable que `forking` car le timing est maîtrisé.

```ini
Type=notify
ExecStart=/usr/bin/mon-app-qui-notifie
```

## Type=dbus

Le service s'enregistre sur le bus D-Bus. Systemd surveille l'apparition du service sur le bus.

```ini
Type=dbus
BusName=com.example.MonService
ExecStart=/usr/bin/mon-app-dbus
```

## Type=idle

Le service démarre après que tous les autres services soient prêts, et uniquement si aucune autre tâche n'est en cours. Utile pour les services non critiques.

```ini
Type=idle
ExecStart=/usr/bin/mon-app
```

## Comparaison rapide

| Type | Cas d'usage | Exemple |
| --- | --- | --- |
| **simple** | Applications au premier plan | Node.js, Python Flask |
| **forking** | Applications qui se daémonisent | Tomcat, Apache, Nginx |
| **oneshot** | Tâches ponctuelles | Scripts de configuration |
| **notify** | Applications qui signalent | Services modernes (systemd-aware) |
| **dbus** | Services D-Bus | Services système avancés |
| **idle** | Services non critiques | Tâches de maintenance |

Pour **Tomcat**, `Type=forking` est correct puisque `startup.sh` lance le processus en arrière-plan et revient immédiatement.