# cs-fastly-bouncer

A bouncer that syncs the decisions made by CrowdSec with Fastly's VCL. Manages multi account, multi service setup. Supports IP, Country and AS scoped decisions.


# Installation:

## Using Docker wrapper (quickest)

Make sure you have docker and git installed.

```console
git clone https://github.com/crowdsecurity/cs-fastly-bouncer 
cd cs-fastly-bouncer
sudo ./install.sh 
sudo crowdsec-fastly-bouncer \
     -c /etc/crowdsec/bouncers/crowdsec-fastly-bouncer.yaml\
    -g <FASTLY_TOKEN_1>,<FASTLY_TOKEN_2> \
     -o /etc/crowdsec/bouncers/crowdsec-fastly-bouncer.yaml # generate config
sudo vim /etc/crowdsec/bouncers/crowdsec-fastly-bouncer.yaml # Review the generated config, add captcha keys etc.
sudo systemctl start crowdsec-fastly-bouncer
```

**Note:** The above installation runs the bouncer inside a docker container. Hence it will have access just to the config files which are mounted. By default `/etc/crowdsec/bouncers/crowdsec-fastly-bouncer.yaml` and `/var/log/crowdsec-fastly-bouncer.log` are mounted. To modify this behaviour please edit the file at `/usr/local/bin/crowdsec-fastly-bouncer`

## Using python pip 

Make sure you have python3.8+ installed. Now in a virtual environment run the following:

```
git clone https://github.com/crowdsecurity/cs-fastly-bouncer 
cd cs-fastly-bouncer
pip install ./ 
crowdsec-fastly-bouncer -g <FASTLY_TOKEN_1>,<FASTLY_TOKEN_2> > config.yaml # generate config
vim config.yaml # Add crowdsec LAPI key, url, recaptcha keys etc
crowdsec-fastly-bouncer -c config.yaml # Run it !
```

See how to obtain fastly account tokens [here](https://docs.fastly.com/en/guides/using-api-tokens). The tokens must have write access for the configured services.

**Note:** If your bouncer is not installed on the same machine than LAPI, don't forget to set the `lapi_url` and `lapi_key` in the configuration file /etc/crowdsec/bouncers/crowdsec-fastly-bouncer.yaml

**Note:** For captcha to work you must provide the `recaptcha_site_key` and `recaptcha_secret_key` for each service. Learn how [here](http://www.google.com/recaptcha/admin)

## Deploy the new configuration:

After starting the bouncer, go in the fastly web UI. For each configured service review the version created by the bouncer. If everything looks good, you can deploy the new configration !

# Configuration:

```yaml
crowdsec_config: 
  lapi_key: ${LAPI_KEY} 
  lapi_url: "http://localhost:8080/"

fastly_account_configs:
  - account_token: # Obtain this from fastly
    services: 
      - id: # The id of the service
        recaptcha_site_key: # Required for captcha support
        recaptcha_secret_key: # Required for captcha support
        max_items: 5000 # max_items refers to the capacity of IP/IP ranges to ban/captcha. 

update_frequency: 10 # Duration in seconds to poll the crowdsec API
log_level: info # Valid choices are either of "debug","info","warning","error"
log_mode: file # Valid choices are "file" or "stdout"
log_file: /var/log/crowdsec-fastly-bouncer.log # Ignore if logging to stdout
```

# Helpers:

The bouncer has few builtin helper features:

## Auto config generator:

**Usage:**

```console
sudo crowdsec-fastly-bouncer -c <PATH_TO_BASE_CONFIG>\
    -g <FASTLY_TOKEN_1>,<FASTLY_TOKEN_2> 
```

This will print boilerplate config with sane defaults for the provided comma separted fastly tokens. Always review the generated config before proceeding further.

Crowdsec config is copied from the config at `PATH_TO_BASE_CONFIG`. 

##  Cleaner:

**Usage:**

```console
sudo crowdsec-fastly-bouncer -c <PATH_TO_BASE_CONFIG> -d
```

This deletes the fastly resources created by the bouncer. It only works if the configured service version is not locked. It is useful for quick iteration before deploying the new service. 
