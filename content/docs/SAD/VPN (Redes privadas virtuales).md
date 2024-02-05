# OpenVPN
## Acceso remoto
**Descripción del escenario**
- `cliente_red1`: tendrá IP 192.168.1.10, y conectado al `servidor` por la interfaz ens4
- `servidor`: en su interfaz ens4 tendrá IP 192.168.1.1, y en la ens5 la 172.22.0.1
- `cliente_red2`: en su interfaz ens4 tendrá IP 172.22.0.10, y conectado al `servidor`por la interfaz ens4.