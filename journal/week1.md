# Week 1 — App Containerization
Beforehand..
#Open VS browser
#click on search to download python and docker extension
cd backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
python3 -m flask run --host=0.0.0.0 --port=4567
cd ..

#go to ports and unlock the port 4567
![image](https://user-images.githubusercontent.com/86881008/222914102-ac4649d6-f177-4024-af82-d6f514e8887d.png)

#click on the port link and add at the ending of it to api/activities/home
https://4567-nborzi-awsbootcampcrudd-i0oeuuhr8n4.ws-eu89.gitpod.io/api/activities/home
Conteinarized BackEnd 
#inside backend folder create a new file 'Dockerfile' and paste the following content 
docker run --rm -p 4567:4567 -it backend-flask
FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
unset FRONTEND_URL="*"
unset BACKEND_URL="*"

#run in the background 
docker container run --rm -p 4567:4567 -d backend-flask
