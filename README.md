# Vegetables Classifications Project 

## Description of the Problem :
- From vegetable production to delivery, several common steps are operated manually. Like picking, and sorting vegetables. Therefore, we decided to solve this problem using deep learning, by developing a model that can detect and classify vegetables. The model can be deployed on various devices and can also solve other problems related to the identification of vegetables, like automatic labeling of the vegetables without any need for human intervention.

For more information on the dataset it is found in Kaggle [here](https://www.kaggle.com/datasets/misrakahmed/vegetable-image-dataset)  

Steps to Download the Dataset from Kaggle are available in the notebook named ```notebook.ipynb``` (convenient :wink:) as well as
For more details on how to download the data to Google Colab, read [this](https://www.kaggle.com/general/74235)

**To solve this problem of vegetable classification, I used a CNN (Convolution Neural Network) based architecture and transfer learning eg. Xcpetion and also deployed the service to AWS lambda**

- Libaries/Language Used : Python,  Numpy, Matplotlib, Tensorflow, Keras, Tensorflow Lite, Flask, gunicorn, keras-image-helper.
- Technologies/Cloud : Google Colab (or any other GPU enabled Jupyter Notebook provider ), AWS EC2 instance, AWS ECR (Amazon Elastic Container Registry), Docker, AWS Lambda.

--- 
<br>

## Instructions on how to run the project -

- First install the requirements
  
  - ```pip install -r requirements.txt```

Note - Tensorflow can be installed for either CPU or GPU (details [here](https://www.tensorflow.org/install/pip))

- Check out ```notebook.ipynb``` for downloading the dataset from Kaggle, EDA for the Image Data, Training the model using transfer learning and Hyperparameter tuning.

After training the model, was saved and named ```xception_v4_1_10_0.998.h5``` (with accuracy around 99%), we will use this model for deployment.

- Now we convert the ```xception_v4_1_10_0.998.h5``` keras model into ```tflite``` format which is lightweight and used only for inference and deployment to the cloud. For converting the keras model from ```.h5``` format to ```.tflite``` format, we use the notebook ```tensorflow-lite-model.ipynb``` and convert the model as ```model.tflite``` which we will use further for deployment.
- Now we create the Flask app (using gunicorn production ready web server). By running the ```predict.py``` file using ```gunicorn --bind 0.0.0.0:9696 predict:app```.
- Open another terminal and run the ```test.py``` file using ```python test.py``` and we see the results/prediction of image classification by our model running as as web service (REST API).

- Now we have our model running as web service locally, but we will deploy it to the cloud using AWS Lambda and for that we need to package all the libraries and files in a ```Dockerfile``` to create a docker image. 

---
<br>

## To Run/Deploy the Project Locally on Docker a Container

1. Make sure your system has python > 3.9 version installed

2. Docker installed and running/active

3. Git installed

4. Clone the repository using --> ```git clone```

5. ```cd Vegetable-Classification```

6. To build the image for running the container run ```docker build -t <ImageName> .```

7. Command to run the docker container: ```docker run -it -p 8080:8080 --rm <ImageName>:latest```

8. Update the test.py file and change it to local run on local host/uncomment the line (read the file for more)

9. Now run the ```test.py``` file from your local system using ```python test.py```  and you will see the prediction from the model which has been deployed locally using the image for AWS lambda.


Now we can use this image and deploy it on AWS Lambda.

---
<br>

## Uploading the image to AWS ECR -
- We will us the ```aws cli``` for uploading the the docker image to AWS ECR repository.
-  To install the aws cli run ```pip install awscli```
- We need to configure the ```awscli``` on our local computer (See [here](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html))
- Make sure ```awscli``` is installed and configured using your credentials.
- Now we create the ecr empty repository locally using ```aws ecr create-repository --repository-name <RepoName>``` , which will return the URI for the repository, note it down.
- Now we need to login into the ecr registry as it is only available for the aws account holder (this will return the password for the ecr repository) using this command ```aws ecr get-login --no-include-email```
- Now we will tag the previously tested and created docker image with the above ```REMOTE_URI```, ```docker tag <ImageName>:latest ${REMOTE_URI}```
- Now we can push the created docker image to the ecr registry ```docker push ${REMOTE_URI}```
- Now we can check the aws ecr section using the AWS console that the image has been successfully pushed to ecr repository by going to Amazon ECR > Repositories.


---
<br>

## Deploying the Docker Image from ECR using the AWS Lambda

- Now we can use this image (pushed to ECR) for our lambda function. We will use the web console for using the lambda function, go to AWS Lambda > Create Function and then create a function from a container image.
- We need to change the default configurations, Edit > increase the memory to 1024 or more and timeout to 30 seconds
- Now go to test and create an test event after the setup, then we can test the event and see the same results as on the local setup
- Now we can use the created lambda function and expose it as a web service using an API gateway.
- Go to API Gateway > REST API, Choose New api > Choose a name > Create API.
- Go to Actions > Create resources > Set the endpoint name as predict and create it.
- Now choose the actions > Select the POST method, then choose the lambda function created earlier and save.
- Now grant the permission of gateway service to invoke the lambda function
- Click on test > and put the json in request body
- Now we can see that it responds with the classification result from our model which was deployed using the lambda function
- Now we can take this api gateway and expose it, create new stage, name it and click deploy
- Now it will give us the url which we can use to test the API gateway
- Now we can use our ```test.py``` script created earlier and modify the given url to test (uncomment the url as mentioned in the ```test.py``` script)

