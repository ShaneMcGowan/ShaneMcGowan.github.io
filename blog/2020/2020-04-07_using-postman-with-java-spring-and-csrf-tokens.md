---
title: Using Postman with Java Spring and CSRF Tokens
published: true
description: Java Spring CSRF Tokens VS. Postman, Who will win? 
tags: springboot, Postman, CSRF, XSRF
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/zjzg40frzfgwke0u9rkb.png
---

Java Spring will return a `403 Forbidden` if any request besides a `GET` request is missing a Cross Site Request Forgery Token (CSRF Token) in the `X-XSRF-TOKEN` Header. Here is how to fix that issue when using Postman. I have seen people online suggest that you disable CSRF Tokens but please don't do that. That is silly. Those people are sily. 


# Creating an environment

We need to create an environment in which to store our `CSRF Token`
- In the top right of Postman, click the cog.
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/frzzfll8gvrz960inwuk.png)

- In the Pop Up window, Click `Add`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/3uj3nmamli2hl9d9w7me.png)



- Enter an appropriate `Environment Name`
- Enter `xsrf-token` in the first column.
- Click `Add` in the bottom right corner

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/57yyelhwtqqsk4pmhi9c.png)

- Ensure your environment is selected in the drop-down in the top right.


# Getting the CSRF Token

`GET` requests do not require a CSRF Token to be allowed through our `SpringSecurityConfig`
- Create a `GET` request
- Navigate to the `Tests` tab
- Enter `pm.environment.set("xsrf-token", decodeURIComponent(pm.cookies.get("XSRF-TOKEN")));`
 
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/zhxk5toyzly5o8pp3jsn.png)

Now when you call this endpoint in Postman, your CSRF Token will be stored in your environment variables.



# Using the CSRF Token
- Go to your request that requires the CSRF Token
- Navigate to the `Headers` tab
- Enter a key of `X-XSRF-TOKEN` and a value of `{{xsrf-token}}`, the `{{xsrf-token}}` value will be populated from our Environment we created earlier. 
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/tsyur8aa0q0sek13xepi.png)

Your request should now be from from CSRF errors

# Things to watch out for
- Be sure you have actually selected an Environment. I have forgotten to do this several times.
- Be sure to call the `GET` request again to populate the value in case it has become invalid or has expired.
- Have a nice day
