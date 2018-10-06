## nginx python installation steps in ubuntu

####switch user to root user

		sudo -i
#### update the packages

		sudo apt-get update => to update packages

1. install nginx

		`sudo apt-get install nginx`
	
		`service start nginx`

2. install python and pip:
	
		`sudo apt-get install python python-pip`

3. make dir 'python' in root directory: `cd /var/www/html`

		`mkdir python` for demo. you can create directory as you wish!!
		`chmod -R 777 python`

4. now install virtualenv in python directory:

		`cd /var/www/html/python`
		`python -m virtualenv myappenv` OR `pip install virtualenv myappenv`
	
		`source myappenv/bin/activate` => to activate virtualenv
	
		`pip install flask`
	
		`pip install gunicorn`

5. now create sample application code. 

		`cd /var/www/html/python`
	
		`touch app.py`
	
		`gedit app.py` => will open file. so `app.py` file as below
		============================================================
			#!/usr/bin/python
			# -*- coding: utf-8 -*-
	
			from flask import Flask,render_template, request, url_for, flash
			from random import randint
	
	
			app = Flask(__name__)
			app.secret_key = "xxxabctets"
	
	
			@app.route("/")
			def index():
				return 'Welcome to Python'
	
			if (__name__ == "__main__"):
				app.run(debug=True)
		==================================================

	save and close it

6. just test the python application in virtual env
		
		`python app.py`

		`deactivate` => to deactivate virtualenv

7. now configure nginx with WSGI application to run python app.

	1. for that we have installed gunicorn under virtualenv

			`cd /var/www/html/python`
	
			`touch gunicorn.conf`

			`gedit gunicorn.conf`, it will open file. paste below code.
			===================================	
			chdir = '/var/www/html/python'
			workers = 5
			proc_name = 'app'
			bind = '0.0.0.0:5003'
			timeout = 60
			accesslog = '/var/www/html/python/log/python_app.access.log'
			errorlog = '/var/www/html/python/log/python_app.error.log'
			============================================		
	
		Test gunicorn conf working or not ??
	
			`gunicorn --workers=2 -c /var/www/html/python/gunicorn.conf app:app` or
	
			`source myappenv/bin/activate`
		
			`gunicorn --workers=2 -c /var/www/html/python/gunicorn.conf app:app`

		now open your browser and run `http://0.0.0.0:5003`
	
		`deactivate`

		This process we need to run every time. To run permanently we need to add job in supervisorctl
		

	2. install supervisor job

			sudo apt-get install supervisor

			service supervisor restart

			cd /etc/supervisor/conf.d

			touch python_app.conf

			gedit python_app.conf

			[program:python_app]
			command=/var/www/html/python/myappenv/bin/python /var/www/html/python/myappenv/bin/gunicorn -c /var/www/html/python/gunicorn.conf app:app
			directory=/var/www/html/python/
			numprocs=1
			autostart=true
			autorestart=true
			user=root
			redirect_stderr=true
			stdout_logfile=/var/log/project/python_app.interface.info_stdout.log
			stderr_logfile=/var/log/project/python_app.interface.error_stderr.log

		save file and close it.

		now update the supervisor job and update new job conf.

			supervisorctl reread

			supervisorctl update

			supervisorctl reload

		start our new added job

			supervisorctl start python_app

	3. now change the nginx configuration to use reverse proxy.

			cd /etc/nginx/conf.d
	
			touch pythonapp.conf
	
			gedit pythonapp.conf
	
			server {
				listen 80;
				server_name python.example.com;
				root /var/www/html/python;
				error_log /var/log/nginx/python.example.com.error.log;
			
			    location / {
			        proxy_pass http://127.0.0.1:5003;
			        proxy_set_header Host $host;
			        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			    }
			}

	 	save and close it.

		now restart the nginx service

			`service nginx restart`
8. add host in /etc/hosts

		127.0.0.1 python.example.com

9. finally test our application in browser.

		http://python.example.com


		















