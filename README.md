[![MLOPs Python Cookbook with Github Actions](https://github.com/lisanaberberi/Python-MLOps-Cookbook/actions/workflows/main.yml/badge.svg)](https://github.com/lisanaberberi/Python-MLOps-Cookbook/actions/workflows/main.yml)

# Python MLOps Cookbook 
This is an example of a Containerized Flask Application the can be the core ingredient in many "recipes", i.e. deploy targets..

### Steps to run for this project:

   - Create a Github Repo (if not created)
   - Open AWS Cloud Shell
   - Create ssh-keys in AWS Cloud Shell
   - Upload ssh-keys to Github
   - Create scaffolding for project (if not created)
   -   Makefile, requirements.txt etc.


![mlops-color](https://user-images.githubusercontent.com/58792/121539559-c6787e80-c9d3-11eb-9f48-5d25924fad25.png)
* [Read Practical MLOps Online](https://learning.oreilly.com/library/view/practical-mlops/9781098103002/)
* [Purchase Practical MLOps](https://www.amazon.com/Practical-MLOps-Operationalizing-Machine-Learning/dp/1098103017)


## Github Container Registery
Feel free to test my ML project:  `docker pull ghcr.io/noahgift/python-mlops-cookbook:latest`


## Assets in repo

* `Makefile`:  [View Makefile](https://github.com/noahgift/Python-MLOps-Cookbook/blob/main/Makefile)
* `requirements.txt`:  [View requirements.txt](https://github.com/noahgift/Python-MLOps-Cookbook/blob/main/requirements.txt)
* `cli.py`: [View cli.py](https://github.com/noahgift/Python-MLOps-Cookbook/blob/main/cli.py)
* `utilscli.py`: [View utilscli.py](https://github.com/noahgift/Python-MLOps-Cookbook/blob/main/utilscli.py)
* `app.py`:  [View app.py](https://github.com/noahgift/Python-MLOps-Cookbook/blob/main/app.py)
* `mlib.py`:  [View mlib.py](https://github.com/noahgift/Python-MLOps-Cookbook/blob/main/mlib.py)Model Handling Library
* `htwtmlb.csv`: [View CSV](https://github.com/noahgift/Python-MLOps-Cookbook/blob/main/htwtmlb.csv) Useful for input scaling
* `model.joblib`: [View model.joblib](https://github.com/noahgift/Python-MLOps-Cookbook/raw/781053e4d45ebeeb64ecdf2dc1b896b338530aab/model.joblib)
* `Dockerfile`:  [View Dockerfile](https://github.com/noahgift/Python-MLOps-Cookbook/blob/main/Dockerfile)
*  `Baseball_Predictions_Export_Model.ipynb`:  [Baseball_Predictions_Export_Model.ipynb](https://github.com/noahgift/Python-MLOps-Cookbook/blob/main/Baseball_Predictions_Export_Model.ipynb)

![Course2-Duke-Flask-Containerized](https://user-images.githubusercontent.com/58792/110816231-289cd880-8259-11eb-8ab7-45c4ef5190ad.png)

### CLI Tools

There are two cli tools.  First, the main `cli.py` is the endpoint that serves out predictions.
To predict the height of an MLB player you use the following: ` ./cli.py --weight 180`

-Another example of predict function results is:

$ ./cli.py --weight 340
Output: 6 foot, 10 inches

![predict-height-weight](https://user-images.githubusercontent.com/58792/110970118-6c5e1380-8327-11eb-95b2-aeba679c0270.png)

The second cli tool is `utilscli.py` and this perform model retraining, and could serve as the entry point to do more things.
For example, this version doesn't change the default `model_name`, but you could add that as an option by forking this repo.

`./utilscli.py retrain --tsize 0.4`

Here is an example retraining the model.
![model-retraining](https://user-images.githubusercontent.com/58792/110986838-31b2a600-833c-11eb-977f-13143d4471c7.png)

Additionally the you can query the API via the CLI allowing you to change both the host and the value passed into the API.
This is accomplished through the requests library.

`./utilscli.py predict --weight 400`

![predict-cli](https://user-images.githubusercontent.com/58792/111043970-b6bcbe80-8413-11eb-9298-5d091e5db0dd.png)



### Flask Microservice

The Flask ML Microservice can be run many ways.

#### Containerized Flask Microservice Locally

You can run the Flask Microservice as follows with the commmand: `python app.py`.

```
(.venv) ec2-user:~/environment/Python-MLOps-Cookbook (main) $ python app.py 
 * Serving Flask app "app" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: on
INFO:werkzeug: * Running on http://127.0.0.1:8080/ (Press CTRL+C to quit)
INFO:werkzeug: * Restarting with stat
WARNING:werkzeug: * Debugger is active!
INFO:werkzeug: * Debugger PIN: 251-481-511
```

To serve a prediction against the application, run the `predict.sh`.


```
(.venv) ec2-user:~/environment/Python-MLOps-Cookbook (main) $ ./predict.sh                             
Port: 8080
{
  "prediction": {
    "height_human_readable": "6 foot, 2 inches", 
    "height_inches": 73.61
  }
}
```


#### Containerized Flask Microservice

Here is an example of how to build the container and run it locally, this is the contents of [predict.sh](https://github.com/noahgift/Python-MLOps-Cookbook/blob/main/predict.sh)

```
#!/usr/bin/env bash

# Build image
#change tag for new container registery, gcr.io/bob
docker build --tag=noahgift/mlops-cookbook . 

# List docker images
docker image ls

# Run flask app
docker run -p 127.0.0.1:8080:8080 noahgift/mlops-cookbook
```

#### Automatically Build Container via Github Actions and Push to Github Container Registery

To setup the container build process do the following.  This is also covered by Alfredo Deza in Practical MLOps book in greater detail.

```
  build-container:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Loging to Github registry
      uses: docker/login-action@v1
      with:
        registry: ghcr.io
        username: ${{ github.repository_owner }}
        password: ${{ secrets.BUILDCONTAINERS }}
    - name: build flask app
      uses: docker/build-push-action@v2
      with:
        context: ./
        #tags: alfredodeza/flask-roberta:latest
        tags: ghcr.io/noahgift/python-mlops-cookbook:latest
        push: true 
    
```

![container-registry](https://user-images.githubusercontent.com/58792/111001486-d8ee0800-8351-11eb-984a-967558023cc8.png)

#### Automatically Build Container via Github Actions and Push to Dockerhub Container Registery


## Build Targets

With the project using DevOps/MLOps best practices including linting, testing, and deployment, this project can be the base to deploy to many deployment targets.

[In progress....]


### Other Tools and Frameworks

[In progress....]

#### FastAPI

* [fastapi](https://fastapi.tiangolo.com)


### AWS

#### AWS App Runner

Watch a YouTube Walkthrough on AWS App Runner for this repo here:  https://www.youtube.com/watch?v=zzNnxDTWtXA

![mlops](https://user-images.githubusercontent.com/58792/119266194-d3196c00-bbb7-11eb-929e-77718411ffd5.jpg)

#### AWS Co-Pilot



