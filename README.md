# How to setup and host fastapi project on ubuntu server

* connect to server via ssh
*  run updates & install pip and venv

```bash
sudo apt-get updates
sudo apt-get upgrade

sudo apt-get install pyhton3-pip
sudo apt-get install pyhton3-venv
```

* clone repo, create / activate virtual environment & install reqs

```bash
git clone https://github.com/username/repo_name
cd repo_name
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
# replace username and repo_bame with real values for your repository
```

* Install nginx

```bash
sudo apt-get install nginx
```

* update nginx configuration to proxy to localhost

```bash
sudo nano /etc/nginx/sites-available/default

```

* look for this block in the file

```
location / {
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    try_files $uri $uri/ =404;
}
```

* update it to look like the following,update the port to match the one uvicorn or gunicorn is running on.

```
location / {
    include proxy_params;                         
    proxy_pass http://127.0.0.1:8000;
}
```

* save and exit the file
    - ctrl x
    - Y
    - Enter

* Restart nginx and run your server

```bash
sudo systemctl restart nginx

uvicorn filename:app
# replace filename with your fastapi file name
```

* confirm if the web server is listening in public ip address and then stop the uvicorn server

* create service for project
note - remove {} and change the contents in {} to your values

```bash
sudo nano /etc/systemd/system/{project_name}.service
```

my example 
```bash
sudo nano /etc/systemd/system/caesar.service
```

* paste the following snipet
note - remove {} and change the contents in {} to your values

```
[Unit]
Description=Gunicorn instance to serve MyApp
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory={absolute path to your repo directory}
Environment="PATH={absolute path to your repo directory}/venv/bin"
ExecStart={absolute path to your repo directory}/venv/bin/gunicorn -w 4 -k uvicorn.workers.Uvicorn>

[Install]
WantedBy=multi-user.target
```

#### my example 

```
[Unit]
Description=Gunicorn instance to serve MyApp
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/home/ubuntu/caesar
Environment="PATH=/home/ubuntu/caesar/venv/bin"
ExecStart=/home/ubuntu/caesar/venv/bin/gunicorn -w 4 -k uvicorn.workers.Uvicorn>

[Install]
WantedBy=multi-user.target

```

save and exit the file (ctrl + x, y, Enter)

* start the service 
note - replace project_name with the name of your project
```bash
systemctl daemon-reload
sudo systemctl start project_name
sudo systemctl enable project_name

# to stop the server
sudo systemctl start project_name
```

* Done.