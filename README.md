
Scenario
=========

- Two namespaces : 
	- dev
	- prod 

- Two users
	- kubia
	- coddy

- Two kubernetes roles
	- Edit
	- View

- Relations between Namespace and user and role

	|  Namespace |  User  |  Role |
	|------------|--------|-------|
	| dev  		 |  coddy |  Edit |
	| prod 		 |  coddy |  View |
	| prod  	 |  kubia |  Edit |


Definitions from official Kubernetes documentation
===

	
>	**Edit Role** : Allows read/write access to most objects in a namespace.
	This role does not allow viewing or modifying roles or role bindings. However,
	this role allows accessing Secrets and running Pods as any ServiceAccount in the namespace,
	so it can be used to gain the API access levels of any ServiceAccount in the namespace.

> 	**View Role** : Allows read-only access to see most objects in a namespace. It does not allow viewing roles or role bindings.
	This role does not allow viewing Secrets, since reading the contents of Secrets enables access to ServiceAccount credentials in the namespace,
	which would allow API access as any ServiceAccount in the namespace (a form of privilege escalation).


User Creation
===

**kubia** user
---

	# Create the private key 
	openssl genrsa -out kubia.key 2048

	# Create the signature request file (csr)
	openssl req -new -key kubia.key -out kubia.csr

	# Create the signature file (crt) with the CA of kubernetes 
	sudo openssl x509 -req -in kubia.csr -CA /home/hamza/.minikube/ca.crt -CAkey /home/hamza/.minikube/ca.key -CAcreateserial -out kubia.crt -days 10000

**coddy** user
---

	# Create the private key 
	openssl genrsa -out coddy.key 2048

	# Create the signature request file (csr)
	openssl req -new -key coddy.key -out coddy.csr

	# Create the signature file (crt) with the CA of kubernetes 
	sudo openssl x509 -req -in coddy.csr -CA /home/hamza/.minikube/ca.crt -CAkey /home/hamza/.minikube/ca.key -CAcreateserial -out coddy.crt -days 10000


Create the users in kubernetets : Adding the certificate to the API server
===

**kubia** user
---

	kubectl config set-credentials kubia --client-certificate=kubia.crt --client-key=kubia.key

**coddy** user
---

	kubectl config set-credentials coddy --client-certificate=coddy.crt --client-key=coddy.key


Contexts Creation
===

**kubia** user
---

	kubectl config set-context kubia-context --user=kubia --cluster=minikube

**coddy** user
---

	kubectl config set-context coddy-context --user=coddy --cluster=minikube

Namespaces Creation
===

	kubectl create namespace dev

	kubectl create namespace prod


RoleBindings Creation
===

	cd kubia-authZ
	kubectl create -f coddy.rolebinding.yml --save-config

	cd coddy-authZ
	kubectl create -f coddy.rolebinding.yml --save-config

Check the configuration
===

	kubectl config view

	kubectl config current-context

	kubectl config use-context REPLACE_THIS_VAR_WITH_THE_CONTEXT_NAME

Display RoleBindings By Namespace
===

	kubectl get rolebinding -n dev

	kubectl get rolebinding -n prod

Example of use : coddy-context
===
	# Check the current context
	kubectl config current-context

	# Switch to the coddy-context 
	kubectl config use-context coddy-context

	# Sheck the current context
	kubectl config current-context

	# Try to create a pod in the namespace `prod` => FORBIDDEN
	kubectl create -f pods/nginx-pod.yml -n prod

	# Try to create a pod in the namespace `dev`  => OK
	kubectl create -f pods/nginx-pod.yml -n dev

Remove a user from the kube config
===

	# Replace USER_NAME by the real name
	kubectl config unset users.USER_NAME

Remove a context
===

	# Replace CONTEXT_NAME with the context to delete
	kubectl config delete-context CONTEXT_NAME



