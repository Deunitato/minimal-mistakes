---
published: true
---
Apart from accepting in Json, uploading data through multipartform is useful thus this post is dedicated into researching more about this new way of uploading and sending data.
# Multipart Form Data

## Set up (Postman)
![kube-multiform-2.PNG]({{site.baseurl}}/img/kube-multiform-2.PNG)

1) Enter url here

2) Check header

3) Ensure that under body> form-data

![kube-multiform-3.PNG]({{site.baseurl}}/img/kube-multiform-3.PNG)


## Findings

### Using text
Using the original kube code of just transforming the string:

![kube-multiform-1.PNG]({{site.baseurl}}/img/kube-multiform-1.PNG)

- Entering a text is fine
> Note: The key must be in strData form

### Upload file
![kube-multiform-4.PNG]({{site.baseurl}}/img/kube-multiform-4.PNG)

```
Traceback (most recent call last): File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 2447, in wsgi_app response = self.full_dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1952, in full_dispatch_request rv = self.handle_user_exception(e) File "/usr/local/lib/python3.7/site-packages/flask_cors/extension.py", line 161, in wrapped_function return cors_after_request(app.make_response(f(*args, **kwargs))) File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1821, in handle_user_exception reraise(exc_type, exc_value, tb) File "/usr/local/lib/python3.7/site-packages/flask/_compat.py", line 39, in reraise raise value File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1950, in full_dispatch_request rv = self.dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1936, in dispatch_request return self.view_functions[rule.endpoint](**req.view_args) File "/usr/local/lib/python3.7/site-packages/seldon_core/wrapper.py", line 75, in TransformInput requestJson = get_request() File "/usr/local/lib/python3.7/site-packages/seldon_core/flask_utils.py", line 54, in get_request return get_multi_form_data_request() File "/usr/local/lib/python3.7/site-packages/seldon_core/flask_utils.py", line 36, in get_multi_form_data_request req_dict[fileKey] = request.files[fileKey].read().decode("utf-8") UnicodeDecodeError: 'utf-8' codec can't decode byte 0x89 in position 0: invalid start byte
```
> Using binData does not work either
> My guess is that the image must be encoded in base64

![kube-multiform-5.PNG]({{site.baseurl}}/img/kube-multiform-5.PNG)

> Converting the img to base64 doesnt work either

Error given:

```
Traceback (most recent call last): File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 2447, in wsgi_app response = self.full_dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1952, in full_dispatch_request rv = self.handle_user_exception(e) File "/usr/local/lib/python3.7/site-packages/flask_cors/extension.py", line 161, in wrapped_function return cors_after_request(app.make_response(f(*args, **kwargs))) File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1821, in handle_user_exception reraise(exc_type, exc_value, tb) File "/usr/local/lib/python3.7/site-packages/flask/_compat.py", line 39, in reraise raise value File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1950, in full_dispatch_request rv = self.dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1936, in dispatch_request return self.view_functions[rule.endpoint](**req.view_args) File "/usr/local/lib/python3.7/site-packages/seldon_core/wrapper.py", line 78, in TransformInput user_model, requestJson, seldon_metrics File "/usr/local/lib/python3.7/site-packages/seldon_core/seldon_methods.py", line 218, in transform_input user_model, features, class_names, meta=meta File "/usr/local/lib/python3.7/site-packages/seldon_core/user_model.py", line 245, in client_transform_input return user_model.transform_input(features, feature_names) File "/app/times2.py", line 25, in transform_input return "It has transformed, the old was " + X TypeError: can only concatenate str (not "bytes") to str
```


### Changing the code base to accept base64 code

Using JSON:

![kube-multiform-7.PNG]({{site.baseurl}}/img/kube-multiform-7.PNG)



Using multipart:

![kube-multiform-6.PNG]({{site.baseurl}}/img/kube-multiform-6.PNG)


```
"/usr/local/lib/python3.7/site-packages/flask/app.py", line 1950, in full_dispatch_request rv = self.dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1936, in dispatch_request return self.view_functions[rule.endpoint](**req.view_args) File "/usr/local/lib/python3.7/site-packages/seldon_core/wrapper.py", line 75, in TransformInput requestJson = get_request() File "/usr/local/lib/python3.7/site-packages/seldon_core/flask_utils.py", line 54, in get_request return get_multi_form_data_request() File "/usr/local/lib/python3.7/site-packages/seldon_core/flask_utils.py", line 25, in get_multi_form_data_request req_dict[key] = json.loads(request.form.get(key)) File "/usr/local/lib/python3.7/json/__init__.py", line 348, in loads return _default_decoder.decode(s) File "/usr/local/lib/python3.7/json/decoder.py", line 337, in decode obj, end = self.raw_decode(s, idx=_w(s, 0).end()) File "/usr/local/lib/python3.7/json/decoder.py", line 355, in raw_decode raise JSONDecodeError("Expecting value", s, err.value) from None json.decoder.JSONDecodeError: Expecting value: line 1 column 1 (char 0)
```

-> Doesnt work.. sighs

### Change code to just push out data without manupilation
- Change the code in such that it only returns the value
- Send a textfile instead of image
- The server recieves but returns an error

![kube-multiform-8.PNG]({{site.baseurl}}/img/kube-multiform-8.PNG)

- Sent image
> Failed without using `binData`
![kube-multiform-9.PNG]({{site.baseurl}}/img/kube-multiform-9.PNG)

> Console still throw errors

![kube-multiform-11.png]({{site.baseurl}}/img/kube-multiform-11.png)


# Metrics

Prerequsites:
- Installation of helm
- Prometheous

## Installing helm (WSL)
```
# Download the install shell script
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3

# Allow to Run
chmod 700 get_helm.sh
# Install
./get_helm.sh

# Confirm it works
$helm version
    version.BuildInfo{Version:"v3.0.2", GitCommit:"19e47ee3283ae98139d98460de796c1be1e3975f", GitTreeState:"clean", 		GoVersion:"go1.13.5"}
```

`helm init`

## Setting up helm


1. Installing the repo

`SELDON_VERSION=v1.1.0`
`SELDON_URL=https://github.com/SeldonIO/seldon-core.git`

`git clone $SELDON_URL --branch $SELDON_VERSION`

2.Now read the service account:

`kubectl create serviceaccount --namespace kube-system tiller`
`
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller`

`kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'`

3. Test if tiller is working

`helm version`

If running the command show "tiller has not be installed" and you have com, check resetting helm

3. Installing the chart using helm
- Ensure that the location is not in the repo (Confirm with Divya)

`helm install ./seldon-core-analytics --repo https://storage.googleapis.com/seldon-charts --na
mespace seldon-system`

4. Starting the server

`kubectl port-forward 
svc/filled-umbrellabird-grafana 3000:80 -n seldon-system`

Note: You can only do a localhost access (Understand how the structure of the application suppose to look like, users are not suppose to be able to access the endpoint)

5. Access the server
`localhost:3000` on your host web browser


## Resetting helm

Sometimes you might encounter a problem whereby the console might claim that you are missing`tiller` but attempting to install it might cause the console to claim that it has already been install.
To fix this, you have to reset your helm installation using the following:


Try deleting your cluster tiller:

```
kubectl get all --all-namespaces | grep tiller
kubectl delete deployment tiller-deploy -n kube-system
kubectl delete service tiller-deploy -n kube-system
kubectl get all --all-namespaces | grep tiller
```

Initialise it again:


`helm init`




