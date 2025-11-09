
# Create the helmchart
```
helm create webserver
```


# Follow along with the video
- Create the files per the video, copying and pasting from templates-original
- you can also use the files in the solution folder

# Install the first one
```
helm install subash-webapp-release webserver/ --values webserver/values.yaml --namespace custom-backend --create-namespace 
```

# Upgrade after templating
```
helm upgrade subash-webapp-release webserver/ --values webserver/values.yaml
```

# Accessing it
```
minikube tunnel
```

# Create dev/prod
```
k create namespace dev
k create namespace prod
helm install mywebapp-release-dev webserver/ --values webserver/values.yaml -f webserver/values-dev.yaml -n dev
helm install mywebapp-release-prod webserver/ --values webserver/values.yaml -f webserver/values-prod.yaml -n prod
helm ls --all-namespaces
```
