# Net-Automation
#### _An automated program that looks for a mac address on a network of devices_


It is an automated program, it is used in a network of devices with the same password, the program asks for an ip to connect by ssh to a device and when entering the mac address that you want to search, the program will begin to connect to the different devices looking for the mac address in all devices until it finds it.
Use the following commands:
- Show mac address-table
- Show cdp neigbords
- Show ip arp 

## Import

- netmiko 
- ConnectHandler

> The show mac address-table command searches for the desired mac on the connected device, if it finds it it takes the port data and prints it.
>The show cdp neigbords detail command looks for the available neighbors and the ip to be able to connect by ssh.
>The show ip arp command is used to get the desired mac ip.

Es un programa automatizado, se usa en una red de dispositivos con una misma contraseña, el programa pide una ip para conectarse por ssh a un dispositivo y al ingresar la mac address que se desea buscar, el programa empezara a conectarse a los distintos dispositivos buscando la mac address en todos los dispositivos hasta encontrarla.
Usa los siguientes comandos:
- Show mac address-table
- Show cdp neigbords detail
- Show ip arp 

## Importar
- netmiko
- ConnectHandler

> El comando show mac address-table  busca la mac deseada en el dispositivo conectado, si lo encuentra toma el dato del puerto y lo imprime.
>El comando show cdp neigbords detail busca los vecinos disponibles y la ip para poderse conectar por ssh.
>El comando show ip arp se usa para obtener la ip de la mac deseada.

## Installation

El comando splitlines agrega la informacion del comando mac address-table a listas para que el programa pueda leer la informacion e imprime los datos con las pociciones de la mac address deseada.
```sh
        for line in output_mac_table.splitlines():
            if mac_address in line:
                datos = line.split()
                vlan = datos[0]
                mac = datos[1]
                puerto = datos[-1]
```

El siguiente comando busca la lista de vecinos y los agrega a una lista, obtiene las ip de los vecinos y los compara con una con una lista de dispositivos ya visitados, si aun no ha sido visitdo lo agrega a la lista.

```sh
        output_cdp = connection.send_command("show cdp neighbors detail")
        neighbors = []
        for line in output_cdp.splitlines():
            if "Device ID" in line:
                neighbor_name = line.split(":")[-1].strip()
            if "IP address" in line:
                neighbor_ip = line.split(":")[-1].strip()
                if neighbor_ip not in visitados:
                    neighbors.append((neighbor_name, neighbor_ip))
```

El siguiente for busca un vecino en una lista de vecinos. 

```sh
        for neighbor in neighbors:
            print(f"Buscando en vecino: {neighbor[0]} ({neighbor[1]})")
            device['host'] = neighbor[1]
            if buscar_mac_en_dispositivo(device, mac_address, visitados):
                return True
```

los datos necesarios para ingresar a los dispositivos.

Tambien se pide ingresar manualmente la mac address.

```sh
if __name__ == "__main__":
    username = "cisco"
    password = "cisco"
    host = "192.168.1.1"
    mac_address = input("mac address: ") 

    device_config = {
        'device_type': 'cisco_ios',
        'host': host,
        'username': username,
        'password': password,
    }
```

# Codigo 

    from netmiko import ConnectHandler

    def buscar_mac_en_dispositivo(device, mac_address, visitados):
        try:
      
        visitados.add(device['host'])

        connection = ConnectHandler(**device)
        print(f"Conectado al dispositivo: {device['host']}")

        #comando "show mac address-table"
        output_mac_table = connection.send_command("show mac address-table")
        print(output_mac_table)
        input("dar enter: ")
        prueba=output_mac_table.splitlines()
        print(prueba)
        input("dar enter: ")

        for line in output_mac_table.splitlines():
            if mac_address in line:
                datos = line.split()
                vlan = datos[0]
                mac = datos[1]
                puerto = datos[-1]

      
                output_interfaces = connection.send_command(f"show interfaces {puerto} switchport")
                if "Operational Mode: trunk" in output_interfaces:
                    print(f"La MAC {mac} está en un puerto trunk ({puerto}). Buscando en el vecino...")
                    break
                else:
                    print(f"MAC encontrada en {device['host']} - MAC: {mac}, VLAN: {vlan}, Puerto: {puerto}")
                    connection.disconnect()
                    return True

        # Si no se encontró la MAC, buscar en vecinos
        output_cdp = connection.send_command("show cdp neighbors detail") #Comando neighbords detail
        neighbors = []
        for line in output_cdp.splitlines():
            if "Device ID" in line:
                neighbor_name = line.split(":")[-1].strip()
            if "IP address" in line:
                neighbor_ip = line.split(":")[-1].strip()
                if neighbor_ip not in visitados:
                    neighbors.append((neighbor_name, neighbor_ip))

        connection.disconnect()

     
        for neighbor in neighbors:
            print(f"Buscando en vecino: {neighbor[0]} ({neighbor[1]})")
            device['host'] = neighbor[1]
            if buscar_mac_en_dispositivo(device, mac_address, visitados):
                return True

    except Exception as e:
        print(f"Error en el dispositivo {device['host']}: {e}")
        return False

    return False


    if __name__ == "__main__":
        username = "cisco"
        password = "cisco"
        host = "192.168.1.1"
        mac_address = input("mac address: ") 

    device_config = {
        'device_type': 'cisco_ios',
        'host': host,
        'username': username,
        'password': password,
    }

    dispositivos_visitados = set()

    print(f"Buscando la MAC {mac_address} en la red comenzando desde {host}...\n")
    encontrado = buscar_mac_en_dispositivo(device_config, mac_address, dispositivos_visitados)
    if not encontrado:
        print(f"La dirección MAC {mac_address} no fue encontrada en la red.")
    else:
        print("Búsqueda finalizada.")

