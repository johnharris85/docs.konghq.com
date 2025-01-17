---
name: API Security 
publisher: Imperva 

categories: 
  #- authentication
  - security
  #- traffic-control
  #- serverless
  #- analytics-monitoring
  #- transformations
  #- logging
  #- deployment
# Array format only; if your plugin applies to multiple categories,
# uncomment the most applicable category.

type: plugin # (required) String, one of:
  # plugin          | extensions of the core platform
  # integration     | extensions of the Kong Admin API

desc: Integrate Kong Gateway with Imperva API Security to discover, monitor, and protect APIs
description: |
  The Imperva API Security plugin connects Kong Gateway with the Imperva 
  API Security service, providing continuous discovery and monitoring of APIs 
  exposed by the API gateway. This enables security teams to protect business 
  applications and data against unauthorized access. 
  
  The plugin operates with a very low CPU and memory footprint, avoiding any 
  negative impact on the inline performance of the gateway or your applications.
  
  The Imperva API Security plugin captures API calls with request/response payloads 
  and sends them to the Imperva API Security service for inspection. API calls are 
  copied and streamed through Kong Gateway. You provide the API Security 
  receiver service endpoint though the plugin's configuration, so the API data is 
  kept under the control of the application owner.

# (required) extended description.
# Use YAML pipe notation for extended entries.
# EXAMPLE long text format (do not use this entry)
# description: |
#   Maintain an indentation of two (2) spaces after denoting a block with
#   YAML pipe notation.
#
#   Lorem Ipsum is simply dummy text of the printing and typesetting
#   industry. Lorem Ipsum has been the industry's standard dummy text ever
#   since the 1500s.

#support_url:
  # (Optional) A specific URL of your own for this extension.
  # Defaults to the url setting in your publisher profile.

#source_code:
  # (Optional) If your extension is open source, provide a link to your code.

#license_type:
  # (Optional) For open source, use the abbreviations in parentheses at:
  # https://opensource.org/licenses/alphabetical

#license_url:
  # (Optional) Link to your custom license.

#privacy_policy:
  # (Optional) If you have a custom privacy policy, place it here

#privacy_policy_url:
  # (Optional) Link to a remote privacy policy

#terms_of_service:
  # (Optional) Text describing your terms of service.

#terms_of_service_url:
  # (Optional) Link to your online TOS.

# COMPATIBILITY
# Uncomment at least one of 'community_edition' (Kong Gateway open-source) or
# 'enterprise_edition' (Kong Gateway Enterprise) and set `compatible: true`.

kong_version_compatibility: # required
  #community_edition: # optional
    #compatible: true
  enterprise_edition: # optional
    compatible:
      - 3.0.x
      - 3.2.x


#########################
# PLUGIN-ONLY SETTINGS below this line
# If your plugin is an extension of the core platform, ALL of the following
# lines must be completed.
# If NOT defined as a 'plugin' in line 32, delete all lines up to '# BEGIN MARKDOWN CONTENT'
params:
  name: imp-app-sec-connector
  service_id: true
  consumer_id: false
  route_id: true
  protocols:
    - name: http
    - name: https
  dbless_compatible: yes
  dbless_explanation: null
  yaml_examples: true
  k8s_examples: true
  konnect_examples: false
  manager_examples: false
  examples: true

  config: # Configuration settings for your plugin
    - name: destination_addr
      required: yes
      default:
      datatype: string
      encrypted: true
      value_in_examples: 10.0.0.1
      description: |
        Destination address of the API security receiver. 
        This can be an IP address or domain name, for example `logconsumer.myapisec.mydomain`.
    - name: destination_port
      required: no
      default: 8080
      datatype: number
      encrypted: false
      description: |
        Destination port of the API security receiver.
        Accepts one of the following values: `8080`, `80`, `8443` or `443`.      
    - name: connection_type
      required: no
      default: tcp
      datatype: string
      encrypted: false
      description: |
        The connection protocol to use. Accepts one of the following values: `tcp` or `http`.
    - name: method
      required: no
      default: POST
      datatype: string
      encrypted: false
      description: |
        The request method to use. Accepts one of the following values: `POST`, `PUT`, or `PATCH`.
    - name: max_body_size
      required: no
      default: 1048576
      datatype: number
      encrypted: false
      description: Maximum payload body size to capture, in bytes.
    - name: ssl
      required: no
      default: false
      datatype: boolean
      encrypted: false
      description: |
        Whether or not to use an TLS/SSL tunnel to send API traffic capture to the destination.
    - name: request_body_flag
      required: no
      default: true
      datatype: boolean
      encrypted: false
      description: |
        Determines whether to send the request body payload to the destination. 
        
        Set to `false` to disable. Use **only** for debugging purposes. 
        API security will not fully function without inspection of the request body payload.
    - name: response_body_flag
      required: no
      default: true
      datatype: boolean
      encrypted: false
      description: |
        Determines whether to send the response body payload to the destination. 
        
        Set to `false` to disable. Use **only** for debugging purposes. 
        API security will not fully function without inspection of the response body payload.
    - name: retry_count
      required: no
      default: 0
      datatype: number
      encrypted: false
      description: |
        Number of retries if sending the API call capture fails. No retry by default.
    - name: queue_size
      required: no
      default: 1
      datatype: number
      encrypted: false
      description: |
        Number of API logs to keep in the queue for retries. Default is 1, meaning no retries. 
        Set to a number larger than 1 to enable retries.
    - name: flush_timeout
      required: no
      default: 2
      datatype: number
      encrypted: false
      description: Number of seconds to wait before flushing the queue. 
    - name: timeout
      required: no
      default: 6000000
      datatype: number
      encrypted: false
      description: Number of milliseconds to keep a single connection open to the destination. 
  extra:
    # This is for additional remarks about your configuration.


###############################################################################
# END YAML DATA
# Beneath the next --- use Markdown (redcarpet flavor) and HTML formatting only.
#
# The remainder of this file is for free-form description, instruction, and
# reference matter.
# If you include headers, your headers MUST start at Level 2 (parsing to
# h2 tag in HTML). Heading Level 2 is represented by ## notation
# preceding the header text. Subsequent headings,
# if you choose to use them, must be properly nested (eg. heading level 2 may
# be followed by another heading level 2, or by heading level 3, but must NOT be
# followed by heading level 4)
###############################################################################
# BEGIN MARKDOWN CONTENT
---
## How it works
The plugin sends a copy of API call requests/responses to the Imperva API receiver. The receiver service destination address and port are specified as config parameters. Additional parameters are used to control how the API captures are sent. 

## How to install

{:.note}
> If you are using Kong's [Kubernetes Ingress Controller](https://github.com/Kong/kubernetes-ingress-controller), the installation is slightly different. Review the [docs for Kubernetes Ingress](/kubernetes-ingress-controller/latest/guides/setting-up-custom-plugins/).

The `.rock` file is a self-contained package that can be installed locally or from a remote server.

If the LuaRocks utility is installed in your system (this is likely the case if you used one of the official installation packages), you can install the `rock` in your LuaRocks tree, that is, the directory in which LuaRocks installs Lua modules.

### Install the Imperva Plugin

```shell
 luarocks install imp-appsec-connector
```

### Update your loaded plugins list
In your `kong.conf`, append `imp-appsec-connector` to the `plugins` field. Make sure the field is not commented out.

```yaml
plugins = bundled,imp-appsec-connector         # Comma-separated list of plugins this node
                                	       # should load. By default, only plugins
                                               # bundled in official distributions are
                                               # loaded via the `bundled` keyword.
```


### Restart Kong

After LuaRocks is installed, restart Kong before enabling the plugin:

```shell
kong restart
```

## Changelog

**imp-appsec-connector 0.1.1**

* Introduced the Imperva API Security plugin. 
This plugin is compatible with {{site.base_gateway}} 3.0.x and 3.2.x at release time.


