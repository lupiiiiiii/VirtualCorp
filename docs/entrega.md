# VirtualCorp - Delegació 20

## 1. Adreçament (10.20.0.0/16)

| Router | Interfície | Adreça | Xarxa |
|---|---|---|---|
| mikrotik0 | ether2 | 10.20.0.225/30 | enllaç r0-r1 |
| mikrotik0 | ether3 | 10.20.0.193/27 | DMZ |
| mikrotik1 | ether2 | 10.20.0.226/30 | enllaç r0-r1 |
| mikrotik1 | ether3 | 10.20.0.229/30 | enllaç r1-r2 |
| mikrotik1 | ether4 | 10.20.0.1/25 | planta baixa general |
| mikrotik2 | ether2 | 10.20.0.230/30 | enllaç r1-r2 |
| mikrotik2 | ether3 | 10.20.0.233/30 | enllaç r2-r3 |
| mikrotik2 | ether4 | 10.20.1.1/24 | planta 1 |
| mikrotik2 | ether5 | 10.20.2.1/24 | planta 2 |
| mikrotik3 | ether2 | 10.20.0.234/30 | enllaç r2-r3 |
| mikrotik3 | ether3 | 10.20.3.1/25 | planta 3 |
| mikrotik3 | ether4 | 10.20.3.129/25 | planta 3 direcció (aïllada) |

mikrotik0 surt a Internet per ether1 (NAT de VirtualBox) amb masquerade.
La resta de routers tenen ruta per defecte cap al router anterior i
aprenen les xarxes per RIP.

## 2. Decisions

- Disseny adaptat de 10.1.x.x a 10.20.x.x a deploy_routers.sh.
- Clau pública del professor (configs/ssh/vcorp_auditor.pub) importada a
  l'usuari sysadmin dels quatre routers (router_base.sh), sense modificar-la.
- Les claus privades són a ~/.ssh i mai al repositori.

## 3. Problemes trobats i resolts

- VirtualBox no arrencava VMs (VERR_VMX_IN_VMX_ROOT_MODE): els mòduls KVM
  ocupaven la virtualització. Solució: descarregar kvm_intel i kvm abans
  d'arrencar. VirtualBox 7.0.16 només compila per al nucli 6.14.
- make ssh-keys regenera vcorp_auditor.pub; es va restaurar amb git checkout
  i es va verificar el hash (sha256 38bcf090...d456126).
- La importació de la clau de l'auditor fallava a mikrotik1 perquè RouterOS
  encara no registrava el fitxer pujat. Solució: sleep 3 entre scp i import.
- Pèrdues puntuals de paquets cap a Internet just després d'arrencar;
  probablement del NAT de VirtualBox. Els pings entre routers sempre 0%.

## 4. Limitacions

- Només mikrotik0 és accessible per SSH des de l'amfitrió; mikrotik1-3 es
  gestionen per la consola de VirtualBox.
- Amb claus importades, RouterOS 7 desactiva el login SSH per contrasenya.
- La contrasenya de sysadmin és a l'script (millora: gestionar-la com a secret).

## 5. Validació

[sysadmin@mikrotik0] > /ip address print
Flags: D - DYNAMIC
Columns: ADDRESS, NETWORK, INTERFACE, VRF
#   ADDRESS         NETWORK      INTERFACE  VRF 
0   10.20.0.225/30  10.20.0.224  ether2     main
1   10.20.0.193/27  10.20.0.192  ether3     main
2 D 10.0.2.15/24    10.0.2.0     ether1     main
[sysadmin@mikrotik0] > /user ssh-keys print
Columns: USER, KEY-TYPE, BITS, INFO
#  USER      KEY-TYPE  BITS  INFO                      
0  sysadmin  ed25519    256  vcorp_sysadmin@virtualcorp
1  sysadmin  ed25519    256  vcorp_auditor@virtualcorp 
[sysadmin@mikrotik0] > /ping 10.20.0.226 count=4 
  SEQ HOST                                     SIZE TTL TIME       STATUS      
    0 10.20.0.226                                56  64 1ms66us   
    1 10.20.0.226                                56  64 1ms908us  
    2 10.20.0.226                                56  64 2ms580us  
    3 10.20.0.226                                56  64 1ms686us  
    sent=4 received=4 packet-loss=0% min-rtt=1ms66us avg-rtt=1ms810us 
   max-rtt=2ms580us 

[sysadmin@mikrotik0] > /ping 10.20.0.230 count=4  
  SEQ HOST                                     SIZE TTL TIME       STATUS      
    0 10.20.0.230                                56  63 2ms704us  
    1 10.20.0.230                                56  63 2ms914us  
    2 10.20.0.230                                56  63 2ms915us  
    3 10.20.0.230                                56  63 2ms247us  
    sent=4 received=4 packet-loss=0% min-rtt=2ms247us avg-rtt=2ms695us 
   max-rtt=2ms915us 

[sysadmin@mikrotik0] > /ping 10.20.0.234 count=4 
  SEQ HOST                                     SIZE TTL TIME       STATUS      
    0 10.20.0.234                                56  62 2ms650us  
    1 10.20.0.234                                56  62 3ms625us  
    2 10.20.0.234                                56  62 3ms765us  
    3 10.20.0.234                                56  62 4ms608us  
    sent=4 received=4 packet-loss=0% min-rtt=2ms650us avg-rtt=3ms662us 
   max-rtt=4ms608us 

[sysadmin@mikrotik0] > /ping 1.1.1.1 count=4          
  SEQ HOST                                     SIZE TTL TIME       STATUS      
    0 1.1.1.1                                    56 254 16ms129us 
    1 1.1.1.1                                    56 254 17ms492us 
    2 1.1.1.1                                    56 254 17ms106us 
    3 1.1.1.1                                    56 254 17ms529us 
    sent=4 received=4 packet-loss=0% min-rtt=16ms129us avg-rtt=17ms64us 
   max-rtt=17ms529us 
