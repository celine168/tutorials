# Connecting JupyterHub to OAuth

This tutorial teaches how to set up Google OAuth on JupyterHub. It loosely follows some of the instructions under the [jupyterhub/oauthenticator](https://github.com/jupyterhub/oauthenticator), but does not use the files in the repository.


# Getting your credentials from Google Console

Google Developers Console generates the client ID and client secret, which is needed to reconfigure JupyterHub.

Visit [https://console.developers.google.com/](https://console.developers.google.com/) 

Go to **Credentials**

![alt_text](images/Installing-OAuth0.png "image_tooltip")

Click **Create credentials**, choose **OAuth client ID**



<p id="gdcalert2" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/Installing-OAuth1.png). Store image on your image server and adjust path/filename if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert3">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/Installing-OAuth1.png "image_tooltip")


Fill out the OAuth consent screen

**Application name:** JupyterHub

**Logo:** [Logo of JupyterHub](https://jupyterhub.readthedocs.io/en/stable/_static/logo.png)

**Authorized domains: **gmail.com, ucdavis.edu

Click **Save**

Choose **Web Application **as your Application Type

Set your **Name** (ex. JupyterHub).

Under **Restrictions**, 

For **Authorized JavaScript origins**, add: 


```
http://<your domain>
https://<your domain>
```


Ex. 

[http://sealion.genomecenter.ucdavis.edu](http://sealion.genomecenter.ucdavis.edu/)

[https://sealion.genomecenter.ucdavis.edu](http://sealion.genomecenter.ucdavis.edu/)

For **Authorized redirect URIs**, add: 


```
http://<your domain>/hub/oauth_callback
https://<your domain>/hub/oauth_callback
```


Ex.

http://sealion.genomecenter.ucdavis.edu/hub/oauth_callback

https://sealion.genomecenter.ucdavis.edu/hub/oauth_callback

Click **Create**

You will receive your **client ID** and **client secret**.

In `jupyterhub-deploy-teaching/group_vars/jupyter_hosts`,

set the following under the section **OAuth**:


```
# Set to `true` to enable.
use_oauth: true

# The OAuth callback URL.
oauth_callback_url: 'http[s]://[your-host]/hub/oauth_callback'

# The OAuth client ID.
# TODO
oauth_client_id: 'FILL IN CLIENT ID HERE'

# The OAuth client secret.
# TODO
oauth_client_secret: 'FILL IN CLIENT SECRET HERE'

oauth_hosted_domain: 'ucdavis.edu'
oauth_login_service: 'UC Davis'
```



# Re-running Ansible


## Modifying config.yml to only re-run specific tasks

The `jupyter_hosts` file gives the settings for Ansible to set up JupyterHub. Changing the `jupyter_hosts` file alone doesn’t actually change anything to your JupyterHub configuration until you run Ansible Playbook again.

First, we’ll set tags on certain tasks so we don’t have to run all of the tasks in the Ansible playbook.

In `jupyterhub-deploy-teaching/roles/jupyterhub/tasks/config.yml`, add the tag `configuration` to each task:


```
- name: install jupyterhub config file
  template: src=jupyterhub_config.py.j2 dest={{jupyterhub_config_dir}}/jupyterhub_config.py owner=root group=root mode=0644
  become: true
  tags:
      	- configuration

- name: install jupyterhub cookie secret
  copy: src="../../../security/cookie_secret" dest={{jupyterhub_srv_dir}}/cookie_secret owner=root group=root mode=0600
  become: true
  tags:
      	- configuration
```



## Modifying the jupyterhub_config.py.j2 template file

The template file `jupyterhub-deploy-teaching/roles/jupyterhub/templates/jupyterhub_config.py.j2` generates the `jupyterhub_config.py` file when Ansible playbook is run. 

Comment out the line setting the proxy_auth_token.


```
#c.JupyterHub.proxy_auth_token = u'{{proxy_auth_token}}'
```


Keeping the proxy_auth_token setting caused errors for some of us when running jupyterhub.

This is most likely because we did not specify one in `jupyterhub_hosts`.

Then, we will run only the tasks in the Ansible playbook with the tag, `configuration`.


```
ansible-playbook -l local -u spicy --tags "configuration" --ask-become-pass deploy.yml
```


This will regenerate the `jupyterhub_config.py` in the `/etc/jupyterhub/` directory.


# Running JupyterHub

When running `jupyterhub`, the JupyterHub looks in the home directory for `jupyterhub_config.py`.

Move the `jupyterhub_config.py` file to your home directory. Alternatively, you can set a symbolic link from `/home/<username>/jupyterhub_config.py`

If you are SSHing into your server, make sure ports 8000 and 443 are redirected to your `localhost:8000` and `localhost:443` respectively.

Run `jupyterhub` and navigate to `localhost:8000` in your browser.


<!-- Docs to Markdown version 1.0β17 -->
