Alex Fernandez
2DAW
2021/2022

# Recuperació 

## M8 - Desplegament d'Aplicacions Web
## UF2 - Servidors d'aplicacions web

Manual per al deploy de l'aplicació "Tweeter", projecte de final de grau utilitzant Docker.

### Enllaços als repositoris de codi

Repositori: [Frontend](https://bitbucket.org/alexfernandez05/tweeter_mern_frontend/src/main/)
Repositori: [Backend](https://bitbucket.org/alexfernandez05/tweeter_mern_backend/src/main/)

---

1. Descarregar els dos repositoris a la nostra màquina local.

----

2. Creem un arxiu `Dockerfile` a cada repositori.
    
    ```bash
    cd repositori/
    touch Dockerfile
    ```

---

3. Contingut `Dockerfile` del repositori frontend:
    
    ```bash
    # Indica la imatge base que utilitzarà per a seguir futures instruccions. Buscarà si la imatge es troba localment, en cas que no, la descarregarà d'internet.
    FROM node:18.2-alpine3.14
    
    # Indica el directori sobre el qual s'aplicaran les instruccions següents.
    WORKDIR /app

    # Afegeix arxius des del nostre directori local
    COPY ["package.json", "package-lock.json*", "./"] 
    
    # Executa la comanda especificada. S'usa per a instal·lar paquets en el contenidor.
    RUN npm install
    
    # Afegeix arxius des del nostre directori local
    COPY . .
    
    # Aquesta instrucció li especifica a docker que el contenidor escolta en els ports especificats en la seva execució.
    EXPOSE 3000
    
    # Comanda CMD que s'executarà dins el contenidor. Només pot existir una instrucció CMD en un DockerFile, si col·loquem més d'un, només l'últim tindrà efecte.
    CMD npm run dev -- --host
    ```

---

4. Contingut `Dockerfile` del repositori backend:
    
    ```bash
    # Indica la imatge base que utilitzarà per a seguir futures instruccions. Buscarà si la imatge es troba localment, en cas que no, la descarregarà d'internet.
    FROM node:18.2-alpine3.14

    # Indica el directori sobre el qual s'aplicaran les instruccions següents.
    WORKDIR /app

    # Afegeix arxius des del nostre directori local.
    COPY ["package.json", "package-lock.json*", "./"] 
    
    # Executa la comanda especificada. S'usa per a instal·lar paquets en el contenidor.
    RUN npm install

    # Afegeix arxius des del nostre directori local.
    COPY . .

    # Aquesta instrucció li especifica a docker que el contenidor escolta en els ports especificats en la seva execució.
    EXPOSE 4000

    # Comanda CMD que s'executarà dins el contenidor. Només pot existir una instrucció CMD en un DockerFile, si col·loquem més d'un, només l'últim tindrà efecte.
    CMD npm run dev
    ```

---

5. Creem les imatges de Docker a partir dels Dockerfiles que acabem de declarar:

    ```bash
    docker build -t frontend .
    docker build -t backend .
    ```

    Podem comprovar que s'han creat correctament amb la comanda: `docker images`.

---

6. Ara crearem un arxiu `docker-compose.yml` que ens permetrà aixecar els dos contenidors alhora amb una sola comanda:

    <br>
    És una eina per a definir i executar aplicacions Docker multicontenidor que permet simplificar l'ús de Docker a partir d'arxius YAML, d'aquesta forma és més senzill crear contenidors que es relacionin entre si, connectar-los, habilitar ports, volums, etc. Ens permet llançar una sola comanda per a crear i iniciar tots els serveis des de la seva configuració(YAML), això significa que pots crear diferents contenidors i al mateix temps diferents serveis en cada contenidor, integrar-los a un volum comú i iniciar-los i/o apagar-los, etc.<br>

    ```yml
    version: "3.7"

    services:
        backend:
            image: backend
            build: ./backend
            ports:
               - 4000:4000
            environment:
               - MONGO_URI=mongodb+srv://tweeter_admin:Dx8GwKMyZhgLXZ5S@cluster0.t3cu0.mongodb.net/test
               - JWT_SECRET=xxxxxxxxxxxxxxx
               - FRONTEND_URL=http://localhost:3000
               - EMAIL_USER=tweeter.social.network@gmail.com
               - EMAIL_PASS=xxxxxxx__
               - EMAIL_HOST=smtp.gmail.com
               - EMAIL_PORT=XXX
        frontend:
            image: frontend
            build: ./frontend
            ports:
               - 3000:3000
            environment:
               - VITE_BACKEND_URL=http://localhost:4000
            depends_on:
               - backend
    ```

    ```bash
    # services: Indica els serveis a utilitzar, podem anidar quantitat de serveis a aquesta instrucció, cada servei pot tenir qualsevol nom, però com bona pràctica el millor és donar noms explícits.

    # image: Permet tagear la imatge que es crearà per a instanciar el contenidor en el qual estarà muntat el servei.
    
    # build: S'utilitza per a indicar el context i indicar la ruta de l'arxiu Dockerfile per a construir el contenidor del servei.

    # ports: S'utilitza per a exposar els ports necessaris des del contenidor.

    # environment: S'utilitza per a definir variables d'entorn que es passaran al contenidor.
    
    # depends_on: S'utilitza per a establir la dependència i comunicació amb altres serveis
    ```

    Executem al terminal `docker-compose up` aixecarem els dos contenidors.

---

7. Ara podem accedir a la nostra aplicació a través del navegador `http://localhost:3000`.


---
---

# Passar a producció

1. Creem un arxiu `.dockerignore` a cada repositori.
    
    ```bash
    cd repositori/
    touch .dockerignore
    ```

    ```bash
    # /backend/.dockerignore 

    node_modules
    .dockerignore
    Dockerfile
    ```

    ```bash
    # /frontend/.dockerignore 

    node_modules
    .dockerignore
    Dockerfile
    Dockerfile.prod
    ```

2. Crearem un Dockerfile separat per a usar en producció anomenat `Dockerfile.prod`, al repositori `/frontend`.

    ```bash
    # build environment
    FROM node:18.2-alpine3.14 as build

    WORKDIR /app

    COPY ["package.json", "package-lock.json*", "./"] 

    RUN npm install

    COPY . .

    CMD npm run build -- --host


    # production environment
    FROM nginx:stable-alpine

    COPY --from=build /app/dist /usr/share/nginx/html

    EXPOSE 80

    CMD ["nginx", "-g", "daemon off;"]
    ```


    Aquí, aprofitem el patró de construcció de diverses etapes per a crear una imatge temporal utilitzada per a construir la imatge definitiva, els arxius estàtics React llestos per a producció, que després es copien en la imatge de producció. La imatge de compilació temporal es descarta juntament amb els arxius i carpetes originals associats amb la imatge. Això produeix una imatge ajustada i llista per a la producció.

    ---

    3. Amb el Dockerfile de producció, creem i etiquetem la nova imatge de producció:


    ```bash
    docker build -f Dockerfile.prod -t frontendbuild:prod .
    ```

    ---

    4. Creguem nou arxiu Docker Compose anomenat `docker-compose.prod.yml`.


    ```bash
    version: "3.7"

    services:
    backend:
        image: backend
        build: ./backend
        ports:
           - 4000:4000
        environment:
           - MONGO_URI=mongodb+srv://tweeter_admin:Dx8GwKMyZhgLXZ5S@cluster0.t3cu0.mongodb.net/test
           - JWT_SECRET=abc43689dgfsdkgo
           - FRONTEND_URL=http://localhost:3000
           - EMAIL_USER=tweeter.social.network@gmail.com
           - EMAIL_PASS=tweeter__
           - EMAIL_HOST=smtp.gmail.com
           - EMAIL_PORT=465
    frontend:
        image: frontendbuild
        build:
            context: ./frontend
            dockerfile: Dockerfile.prod
        ports:
           - 3000:80
        environment:
           - VITE_BACKEND_URL=http://localhost:4000
        depends_on:
           - backend
    ```

    ---

    5. Encenem el contenidor:

    ```bash
    docker-compose -f docker-compose.prod.yml up -d --build
    ```

    Ja podem accedir a la nostra aplicació a través del navegador `http://localhost:3000`.