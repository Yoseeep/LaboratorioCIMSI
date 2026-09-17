https://concordia2.atc.us.es
- Correo María José: mjmoron@us.es
- Despacho: B1.45

- Usuario: L0501
- Contraseña: CMSI2627
- Realm: Proxmox VE

Dentro de la máquina: 
- Usuario: ubuntu
- Contraseña: ubuntu

Cambiar configuración del teclado de Inglés a Español:
```bash
sudo loadkeys es # temporal
```

```bash
sudo nano /etc/default/keyboard
# cambiar "us" -> "es" 
```

