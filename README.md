Keepalived
==========

Ce rôle installe keepalived et le configure en fonction des variables que vous passerez au rôle.
Deux usages de keepalived sont possibles, correspondant aux modes suivants :
- Mode failover (par défaut) : Pour faire du failover automatique d'un service installé sur deux machines virtuelles. Les démons du service sont en actif-passif et keepalived doit être installé sur les deux machines virtuelles.
- Mode HA : Pour faire un loadbalancer L4 (tcp) avec au moins deux serveurs keepalived. Le loadbalancer fait du source NAT pour que se soit transparent pour les real servers.

Les variables du rôle
---------------------

Les valeurs par défaut des variables sont définies dans `defaults/main.yml`. Exemple:
```yaml
---
# keepalived_vrrp_instances:
#   # `name` définit une instance individuelle du protocole VRRP s'exécutant sur une interface.
#   - name: VI_1
#   # `state` définit l'état initial dans lequel l'instance doit démarrer.
#     state: MASTER
#   # `interface` définit l'interface sur laquelle VRRP s'exécute.
#     interface: eth0
#   # `unicast_src_ip` contient l'adresse primaire pour unicast
#     unicast_src_ip: "192.168.1.1"
#   # `secondary_private_ip` fait référence à la 2ème adresse ip
#     secondary_private_ip: "192.168.1.2"
#   # `virtual_router_id` l'identifiant unique.
#     virtual_router_id: 51
#   # `priority` est la priorité annoncée.
#     priority: 255
#   # `authentication` spécifie les informations nécessaires pour que les serveurs participant au VRRP s'authentifient les uns avec les autres.
#     authentication:
#       auth_type: PASS
#       auth_pass: 12345
#   # `virtual_ipaddress` définit les adresses IP (il peut y en avoir plusieurs) dont VRRP est responsable.
#     virtual_ipaddresses:
#       - name: "192.168.122.200"
#         cidr: 24
```
Trouver la liste complète des variables sur le site officiel de keepalived : https://keepalived.org/manpage.html

Exemple de Playbook
----------------

Voici comment vous pourriez utiliser le rôle :

```yaml
---
- name: converge
  hosts: all
  become: yes
  gather_facts: yes

  roles:
    - role: keepalived
      keepalived_vrrp_instances:
        - name: VI_1
          state: MASTER
          interface: eth0
          unicast_src_ip: "172.17.0.6"
          secondary_private_ip: "172.17.0.7"
          virtual_router_id: 51
          priority: 255
          authentication:
            auth_type: PASS
            auth_pass: "12345"
          virtual_ipaddresses:
            - name: "172.17.0.8"
              cidr: 16
```

Un autre exemple:


```yaml
---
- hosts: keepalived
  roles:
    - role: ansible-keepalived
      global_defs:
        notification_email:
          - email1@example.com
          - email2@example.com
        notification_email_from: 'keepalived@example.com'
      vrrp_instances:
        - name: 'VI_01'
          state: 'MASTER'
          interface: 'eth0'
          virtual_router_id: 10
          priority: 100
          smtp_alert: True
          authentication:
            auth_type: 'PASS'
            auth_pass: 'secpass'
          virtual_ipaddress:
            - '192.168.200.17/24 dev eth1'
            - '192.168.200.18/24 dev eth2 label eth2:1'
```  

Voir d'autres exemples dans keepalived/examples/