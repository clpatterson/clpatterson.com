---
title: "Dockerize_hotel_api"
date: 2020-09-07T11:45:51-04:00
draft: true
---

### Where to begin...
* What is docker compose? Nvm, let's start with something simple
* Which registry should I push my final image to? I'll push to docker registry first
* Where should I put my dockerfile? In a seperate directory or in my root project dir?
    * I tried to put my dockerfile in a sub directory and copy the dir two levels up, but got an error. Then I moved my dockerfile to root project directory and that solved the issue. I can look into this later.
* Psycopg2 wasn't pip installing properly when I used python 3.7-alpine. When I switched to the full python3.7 image, everything worked. Can I install the pyscopg2 package on apline version? If so how?
* Python standard library module dateutil is not working...can't be found on my image. Turns out that this dateutil module is not part of the standard lib and I hadn't added it to my requirements file.
* Next, I'm having an issue accessing my flask app while it is running from my docker container. I'm going to try specifying the host and port in my dev config. That didn't work. So now I'm explicitly exposing 5000 in my dockerfile...though I'm exposing it as a run cmd arg, so this shouldn't make a difference...So it turns out there is an issue with how my dev config is being passed to app.run(). When I manually pass configs...app.run(host='0.0.0.0', port=5000) it works.
* How can I rework my configuration, so that it behaves as expected?
* What are the different ways you can run (create) a flask application (instance)?
* Create the application instance globally (app = Flask(__name__)) in the app.py file. Invoke using flask run or app.run() in an if __name__ == '__main__' statement at the bottom of the file. 
* Use the application factory function to create the app instance.

### Glossary
* Container:
* Image:
* Dockerfile: is a text document that contains all the commands used by docker to create your docker image. Where does it live in a project? What does it typically contain?

### Resources
* 
