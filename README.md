# multiple-greenlight-installation
how to install multiple greenlight interfaces on a bigbluebutton server

# multiple-greenlight-installation
hi there you may want to have multiple administration panel for your bbb server so we are going to install two and even more greenlight web interfaces on our bigbluebutton server.

* step 1 :

we assume that you already installed your first greenlight interface using their official website >>> https://docs.bigbluebutton.org/greenlight/gl-install.html#installing-on-a-bigbluebutton-server

so it means that you have docker and docker-compose installed on your server.

* Step 2 :

now we want to install the second greenlight.

your first greenlight is in ~/greenlight directory lets make another directory for the second one.(root permission required)

>mkdir ~/greenlight2

>cd ~/greenlight2

* step 3 :
 run these commands :
 
>docker run --rm bigbluebutton/greenlight:v2 cat ./sample.env > .env

>docker run --rm bigbluebutton/greenlight:v2 bundle exec rake secret

now open .env file and copy the secret key that above command produced and paste it infront of "SECRET_KEY_BASE=" 

then run this command :

>bbb-conf --secret

and copy paste its result in .env file infront of "BIGBLUEBUTTON_ENDPOINT=" and "BIGBLUEBUTTON_SECRET="

run following command to check your configuration :

> docker run --rm --env-file .env bigbluebutton/greenlight:v2 bundle exec rake conf:check

if your config are fine you will see "passed" as result.

in next step run this comman to get docker-compose sample file:

>docker run --rm bigbluebutton/greenlight:v2 cat ./docker-compose.yml > docker-compose.yml

now open it using nano or vim and edit like this :



>container_name: my-second-greenlight-container

>change ports from 127.0.0.1:5000:80 to any port that you want like 127.0.0.1:5010:80 

>in db part change the port like this : 
>>from 127.0.0.1:5235:5235 to 127.0.0.1:5134:5134

you can enter any port that you want

now save the file and exit


* step 4:

open .env file and look for "RELATIVE_URL_ROOT=/b"
and change the "/b" to "/c" or any thing that you want but remember what you entered here because we want to insert it in nginx later
now save and exit 

next run this command to connect to database:

>export pass=$(openssl rand -base64 24); sed -i 's/POSTGRES_PASSWORD=password/POSTGRES_PASSWORD='$pass'/g' docker-compose.yml; sed -i 's/DB_PASSWORD=password/DB_PASSWORD='$pass'/g' .env

then run this command

>docker run --rm bigbluebutton/greenlight:v2 cat ./greenlight.nginx | sudo tee /etc/bigbluebutton/nginx/greenlight2.nginx

if you are installing 3rd or 4th greenlight change the greenlight2.nginx to something else

now open the nignx file of your greenlight :

>nano /etc/bigbluebutton/nginx/greenlight2.nginx

change all 5000 ports to 5010 or whatever that you entered before in docker-compose file
and change all "/b" to "/c" 
they are in these lines : 6,14 and 43

save this file and exit

now open this file:

>nano /etc/nginx/sites-available/bigbluebutton

if this code exist in the end of file delete it or comment it:

>location = / {
>>return 307 /b;
>>>}

reset nginx :

>service nginx restart

go to your directory:
>cd ~/greenlight2

and bring up the container:

>docker-compose up -d

if you did all things right you can access your second greenlight using this url:  https://yourserver.com/c and your first greenlight through https://yourserver.com/b

you can install more greenlight interface on your server using this tutorial.

good luck!







