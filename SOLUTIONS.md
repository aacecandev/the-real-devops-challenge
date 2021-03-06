# THE REAL DEVOPS CHALLENGE - SOLUTIONS

## MOTIVATION

~~I feel highly motivated to~~... **Just  kidding!**

The goal of this PR is to IGZ Devops' Team that I'm the one they're looking for. Also, I will try to propose some improvements while following it.

I'm aware of IGZ's love for memes, so I will try to keep a *funny-formal* style while explaining what I'm doing, and why I did it.

So... allé voy!

## PRELIMINARY CONSIDERATIONS
I would like to remark a few things before getting my hands dirty:

* *My client* (IGZ) deserves all of my time and attention, but in order to have healthy SLAs in both ends, please remember I'm currently working in a full-time position and making ~~the latest~~ a PRODUCTION deployment for my employer (do you guys enjoy complex automated Nokia + Openstack deployments?). 
* With the previous consideration in mind, I will proceed in the following way:
  * The 1.0.0 release **is the basic solution** to this challenge. It aims to get things done and deliver value as fast as possible.
  * Improvements will come once release 1.0.0 is out. This means that further changes will be added once I communicate the solution is ready. Those following releases will be delivered tagged as 1.X.X
  * Given the GitHub limitations, **you will be evaluating the latest changes I do** (no branches, nor WIP, as this PR hides git-flow work in my own repository). To have a more accurate vision of what am I doing, please `git clone https://github.com/aacecandev/the-real-devops-challenge.git` and give a peek to the Git logs.
    * I recommend `git log --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset%n' --abbrev-commit --date=relative --branches` or `sudo <pkg manager install command> tig`.
  * I have some *daddy issues* with privative software, so I'll try to keep myself as close as possible to open source tools and mindset.
  * **IMAGES HAVE ALREADY BEEN PUSHED TO MY DOCKER HUB ACCOUNT**

Enough chattering, let's start!

## FIRST CHALLENGE

I am supposed to have a Mongo DB installation running on my localhost, shouldn't I? If so, please ask me to explain how to install it and configure the necessary stuff to have a working env var like this

``` bash
export MONGO_URI="mongodb://igz:<3@127.0.0.1:27017/?authSource=restaurant&authMechanism=SCRAM-SHA-256"
```

I considered using a containerized Mongo DB, but I opted out to install and configure it manually, because, you know, I need you to know I know how it works.

I had to solve a **problem with DB authentication**, which I'm not sure if takes part on this challenge or is an outdated issue. I didn't opened an issue on GitHub not to interfere wit other ngoing challenges. Anyway, you can find the logs [here](files/authentication_error).

@angelbarrera92 made me sweat so hard to find out **how extract a list element...**

### Checklist

* [x] I **strongly recommend** changing the `app.run` host variable from `0.0.0.0` to `127.0.0.1`. It's asecurity issue having a development server listening externaly.
    * **UPDATE**: This fix needs to be reverted to allow container to communicate using docker bridged network in challenge #3.
* [x] README.md should be updated with `export MONGO_URI="mongodb://igz:<3@127.0.0.1:27017/?authSource=restaurant&authMechanism=SCRAM-SHA-256"` because the authentication method is broken
  * [x] [Github issue](https://github.com/dcrosta/flask-pymongo/issues/142)
  * [x] [Authentication docs](https://pymongo.readthedocs.io/en/stable/examples/authentication.html)
* [x] **IMPROVEMENT**: Code can be simplified by calling the package methods [here](https://github.com/dcrosta/flask-pymongo/blob/master/flask_pymongo/helpers.py#L86) and [here](https://github.com/dcrosta/flask-pymongo/blob/master/flask_pymongo/helpers.py#L53)
* [ ] Error handling for
  * [x] main
  * [ ] API responses
  * [ ] DB connection with `mongo.cx.restaurant.command('ping')`
* [ ] Liveness and Readiness k8s proves

* [ ] Create a `init_db.sh` script that installs, initialize, and securize Mongo DB in an idempotent manner, e.g. using an Ansible role.
  * [x] [Create the DB with a Python script
    * [mongoimport with Python](https://www.geeksforgeeks.org/how-to-import-json-file-in-mongodb-using-python/)
      * **UPDATE** This feature was done with `pymongo` but removed later in challenge #2 to allow CI/CD automatization
* [ ]] Add a `init.sh` script that automates the Flask app dependency installation 
  * https://stackoverflow.com/questions/27691434/python-virtualenv-check-environment
  * Check that venv has been activated
  * Check for mongod status

## SECOND CHALLENGE

For this challenge I'm going to take the risk and make my first contact with GitHub Actions!

There's not much further improvement planned for this challenge, but I would like to have feedback on how I could make more of this specific challenge.

Wish me luck!

I did it! I needed a lot of browsing and reading, but finally I manage to get an automated test running in a multicontainer runner (both app and db)

I embraced the KISS principle and restricted testing only to a Python version for speed purposes.

To secure the pipeline, I used the envsubst CLI tool and replaced some *commercial secrets* with environment variables that I stored in a well-known security keeper provider like Micro$oft CI/CD tool.

Don't worry, I feel safe, I'm in la Mutua Madrileña ;)

### Update

Last moments before closing the PR, I have learned too how to run the test in CircleCI.

Thanks guys, now I have knowledge about 2 more CI/CD tools!

### Checklist

* [ ] [Allow multi environment testing in tox](https://tox.readthedocs.io/en/latest/example/basic.html#a-simple-tox-ini-default-environments)
* [ ] Use Gitlab CI/CD

## THIRD CHALLENGE

In this challenge, even though I've been asked to make the lightest, I'm using a CentOS 8 image as base image because [it is well known that light images such as alpine:latest](https://stackoverflow.com/questions/59186113/alpine-3-9-force-to-use-python-3-6) have quite particular configurations and bugs that can be source of security issues.

My question here is, is it worth to use [specific python docker images?](https://pythonspeed.com/articles/base-image-python-docker-images/) Maybe in future iterations it would be, in order to achieve multi environment testing, by using a different images with their respective environment variables for that specific Python version.

I spent some time in this challenge trying to achieve intra-container connection to the DB. [This lecture about container networking](https://pythonspeed.com/articles/docker-connection-refused/) was quite useful. I was missunderstanding some Docker networking concepts and trying to make the API container communicate with a external database without having it in the same network. **Restlesness is a bad guy**.

Anyway, I have followed as close as possible the [Best Practices Docker guide](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) and made my best to keep the image securized and also used the .dockerignore method to avoid copying useless files while building the image to the inferior layers

### Checklist

* [ ] Enable multi environment Python versions
* [x] ~~Achieve intra-container connection to the local Mongo DB process~~

## FOURTH CHALLENGE

Let's face it, building a Mongo DB Dockerfile is one of the most odious tasks out there. Aren't there kiddos out there, right? **It's a pain in the ass**

To complete this challenge I got help from [this issue](https://github.com/docker-library/mongo/issues/329), from the official Mongo DB repository image. I needed to created a [bash script](files/mongo-import.sh) to add the user "igz", as well as the DB (`restaurant`). After that, I followed challenge instructions, imported the data, and simply wroted a Dockerfile.
This base image is totally weird!

(I hate the close-source SW && DB containers...)

## FIFTH CHALLENGE

This is the easiest part, I really loved and still love the simplicity that compose brings to the container world. 

What did you just said? `podman-compose`?


There's not much to say here, I created my awesome compose yaml file and let it build the image image files (because you haven't built them yet to check if I'm a liar, don't you?)

Given that images are auto provisioned there wasn't need to use volumes here, just creating the ~~damm~~ network, opening the necessary ports and made dependency do the trick.

### Checklist
* [x] Give myself five

## FINAL CHALLENGE

The code for this challenge will be kept in the [k8s folder](k8s/deployment.yml)

I'm deploying to `kind`, in localhost, using the following structure:

``` bash
mongo
  deployment
  service - clusterIP
python
  deployment
  service - clusterIP / NodePort (w/o ingress) / LB (no)
  ingress + ingress controller
```

For security reasons I created a `serviceAccount` *restaurant* (didn't needed, but now you know I'm concerned about security). Also, I used `secrets` to store sensitive data, and spread them in the deployment to keep IGZ money safe.

Due to the fact that I have deployed to kind, I needed a special cluster initialization command:

``` bash
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF
```

And after `kubectl apply -f <all-my-stuff>` I also needed to deploy and ingress-controller which handles the ingress Kubernetes API object, and makes possible to curl directly from localhost to the restaurant API

``` bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/kind/deploy.yaml
```

This part is for sure the one I have to make the deeper refresh / keep learning, so I will not be writing complex files. Anyway, this is working!

### Checklist

* [ ] The DB should be a StatefulSet
* [ ] Implement Liveness and Readiness proves
* [ ] Get the CKA

So... I finished! How did I performed?

## TO-DO, BUT I'M NOT DOING

* [ ] I just deleted some dupes from [.gitignore](.gitignore), but it can be further improved, e.g. all .png could be excluded but those kept into assets folder, to avoid having data/images/this-funny-cat-image.png
* [ ] Can be tox suite and configuration files be updated?

## AUTHORS

* Alejandro Aceituna Cano - ¿Intelygenz? - <3
