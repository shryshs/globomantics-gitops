GitOps for Kubernetes Application Deployment

Introduction

Imagine you are a DevOps Engineer at Globomantics, a rapidly growing tech startup. The company is focused on delivering an innovative platform for managing cloud resources. Your team is responsible for deploying and managing applications on Kubernetes using GitOps practices. First, you must set up the environment by installing Argo CD in your Kubernetes cluster and the Argo CD CLI. Then, you need to deploy the Globomantics application through the Argo CD CLI. Finally, you need to apply Git operations to ensure smooth GitOps updates and rollbacks.

Solution

Objective 1: Access the Kubernetes Machine
Use the Lab Credentials and Instant Terminal to log into the Kubernetes machine, and access its CLI (Command Line Interface, or prompt). You will then perform some basic Kubernetes checks.

Once the lab has spun up, and the Lab Credentials section is available on the right, copy its Public IP and store it locally for later use.

Also copy the Password, and store it for later use.

On the right, towards the top, click Instant Terminal. This will open a new browser tab showing the Instant Terminal command prompt.

At the Instant Terminal $ command prompt, replace <<Public IP>> with the Public IP you copied earlier, and enter the following command:

ssh cloud_user@<<Public IP>>
When prompted to continue connecting, enter y.

When prompted for the password, enter the Password you copied earlier.

You will be at the cloud_user@controlplane prompt.

If the prompt does not display @controlplane, but instead has @ip, the hostname isn't yet ready; log out by pressing Control+D, wait 30 seconds, and then ssh in again. Repeat until the prompt contains controlplane. This bit of the setup can take two to three minutes.

Get the Kubernetes cluster information.

kubectl cluster-info
The output shows the Kubernetes control plane is running. If you get other output or errors, the setup isn't yet complete. Periodically retry every 30 seconds until the output contains running. It can take about two to four minutes for the setup to complete.

List the Kubernetes nodes.

kubectl get nodes
The output shows the controlplane node with a status of Ready.

Great! You have successfully logged into AWS Console and verified the Kubernetes status.

Objective 2: Set up a Core GitOps Environment with Argo CD
As a DevOps engineer at Globomantics, you must set up a GitOps environment using Argo CD. You will install Argo CD in your Kubernetes cluster, followed by Argo CD UI and CLI. You must also connect a Git repository as the source of truth for infrastructure configurations.

Install Argo CD in your Kubernetes cluster.

Create the argocd namespace.

kubectl create namespace argocd
Expected Output: namespace/argocd created.

Apply the Argo CD installation manifest.

kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
Expected Output: Multiple resources created including serviceaccount/argocd-server and serviceaccount/argocd-application-controller.

List the Argo CD pods.

kubectl get pods -n argocd
The Argo CD pods will likely take a minute or two to complete their initialization. If pods are not in Running status, wait and retry the command periodically every 30 seconds until all pods show a Running status.

Expected Output: All pods show status Running (including argocd-server, argocd-application-controller, and the remainder of the pods).

Expose the Argo CD API server through Argo CD UI in the background.

kubectl port-forward svc/argocd-server -n argocd 8080:443 &
Note: The & at the end of the command runs the command in the background.

Expected Output: Forwarding from 127.0.0.1:8080 -> 443.

Press Enter to continue.

Verify the Argo CD UI is running.

wget --no-check-certificate https://localhost:8080
Expected Output: 200 OK response indicating the Argo CD UI is accessible.

Install and configure the Argo CD CLI.

Download the CLI.

curl -sSL -o argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x argocd
sudo mv argocd /usr/local/bin/
Note: Enter the Password from the Lab Credentials section when prompted.

Get the initial admin password.

argocd admin initial-password -n argocd
Expected Output: A random password string

Copy this password as you'll need it to login to Argo CD.

Login to Argo CD, replacing <password> with the admin password you copied a bit earlier.

argocd login localhost:8080 --username admin --password <password> --insecure
Expected Output: 'admin:login' logged in successfully.

Note: You can ignore the Unhandled Error, and note this is insecure, and more secure authentication via SSO would normally be recommended. However, this lab's focus is not on secure authentication, so using a password directly like this for this lab is fine.

Verify the CLI is installed.

argocd version
Expected Output: Shows Argo CD client and server version information.

Note: You can ignore the Unhandled Error here as well, and when this error crops up in some later argocd commands, you can ignore there as well.

Create an empty GitHub repository, and set up a fine-grained access token.

Open a new browser tab and navigate to github.com.

Sign in to your GitHub account, or create a new account if you don't have one.

Create a new repository.

Click the + icon in the top-right corner and select New repository.

Enter globomantics-gitops as the Repository name.

Do NOT initialize with README, .gitignore, or license.

Click Create repository.

Expected Output: A new empty repository is created with the URL https://github.com/<username>/globomantics-gitops.git.

Create a fine-grained personal access token.

Click your profile picture in the top-right corner and select Settings.

At the bottom of the left sidebar, click Developer settings <>.

Click Personal access tokens then Fine-grained tokens.

Click Generate new token.

Enter a Token name of GitOps Lab Token.

Under Repository access, select Only select repositories, then select the globomantics-gitops repository you just created.

Under Permissions click + Add permissions, then select Contents.

On the right of the new Contents row that's added, click the drop-down and select Read and write.

Click Generate token, and on the pop-up click Generate token again.

Copy your personal access token immediately and store it securely.

Install the GitHub CLI, and connect your personal Git repository to Argo CD.

Install GitHub CLI.

sudo apt-get install gh
Enter the Password from the Lab Credentials section when prompted.

If you get a lock error, use sudo kill <PID> on the listed process (replace <PID> with that number), wait a few seconds, then try the apt-get command again. You may need to repeat this once or twice.
Expected Output: Installation progress and completion message.

Store the GitHub token in the environment variable.

export GH_TOKEN=<your-github-token>
Note: Replace <your-github-token> with your actual GitHub personal access token.

Login to GitHub using your token.

gh auth login
Press Enter to select the default GitHub.com from the list of providers.

Expected Output: The value of the GH_TOKEN environment variable is being used for authentication..

Clone the original Globomantics repository.

gh repo clone ps-interactive/lab_gitops-for-kubernetes-application-deployment
Expected Output: Repository cloned successfully.

Navigate to the cloned repository.

cd lab_gitops-for-kubernetes-application-deployment
Change the remote origin to your new repository.

Replace <username> with your actual GitHub username.

git remote set-url origin https://github.com/<username>/globomantics-gitops.git
Configure Git to store GitHub credentials.

git config --global credential.helper store
Note: This will store the credentials in the ~/.git-credentials file when you enter them for the first time.

Push the repository to your GitHub account.

git push -u origin main --force
Enter your GitHub Username when prompted, and when prompted for you Password, enter the personal access token you created and copied earlier.

Expected Output: Branch 'main' set up to track 'origin/main'.

Note: The --force flag is used to overwrite the existing blank repository with the local repository.

Using Argo CD CLI, add your repository to Argo CD.

Add the repository to Argo CD.

Replace <username>, twice, with your GitHub username.

argocd repo add https://github.com/<username>/globomantics-gitops.git --username <username> --password $GH_TOKEN
Expected Output: Repository 'https://github.com/<username>/globomantics-gitops.git' added.

Verify the repository is added.

argocd repo list
Expected Output: Shows your repository in the list with status Successful.

Nice work! You've successfully set up a GitOps environment with Argo CD, and connected a personal Git repository to Argo CD.

Objective 3: Deploy and Manage an Application on Kubernetes via GitOps
As a DevOps engineer at Globomantics, you must deploy the company's API service using Kubernetes Deployments. You'll create a deployment with multiple replicas and specific resource requirements to ensure reliable service operation. You'll also verify the deployment status and pod health. You'll also understand how to use Argo CD to deploy the application to the Kubernetes cluster.

Define application manifests for the Globomantics application.

Navigate to the manifests folder to store the Kubernetes manifest files.

cd manifests
Create a Kubernetes deployment manifest named deployment.yaml.

vim deployment.yaml
In vim, prepare to paste the yaml properly by pressing :, then entering set paste.

Then press i to enter INSERT mode.

Paste the yaml content.

apiVersion: apps/v1
kind: Deployment
metadata:
  name: globomantics
  namespace: globomantics
spec:
  replicas: 2
  selector:
    matchLabels:
      app: globomantics
  template:
    metadata:
      labels:
        app: globomantics
    spec:
      containers:
      - name: globomantics
        image: nginx:1.27.5
        ports:
        - containerPort: 80
Note: The manifest includes:

replicas: Number of Pods to create
selector: MatchLabels for Pod selection
namespace: Target namespace for the deployment
Save and exit the file by pressing ESC, then typing :wq, and then Enter.

Create a service manifest named service.yaml.

vim service.yaml
Paste in the yaml content as before, saving and exiting the file.

apiVersion: v1
kind: Service
metadata:
  name: globomantics
  namespace: globomantics
spec:
  selector:
    app: globomantics
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
Note: The service manifest creates a ClusterIP service to expose the application internally.

Set the email and name to be used for git operations.

git config --global user.email "<your-email>"
git config --global user.name "<your-name>"
Replace <your-name> and <your-email> with your name and email address.

Commit your changes.

git add . 
git commit -m "Added deployment and service manifests"
git push origin main
Expected Output: Shows commit hash and push confirmation.

Synchronize application state with Argo CD.

Create an Argo CD application named globomantics in manual sync mode.

argocd app create globomantics \
--repo <repository-url> \
--path manifests \
--dest-namespace default \
--dest-server https://kubernetes.default.svc \
--sync-policy=manual
Replace <repository-url> with your actual repository URL.

Expected Output: application 'globomantics' created.

Synchronize your application.

argocd app sync globomantics
Expected Output: Shows sync progress and completion with Message as successfully synced.

Inspect and verify the deployed application.

Check the application status.

argocd app get globomantics
Expected Output: Shows application status as Healthy and Synced.

View resources in Kubernetes.

kubectl get all -n globomantics
Expected Output: Shows deployment, service, and pods in the globomantics namespace.

Good job! You've successfully deployed the application and synchronized application state with Argo CD.

Objective 4: Practice Core Git Operations for GitOps Workflow
As the next step, you must execute Git operations to manage your application lifecycle by modifying the deployment manifest, committing the changes, and synchronizing the application with Argo CD. You will then rollback your application to the previous state by reverting the last commit. Finally, you must monitor the application logs to verify the rollback.

Commit and push changes to the GitHub repository.

Open the deployment.yaml file.

vim deployment.yaml
Change the replicas count to 4.

Navigate to the line replicas: 2, and put the cursor over the 2 (one way is to use the arrow keys). Then press r, and then 4.

Save and exit the file by typing :wq, and then Enter.

Commit your changes.

git add . 
git commit -m "Updated replica count"
git push origin main
Expected Output: Shows commit hash and push confirmation.

Synchronize your application.

argocd app sync globomantics
Expected Output: Shows sync progress and completion with successfully synced.

Verify the deployment of the Git changes.

Check the deployment status.

kubectl get deploy -n globomantics
Expected Output: Shows 4/4 replicas available.

List the available Pods.

kubectl get pods -n globomantics
Expected Output: Shows four pods with status Running.

Perform rollback through Git.

Revert the last commit.

git revert HEAD -n
git commit -m "Reverted last commit"
git push origin main
Expected Output: Shows revert commit hash and push confirmation.

Synchronize again via Argo CD.

argocd app sync globomantics
Expected Output: Shows sync progress and completion.

Check the deployment status.

kubectl get deploy -n globomantics
Expected Output: Shows 2/2 replicas available.

Monitor GitOps pipeline status.

Check the logs of the application.

argocd app logs globomantics
Expected Output: Shows application logs and sync history.

Check the logs of the Deployment.

kubectl logs deploy/globomantics -n globomantics
Expected Output: Shows container logs from the deployment.

Well done! You've committed and pushed changes to the GitHub repository, verified the automatic deployment of the Git changes, performed rollback through Git, and monitored the application logs.

Conclusion
Congratulations! You've successfully completed the lab. You've learned how to set up a core GitOps environment with Argo CD, deploy and manage an application on Kubernetes via GitOps, and practice core Git operations for GitOps workflow.
