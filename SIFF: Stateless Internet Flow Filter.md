# SIFF: Stateless Internet Flow Filter

## Goal
enable receiver to control its traffic

## Key ideas:
- path fingerprints for traffic authorization
    - path fingerprint is used as a capability
    - only clients who know their path fingerprint get authorization
- Authorizede or "privileged" packets get priority over non-privileged packets
    - in bandwidth DoS, privileged packets are undisturbed by non-privileged packets

## Overview

Create two internet packet classes:
- unprivileged (best-effort): signaling and legacy traffic
- privileged: receiver controlled traaffic flows

privileged packets given priority at routers

Privileged packets flooding is impossible(with high probability)


## SIFF Handshake

EXP: **unknown. what is it**
DAT: data packet

![](https://i.imgur.com/nGZhAGt.png)

1. client C sends best-effort packet to server S, arriving packet accumulates capability.
2. If S wants to allow C to send privileged traffic, S sends capability back to C
3. C includes capability in packets to send at privileged level

## SIFF Marking: Unprivileged Packets

![](https://i.imgur.com/kmJNGEU.png)

- SIFF routers mark unprivileged packets
- Marking should be unpredictable
    - Hash with key known only to each router
- Markings unique to Sender/Receiver pair
    - Add source IP and dst IP to hash
- Hash calculation must be done in hardware for performance
- Server sends back the capability

## SIFF Marking: Privileged Packets

capability sent back will be reversed: 32 => 23

![](https://i.imgur.com/vLUUOgt.png)

- SIFF routers verify marking in the header
    - correct marking: router rotates it into MSB
    - incorrect marking: router drops packet
- Without receiver help, sender does not learn capability, cannot send privileged traffic
- IP Spoofing: capability does not reach attacker

## Problem: static privilege

- sender can abuse privilege

to solve: dynamic privilege

Solution: key switching
- routers change keys periodically, bubt maintain x>1 valid keys foreach time window
- receiver automatically gets new capabilities

![](https://i.imgur.com/IWxMU2c.png)


Here you can see the A is using the old key 0033. When it arrives R1, the old key matches and it get updated to the new key 2. So after rotation it becomes 2003(should be 3003 if no new key). 

In this way:
- as packet flow carries on, receiver receives updated markings
- if receiver want to continue to enable sender to send privileged traffic, receiver sends updated marking as capability to sender
- if receiver wants to terminate malicious flow, receiver simply stops updating sender with new capability, and routers will soon stop the flow early in network


## SIFF Performance

- DDoS: Attackers flood "forged(guessed)" privileged traffic
![](https://i.imgur.com/Yv1zQZe.png)


## Summary
- DoS-less sender/receiver communication
    - receivers can stop malicious flows
    - 1 unprivileged packet establishes privileged connection
- Lightweight at routers(stateless)
- No trust requiered between ISPs
- No authentication required at routers

## Limitations
- Not distinguish bad/good senders
    - An attacker could rotate machines for persistent attack
- Router upgrade is required
    - Path that does not have SIFF router may become congested by attack.
- Collusion attack is still a risk
    - If a malicious sender colludes with some intermediate router in route, the router could partly help the sender forge capability.
- Only granularity of host, not service
- Flooding of EXP packets
