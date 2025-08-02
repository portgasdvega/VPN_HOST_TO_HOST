## **Laboratorio VPN Host a Host con WireGuard (Parrot OS ↔ Kali Linux)**

### Objetivo

Establecer una conexión segura punto a punto (host-to-host) entre dos máquinas virtuales utilizando **WireGuard** como solución VPN. Se busca demostrar el cifrado del tráfico, la comunicación directa entre hosts y la capacidad de transporte de servicios (HTTP) sobre el túnel VPN.

---

### Herramientas utilizadas

| Componente           | Descripción                           |
| -------------------- | ------------------------------------- |
| Parrot OS            | Host físico y nodo A (VPN peer 1)     |
| Kali Linux VM        | Máquina virtual y nodo B (VPN peer 2) |
| WireGuard            | Herramienta VPN de cifrado moderno    |
| Python3              | Servidor HTTP simple en Kali          |
| Curl                 | Cliente HTTP en Parrot                |
| Wireshark (opcional) | Herramienta de captura de paquetes    |

---

### Topología de red

```
    [Parrot OS]                   [Kali Linux]
 IP real: 192.168.20.1        IP real: 192.168.20.128
 VPN: 10.10.10.1/24           VPN: 10.10.10.2/24
           ↕                         ↕
         [ WireGuard VPN Tunnel cifrado ]
              Puerto: 51820 (UDP)
```

---

### Claves utilizadas

**Claves públicas intercambiadas:**

* **Parrot (host físico):**

  ```text
  PublicKey = /koLfeC52bWhqurRi9yuNwOv3a26eAMjC4h1zvC9IQA=

  ```
* **Kali (VM):**

  ```text
  PublicKey = b24wB2Rj3A42iAfgrU1Uv0UZOz33bawMcmfDaYLi5R8=
  
  ```

---

### ⚙️ Archivos de configuración `wg0.conf`

#### 📄 En Parrot OS (`/etc/wireguard/wg0.conf`):

```ini
[Interface]
PrivateKey = OM5Ww/rvY9ds8X9fUhZZuJfg7PjgoxBSqqS/0DvVN24=
Address = 10.10.10.1/24
ListenPort = 51820

[Peer]
PublicKey = b24wB2Rj3A42iAfgrU1Uv0UZOz33bawMcmfDaYLi5R8=
AllowedIPs = 10.10.10.2/32
Endpoint = 192.168.20.128:51820
PersistentKeepalive = 25


```

#### 📄 En Kali Linux (`/etc/wireguard/wg0.conf`):

```ini
[Interface]
PrivateKey = cL3rzqbQnj9CBM2Hv1wBZae3wNJzpUBYUnkBgUzmj0g=
Address = 10.10.10.2/24
ListenPort=51820

[Peer]
PublicKey = /koLfeC52bWhqurRi9yuNwOv3a26eAMjC4h1zvC9IQA=
AllowedIps = 10.10.10.1/32
Endpoint = 192.168.20.1:51820
PersistentKeepalive=25

```

---

### Pruebas de conectividad

#### Ping desde Parrot a Kali:

```bash
ping 10.10.10.2
```

#### Ping desde Kali a Parrot:

```bash
ping 10.10.10.1
```

Ambos pings exitosos. Confirmación de túnel activo.

---

### Prueba de servicio: HTTP sobre VPN

#### 1. En **Kali**, levantar servidor web:

```bash
sudo python3 -m http.server 80
```

#### 2. En **Parrot**, acceder al servicio:

```bash
curl http://10.10.10.2
```

Se muestra el HTML con el contenido del directorio del usuario de Kali.
Desde Kali se recibe petición GET desde la IP VPN `10.10.10.1`.

---

### Validación técnica

* Tráfico entre Parrot y Kali viaja encapsulado en túnel WireGuard.
* La IP virtual utilizada (10.10.10.0/24) permite aislamiento completo del entorno real.
* WireGuard proporciona cifrado moderno con claves públicas, manteniendo integridad, confidencialidad y autenticidad.

---

### Capturas recomendadas para incluir (si lo deseas)

* `ping` entre hosts (terminal)
* `curl` mostrando contenido del servidor
* consola de Kali mostrando logs HTTP
* interfaz `wg0` con `sudo wg`
* captura de Wireshark del túnel

---

### Conclusiones

* Se logró establecer una VPN host-to-host funcional y cifrada usando WireGuard.
* La VPN permite transportar tráfico de red (ICMP, HTTP, etc.) entre sistemas.
* Este tipo de conexión es ideal para túneles seguros entre servidores o estaciones de administración.
* La práctica demuestra un uso realista de VPN en entornos de ciberseguridad y redes ofensivas/controladas.
