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

# Metrics




