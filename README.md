# Backend | Plataforma iot

<div style="text-align: center">
    <img src="https://academy.dignal.com/wp-content/uploads/2023/01/dignal_academy_logo_site.png" width="400" alt="Dignal Academy"/>
</div>

## Contenido

- Este repositorio contiene el backend de nuestra plataforam iot, hecho en node.

# Uso

- Seguimos los pasos igual que el frontend hasta llegar a clonar el proyecto.
- Para iniciar debes clonar este repositorio en tu equipo por medio del siguiente comando de git:
- Si eres usuario windows debes clonar esta carpeta dentro de la carpeta raíz del WSL.

```
git clone git@github.com:Dignal-Electronics/iot-backend-2024.git
```

- El proyecto debe ser levantado usando docker, para ello primero debemos de instalar las dependencias usando el comando `npm install`:

```
docker compose -f .docker/compose.yaml run node npm install
```

- Después de instalar las dependencias debemos levantar el contendor:

```
docker compose -f .docker/compose.yaml up -d
```

## Configuración inicial

- Configurar archivo config.json ubicado en la carpeta config (solo development, en caso de ser necesario)

- Ejecutar comando `npx sequelize-cli db:create` para crear la base de datos

```
docker compose -f .docker/compose.yaml run node npx sequelize-cli db:create
```

- Ejecutar comando `npx sequelize-cli db:migrate` para construcción de tablas base.

```
docker compose -f .docker/compose.yaml run node npx sequelize-cli db:migrate
```

- Configurar archivo .env ubicado en el directorio raíz del proyecto

- Ejecutar el comando `npx sequelize-cli db:seed:all` para crear usuario demo.
```
docker compose -f .docker/compose.yaml run node npx sequelize-cli db:seed:all
```

* Luego de todo este procedimiento se tiene que enlazar el backend y el frontend para ello primero apagamos los contenedores con el comando:
* para ello debemos entrar al directorio del backend ( **wie-plataforma-iot/wie-backend-2024** ) )y lo mismo para el frontend
```
docker compose -f .docker/compose.yaml down
```

- Despues se crea un archivo **docker-compose** en la carpeta **wie-plataforma-iot** para que enlace al frontend y el backend
- para ello nos ubicamos en la terminal wie-plataforma-iot y ejecutamos **`code .`** para abrir una pantalla en el vscode y a ese nivel crear
- el archivo **`docker-compose.yaml`** con esto se conecta los contenedores del backend y el frontend
```
services:
  node-be:
    build: wie-backend-2024/.docker/node
    ports:
      - "88:88"
      - "2810:2810"
    volumes:
      - ./wie-backend-2024:/usr/app
    environment:
      - DB_HOST=mysql
      - DB_USER=root
      - DB_PASSWORD=admin
      - DB_NAME=iot
    depends_on:
      - mysql
    networks:
      - iot-network
    command: npm run start
  node-fe:
    build: wie-front-2024/.docker/node
    ports:
      - "8080:8080"
    volumes:
      - ./wie-front-2024:/usr/app
    networks:
      - iot-network
    command: npm run dev
  mysql:
    image: mysql:8.0
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=admin
      - MYSQL_DATABASE=iot
    volumes:
      - mysql-data:/var/lib/mysql
    networks:
      - iot-network
  emqx:
    image: emqx:5.7.1
    container_name: emqx
    environment:
      - "EMQX_NODE_NAME=emqx@node1.emqx.io"
      - "EMQX_CLUSTER__DISCOVERY_STRATEGY=static"
      - "EMQX_CLUSTER__STATIC__SEEDS=[emqx@node1.emqx.io,emqx@node2.emqx.io]"
    healthcheck:
      test: [ "CMD", "/opt/emqx/bin/emqx", "ctl", "status" ]
      interval: 5s
      timeout: 25s
      retries: 5
    networks:
      - iot-network
    ports:
      - 1883:1883
      - 8083:8083
      - 8084:8084
      - 8883:8883
      - 18083:18083

networks:
  iot-network:
    driver: bridge

volumes:
  mysql-data:
```

- al final desde la terminal y en la carpeta ***wie-plataforma-iot*** levantamos los contenedores con el comando:
```
docker compose up -d
```

- una vez que hemos dockerizado el frontend con el backend en un solo contenedor ejecutamos lo siguiente para crear base de datos, tablas y usuario inicial:
- Ejecutar comando npx sequelize-cli db:create para crear la base de datos
```
docker compose run node-be npx sequelize-cli db:create
```
- Ejecutar comando npx sequelize-cli db:migrate para construcción de tablas base.
```
docker compose run node-be npx sequelize-cli db:migrate
```
- Ejecutar el comando npx sequelize-cli db:seed:all para crear usuario demo.
```
docker compose run node-be npx sequelize-cli db:seed:all
```

