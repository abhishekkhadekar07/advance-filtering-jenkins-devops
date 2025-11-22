In this repo I have used github webhooks in which i have used link for hitting the post metthod is which i derived from ngrok by command ngrok http 80 then it given some link that link need to be updated everytime when it runs so we need to change webhook link which generated from ngrok http 80
In jenkins which is running locally which has docker and all other dependencies installed already
so in jenkins we need to set
credencials of docker in credentials
first create some pipe line in which set pipeline from scm which is present in the git repo itself from there it self it will run the automation also select GitHub hook trigger for GITScm polling for sure to run repo on push event
try1
