## Week 1 — App Containerization
Beforehand..
Open VS browser
click on search to download python and docker extension

     cd backend-flask
     export FRONTEND_URL="*"
     export BACKEND_URL="*"
     python3 -m flask run --host=0.0.0.0 --port=4567
     cd ..

# go to ports and unlock the port 4567
![image](https://user-images.githubusercontent.com/86881008/222914102-ac4649d6-f177-4024-af82-d6f514e8887d.png)

# click on the port link and add at the ending of it to api/activities/home
https://4567-nborzi-awsbootcampcrudd-i0oeuuhr8n4.ws-eu89.gitpod.io/api/activities/home
# create the conatiner 
  $ docker build -t  backend-flask ./backend-flask

# Conteinarized BackEnd 
# inside backend folder create a new file 'Dockerfile' and paste the following content 
      FROM python:3.10-slim-buster

      ### Inside Container
      ### make a new folder inside container
      WORKDIR /backend-flask

      ### Outside Container -> Inside Container
      ### this contains the libraries want to install to run the app
      COPY requirements.txt requirements.txt

      ### Inside Container
      ### Install the python libraries used for the app
      RUN pip3 install -r requirements.txt

      ### Outside Container -> Inside Container
      ### . means everything in the current directory
      ### first period . - /backend-flask (outside container)
      ### second period . /backend-flask (inside container)
      COPY . .

      ### Set Enviroment Variables (Env Vars)
      ### Inside Container and wil remain set when the container is running
      ENV FLASK_ENV=development

      EXPOSE ${PORT}

      ### CMD (Command)
      ### python3 -m flask run --host=0.0.0.0 --port=4567
      CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]


# go back to the main directory (cd.. and run the docker) check with 'docker ps' to see if it is running
    $docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask

# Send Curl to Test Server
curl -X GET http://localhost:4567/api/activities/home -H "Accept: application/json" -H "Content-Type: application/json"


## Containerize Frontend

## Run NPM Install

We have to run NPM Install before building the container since it needs to copy the contents of node_modules

```
cd frontend-react-js
npm i
```

### Create Docker File

Create a file here: `frontend-react-js/Dockerfile`

```dockerfile
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```

### Build Container

```sh
docker build -t frontend-react-js ./frontend-react-js
```

### Run Container

```sh
docker run -p 3000:3000 -d frontend-react-js
```

### Create a docker-compose file

Create `docker-compose.yml` at the root of your project. Click with right button and select Docker compose up or write in ssh 'docker-compose up'. Once docker is up and running go to ports and unlock the port 3000, click on the link and you should be able to access the url of cruddur
![image](https://user-images.githubusercontent.com/86881008/222960081-28086096-99ba-49ae-9955-458a4467fbee.png)


```yaml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

## Adding DynamoDB Local and Postgres

We are going to use Postgres and DynamoDB local in future labs
We can bring them in as containers and reference them externally

Lets integrate the following into our existing docker compose file:

### Postgres

```yaml
services:
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data
volumes:
  db:
    driver: local
```

To install the postgres client into Gitpod

```sh
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev
```

### DynamoDB Local

```yaml
services:
  dynamodb-local:
    # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
    # We needed to add user:root to get this working.
    user: root
    command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
    image: "amazon/dynamodb-local:latest"
    container_name: dynamodb-local
    ports:
      - "8000:8000"
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
    working_dir: /home/dynamodblocal
```

Example of using DynamoDB local (create table, create item, list table, get records
https://github.com/100DaysOfCloud/challenge-dynamodb-local

## Volumes

directory volume mapping

```yaml
volumes: 
- "./docker/dynamodb:/home/dynamodblocal/data"
```

named volume mapping

```yaml
volumes: 
  - db:/var/lib/postgresql/data

volumes:
  db:
    driver: local
```
