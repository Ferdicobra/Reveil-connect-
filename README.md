# Réveil Connecté NTP — STM32F746G-Discovery

## Description du projet

L'objectif de ce projet est de réaliser un réveil connecté sur carte STM32F746G-Discovery. La carte récupère l'heure exacte depuis Internet via le protocole NTP, l'affiche sur l'écran LCD tactile, et permet à l'utilisateur de programmer une alarme directement depuis l'interface tactile.

## Matériel requis

- Carte STM32F746G-Discovery
- Câble USB (ST-LINK)
- Câble RJ45 (Ethernet) connecté à une box/routeur avec accès Internet

## Architecture FreeRTOS

L'intégration de FreeRTOS permet de découpler l'affichage, la gestion du temps, la synchronisation réseau et la logique d'alarme en plusieurs tâches indépendantes :

| Tâche | Priorité | Stack | Rôle |
|---|---|---|---|
| `StartDefaultTask` | Normal | 1024 | Init LCD, touch, lwIP, lance les autres tâches |
| `Clock_Thread` | Normal | 256 | Incrémente l'heure chaque seconde |
| `Display_Thread` | BelowNormal | 1024 | Rafraîchit l'écran et gère le touch |
| `NTP_Thread` | Low | 1024 | Synchronise l'heure via UDP port 123 |
| `Alarm_Thread` | Normal | 256 | Surveille l'heure et déclenche l'alarme |

## Protocole NTP

Le protocole SNTP fonctionne sur UDP port 123. La requête minimale consiste en un paquet de 48 octets avec le premier octet à `0x23` :

```c
Bits 7-6 : LI  = 0 (no warning)
Bits 5-3 : VN  = 4 (NTP version 4)
Bits 2-0 : Mode = 3 (client)
→ 0x23
```

Le Transmit Timestamp aux octets 40-43 contient le nombre de secondes depuis le 1er janvier 1900 :

```c
uint32_t unix_time = ntp_seconds - 2208988800UL + UTC_OFFSET_SEC;
```

## Interface tactile
