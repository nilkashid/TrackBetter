
# TrackBetter
As we know there are lot many ready to use tracking solutions. We prefer cost effective and reliable solutions. There are multiple major players in this race like Google, Here maps, MapBox and many more. But it comes with the a huge cost.

If you are looking for cost effective inhouse solution for tracking systems then you are at the right place. This repository contains sample code of backend structure and simple Android SDK which you can easily integrate and make a use of it. Provided mobile client will keep on pushing its lastest location called as <b>publisher</b> and other mobile client called as <b>subscriber

Solution primaraly contains 3 major components:

 * <b>OSRM Server: </b> Using open source OSRM project we can build this server. It will provide path, distance matrices, poly line                   details which you can plot it on map

* <b>Routing Server: </b> It will provide real time routing feature. Publisher(mobile client) will keep on pushing

* <b>Android SDK:</b> Android sample code to integrate tracking mechanisam.

Follow instructions given below to build each ot the component mentioned above

*<b> OSRM Server</b>

OSRM is free and open source routing engine. You can get the details regarding OSRM <a href="https://github.com/Project-OSRM/osrm-backend">here.</a>. For TrackBetter we have used docker setup of OSRM. Following instructions will help you to do so.

To build OSRM server you need to have a mahcine with at least 2GB of ram and 20GB of storage. OSRM depedens upon OpenStreetMap data. So we need to download it from <a href="http://download.geofabrik.de/">GeoFabrik.</a> 
Note: TrackBetter uses India's map data so url contains india-latest.osm.pdf. You can replace it with your require contry, city. Get the require map data from <a href="http://download.geofabrik.de/">GeoFabrik.</a> 

     wget http://download.geofabrik.de/asia/india-latest.osm.pbf
     
To run a OSRM docker image first we need to install <a href="https://docs.docker.com/engine/install/">Docker Engin.</a> Following instructions are for Ubunut OS.

    $ sudo apt-get update  
    $ sudo apt-get install docker-ce docker-ce-cli containerd.io  
    
    // to check whether it is installed it correctly or not.
    $ sudo docker run hello-world

Once you are done with Docker engine installation we can go ahead with the OSRM docker image. This process is bit of time cosuming but it is one time process.

    docker run -t -v $(pwd):/data osrm/osrm-backend:latest osrm-extract -p /opt/car.lua /data/india-latest.osm.pbf
    docker run -t -v $(pwd):/data osrm/osrm-backend:latest osrm-contract /data/india-latest.osrm
    docker run -t -i -p 5000:5000 -v $(pwd):/data osrm/osrm-backend:latest osrm-routed /data/india-latest.osrm
    
    curl 'http://localhost:5000/route/v1/driving/7.436828612,43.739228054975506;7.417058944702148,43.73284046244549?steps=true'
    
Find details about OSRM docker image https://github.com/Project-OSRM/osrm-backend/wiki/Docker-Recipes.

*<b>Routing Server</b>

While building TrackBetter we have used Node.js and MongoDB. give TrackBetter API contains subscriber and publisher APIs. Publisher mobile client will keep on publishing latest location updates to routing server. Routing server is going to store it. Subscriber client is going to subscribe those location updates.

To run routing server we need to have node and mongo install on your machine.

    $ sudo apt-get update
    $ sudo apt-get upgrade
    $ sudo apt install npm
    $ npm -v or npm –version
    
    $ sudo apt-get install -y mongodb-org
    $ sudo systemctl start mongod
    
    If you receive an error similar to the following when starting mongod:
    Failed to start mongod.service: Unit mongod.service not found.
    Run the following command first:
    
    $ sudo systemctl daemon-reload
    
Now we need to install require dependencies to run TrackBetter API server. Following depedendencies with specific version are require:

    "body-parser": "^1.19.0",
    "cookie-session": "^1.1.0",
    "cors": "^2.8.5",
    "express": "^4.17.1",
    "http": "0.0.0",
    "http-proxy": "^1.18.0",
    "kiteconnect": "^3.1.0",
    "mongo": "^0.1.0",
    "mongodb": "^3.5.5",
    "mongoose": "^5.9.5",
    "multer": "^0.1.8",
    "node-gzip": "^1.1.2",
    "npm": "^6.14.3",
    "redis": "^2.8.0",
    "request": "^2.88.2",
    "uniqid": "^5.2.0"
    
You can simply run following command
   
    $ sudo npm install --save express@4.17.1 body-parser mongoose request cookie-session cors http http-proxy kiteconnect mongo mongodb mongoose multer node-gzip redis uniqid --save
    
     Note: If you get the error while running TrackBetter API node server reverify your depedencies versions. You can downgrade or upgrade your depdency versions nd try it once agaian
     To intall specific version :
     
     $ npm install express@4.12.3 --save
     
Before running TrackBetter API server few things we need to configure at code level. 

    in app.js

		let dev_db_url = 'mongodb://localhost:27017/trackbetterdb'; // chnage it to your own database name and url or else you can also keep as it is

		inside controllers/location.controller.js inside get_publisher_last_location function 

		if(subscriberLat && subscriberLong && destLat && destLong){
						// replace your server IP
		                var url = "http://serverIp:5000/route/v1/driving/"+subscriberLong+","+subscriberLat+";"+destLong+","+destLat+"?steps=true";
		                //if you are running OSRM server on same machine then 
		                 url = "localhost:5000/route/v1/driving/"+subscriberLong+","+subscriberLat+";"+destLong+","+destLat+"?steps=true";
                     
Once above configuration is done you can simple run 

    $ node app.js
    
You can also use <a href="https://pm2.keymetrics.io/">PM2</a> which very much helpfull to handle production process for node.js. 

    $ npm install pm2 -g
    $ pm2 start app.js

Once setup is done you can start exploring API. You can get the API reference from routes/location.route.js
	
		folowing are the key apis curl:

		Publisher API:

		curl --location --request POST 'http://serverIp/location/publisher/update_location' \
		--header 'Content-Type: application/x-www-form-urlencoded' \
		--data-urlencode 'publisherId=54' \
		--data-urlencode 'latitude=12.9279234' \
		--data-urlencode 'longitude=77.632686' \
		--data-urlencode 'angle=1086' \
		--data-urlencode 'speed=22' \
		--data-urlencode 'acceleration=234'

		Or if it is running locally

			curl --location --request POST 'http://localhost:80/location/publisher/update_location' \
		--header 'Content-Type: application/x-www-form-urlencoded' \
		--data-urlencode 'publisherId=54' \
		--data-urlencode 'latitude=12.9279234' \
		--data-urlencode 'longitude=77.632686' \
		--data-urlencode 'angle=1086' \
		--data-urlencode 'speed=22' \
		--data-urlencode 'acceleration=234'	


		Subscriber API:

		curl --location --request GET 'http://serverIp/location/publisher/last_location?subscriberLong=77.6953754&subscriberId=234&subscriberLat=12.9705858&publisherId=123'
