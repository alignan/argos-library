---
title: "A resin.io example service"
date: 2017-11-25T14:34:13+01:00
draft: false
tags: [ "IoT", "Resin", "Raspberry Pi", "Docker", "Challenge 2017" ]
---

# A resin.io example service

This is part of my [Challenge to make 26 years before 2017 ends](https://github.com/alignan/things-to-do/blob/master/README.md).

This is part of my Project to build an autonomous and remote-controlled watering system for [my bonsai]({{< relref "my-new-bonsai.md" >}}), and part of my learning journey to use [resin.io]({{< relref "resin-io-about.md" >}}).  The first step was to understand how to deploy to resin.io and my target device.

## Setting up the repository and first tests

The project repository lives at:
https://github.com/alignan/kodama-guardian

Make sure to include your `SSH keys` in resin.io, alternatively you can just import your existing Github keys directly.

I used [the simple server example](https://github.com/resin-io-projects/simple-server-python) from resion.io as a reference for a first tour.

The `Dockerfile` configuration is straightforward, as a base image it can be either `%%RESIN_MACHINE_NAME%%` to allow multiple architecture support, or it can be defined for a specific target, i.e. `aspberry-pi2-python`.

the rest is just python.

Next step is to add the remote end to resin.io to our repository.

````bash
git remote add resin antonio_lignan@git.resin.io:antonio_lignan/antonio2rpi.git
````

And push to resin.io:

````bash
git push resin master
````

The following

````bash

[Info]     Starting build for antonio_lignan/antonio2rpi, user antonio_lignan
[Info]     Dashboard link: https://dashboard.resin.io/apps/738468/devices
[Info]     Building on arm01
[Info]     Fetching base image
[Warning]  No image tag given for resin/raspberry-pi2-python, using default (latest)
[==================================================>] 100%
[Info]     Building Standard Dockerfile project
[Build]    Step 1/7 : FROM resin/raspberry-pi2-python
[Build]    Step 2/7 : WORKDIR /usr/src/app
[Build]    Removing intermediate container 3b71c8e8f087
[Build]    Step 3/7 : COPY ./requirements.txt /requirements.txt
[Build]    Removing intermediate container de12dd545c91
[Build]    Step 4/7 : RUN pip install -r /requirements.txt
[Build]     ---> Running in f64451083aa5
[Build]    Collecting Flask==0.10.1 (from -r /requirements.txt (line 1))
[Build]      Downloading Flask-0.10.1.tar.gz (544kB)
[Build]    Collecting Werkzeug>=0.7 (from Flask==0.10.1->-r /requirements.txt (line 1))
[Build]      Downloading Werkzeug-0.12.2-py2.py3-none-any.whl (312kB)
[Build]    Collecting Jinja2>=2.4 (from Flask==0.10.1->-r /requirements.txt (line 1))
[Build]      Downloading Jinja2-2.10-py2.py3-none-any.whl (126kB)
[Build]    Collecting itsdangerous>=0.21 (from Flask==0.10.1->-r /requirements.txt (line 1))
[Build]      Downloading itsdangerous-0.24.tar.gz (46kB)
[Build]    Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->Flask==0.10.1->-r /requirements.txt (line 1))
[Build]      Downloading MarkupSafe-1.0.tar.gz
[Build]    Building wheels for collected packages: Flask, itsdangerous, MarkupSafe
[Build]      Running setup.py bdist_wheel for Flask: started
[Build]      Running setup.py bdist_wheel for Flask: finished with status 'done'
[Build]      Stored in directory: /root/.cache/pip/wheels/b6/
[Build]      Running setup.py bdist_wheel for itsdangerous: started
[Build]      Running setup.py bdist_wheel for itsdangerous: finished with status 'done'
[Build]      Stored in directory: /root/.cache/pip/wheels/fc/a8/66/24d655233c757e178d45dea2de22a04c6d92766abfb741129a
[Build]      Running setup.py bdist_wheel for MarkupSafe: started
[Build]      Running setup.py bdist_wheel for MarkupSafe: finished with status 'done'
[Build]      Stored in directory: /root/.cache/pip/wheels/88/a7/30/e39a54a87bcbe25308fa3ca64e8ddc75d9b3e5afa21ee32d57
[Build]    Successfully built Flask itsdangerous MarkupSafe
[Build]    Installing collected packages: Werkzeug, MarkupSafe, Jinja2, itsdangerous, Flask
[Build]    Successfully installed Flask-0.10.1 Jinja2-2.10 MarkupSafe-1.0 Werkzeug-0.12.2 itsdangerous-0.24
[Build]     ---> e88897418072
[Build]    Removing intermediate container f64451083aa5
[Build]    Step 5/7 : COPY . ./
[Build]     ---> cadb1b4320ce
[Build]    Removing intermediate container acfb3a3dfa45
[Build]    Step 6/7 : ENV INITSYSTEM on
[Build]     ---> Running in 085183d23e60
[Build]     ---> 185c660d19d9
[Build]    Removing intermediate container 085183d23e60
[Build]    Step 7/7 : CMD python src/example-flask.py
[Build]     ---> Running in 946bc786e890
[Build]     ---> a8c9ed31d3d3
[Build]    Removing intermediate container 946bc786e890
[Build]    Successfully built a8c9ed31d3d3
[Build]    Successfully tagged registry2.resin.io:443/antonio2rpi/
[Success]  Image created successfully
[Info]     Verifying image integrity...
[Success]  Image passed integrity checks!
[Info]     Uploading image to registry...
[==================================================>] 100%
[Success]  Image uploaded successfully!
[Info]     Check your dashboard for device download progress:
[Info]       https://dashboard.resin.io/apps/738468/devices
[Info]     Build took 44 seconds
[Info]     486.60 MB total image size
[Info]     480.67 MB resin/raspberry-pi2-python:latest
[Info]     5.93 MB user additions
			    \
			     \
			      \\
			       \\
			        >\/7
			    _.-(6'  \
			   (=___._/` \
			        )  \ |
			       /   / |
			      /    > /
			     j    < _\
			 _.-' :      ``.
			 \ r=._\        `.
			<`\\_  \         .`-.
			 \ r-7  `-. ._  ' .  `\
			  \`,      `-.`7  7)   )
			   \/         \|  \'  / `-._
			              ||    .'
			               \\  (
			                >\  >
			            ,.-' >.'
			           <.'_.''
			             <'

To git.resin.io:antonio_lignan/antonio2rpi.git
 * [new branch]      master -> master
````

And that's it.

## What a deployed application looks like

A cool thing I found about resin.io is their `Terminal`, which resembles the device's own terminal.  In the image below the running service is shown:

[![](/img/resinio-example-service/00.png)](/img/resinio-example-service/00.png)

