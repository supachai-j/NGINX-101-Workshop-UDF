# Workshop Config Guide

## Accessing the labs

You will be accessing the labs using the F5 Unified Demo Framework (UDF).  Chrome is the preferred browser for access.

1. Open your browser, preferably Chrome and navigate to F5 UDF <https://udf.f5.com/courses>
   1. Select the **Non-F5 Users** option and log in using you UDF credentials.
      1. If this is your first time accessing UDF you should have received an email from noreply@registration.udf.f5.com with your credentials and asking you to reset your password.
      2. **IMPORTANT** You should retain these credentials, as they will be required to any access future courses you attend in the F5 UDF environment.
2. You should see the event(s) under **Happening now**. Find the NGINX 101 Workshop event and click on the **Launch** link at the far right.
3. Click the **Join** button.  Manage SSH Keys should not be required.
4. At the top you will see **Documentation** and **Deployment**.
   1. In the **Documentation** section you can elect to leave the session, see how long the session last and other documentation
   2. Click on the **Deployment** tab. Note that the **nginx-plus** VM will take a minute to provision and will be ready when you have a green arrow.
5. To access the nginx-plus VM, click the **Access** link and select **Web Shell** from the drop-down menu
6. **NOTE**: To paste into the web shell use **ctrl-shift-v**

## Install NGINX Plus using Ansible

7. You will be logged in as root, let's first modify the hostname and then we will use the substitute  user command (su) to use the **ubuntu** account for the remainder of the workshop.
   1. **hostnamectl set-hostname** *yourname*
   2. **su ubuntu**
8. Install our required dependencies for the workshop.
   1. **cd ~/NGINX-101-Workshop-UDF**
   2. **sudo sh 0-install-required-dependencies.sh**  (ignore git errors about local changes conflicting)
9. Verify that nginx is not running
   1. **curl localhost**
10. Take a look at our playbook that will install NGINX Plus. Note the host groups that will be targeted (loadbalancers). Also view the hosts file to see which host(s) will be updated.
    1. **cat hosts**
    2. **cat nginx_install_with_controller_agent.yml**
11. Note that we have cloned a github repository containing all of the files used in this workshop except the NGINX license certificates. We will need to move the license certificates to the correct folder for the scripts to work.
    1. **cp ~/nginx-repo.\* license/**
12. Run the Ansible playbook to install NGINX Plus with the controller agent. (use option 1 or 2)
    1. Full command:
         **ansible-playbook nginx_install_with_controller_agent.yml -b -i hosts**
    2. Scripted equivalent
         **sh 1-run-nginx_plus-playbook.sh**
    Note the output at the end of the command should look something like this: ok=15   changed=7    unreachable=0    failed=0 

## Open the Controller GUI / Install agent on VM

1. Open <https://udfcontroller.nginx.rocks> (User: admin@nginx.com / Nginx1122!)
2. Click the upper left NGINX logo, then click on **Infrastructure**. Note that an instance with your private ip will show in under a minute.
2. Wait for the new instance to appear and then feel free to change the alias by clicking the **Edit** that appears to the right when hover over your instance
3. Select **Graphs** on the side bar and then your instance.  You can also edit the name here by clicking on the gear icon next to your instance name.

### Configure a Gateway

The Gateway is for traffic aggregation for ingress into the network & nginx instances, which is a collection of server blocks.  A server block is similar to a virtual server on a BIG-IP.

4. Click on the NGINX logo and select **Services**.
5. Go to **Gateways** and select the **Create** button (upper right) or **Create Gateway** under *Quick Actions*.
6. Create a new gateway, call it ***yourname*-gw**
7. At the bottom, under **Environment**, select **production** and hit next.
8. In the **Placements**, select your NGINX instance, hit **Next**.
9. Under the hostnames, add
   1. http://www.nginx.rocks
   2. https://www.nginx.rocks
   3. Be sure to hit **Done** after adding each URI.
   4. In **Cert Reference** select the **nginx.rocks** certificate.
   5. In **Protocols** select **All* protocols.
10. Feel free to view the optional configuration options.
11. Publish the gateway by clicking on **Submit**. Wait on the **Gateways** screen until the gateway status is green.
    1. **NOTE:** You may have to enlarge the window or use the slide-bar to see the status.  You and also click on the circle next to the gateway name.

### Create Apps

Apps are customer specified collections of components/traffic that constitute an application or microservice. 

12. On the leftmost column hit **Apps** to show the *My Apps* menu and select **Overview** to show the current list of Apps.    Click on **Create App** under *Quick Actions* or just hit the **Create** button in the upper right.
13. Name your new app ***yourname*-app** and under **Environment** select the **production**.
14. Hit **Submit**.
15. You should be an taken to an *Overview* of the app you just created.  To see a list of all apps click on **Apps** on the side-bar. 

#### Add Components to your app

You need to create Components for your app. Components let you partition an App into smaller, self-contained pieces that are each responsible for a particular function of the overall application.  Components map backend workloads/code/microservices needing traffic routing and services for your app. Each Component contains an ingress definition that includes the fully-qualified domain names (FQDNs) and URIs of servers.  Components are basically a collection of location blocks (paths) and upstreams (server pools).

Components are created from the app overview page. If you are on the **My Apps** overvew page listing all the apps you can get to your app overview page by clicking on the app link or hovering over the app box which will expose the following icons to the far right, Delete (trash can), Edit (pen and paper), View (eyeball).  Selecting **View** will also take you to the overview page for that specific app.  Hit **Create Component** in the upper-right or select **Create Component** under *Quick Actions* on the inner side-bar.

16. In the **Configuration** section *Name* the first component **time1**.
17. In the **Gateways** section, select your gateway you configured earlier. **Because this is a shared environment where everyone is building the exact same app it is important you select only the gateway you created**
18. In the **URIs** section, select **Add URI** (link is on top right of screen) and enter the *URI*: **/time1**
19. *Important* hit **Done** in the URI box.
20. Click **Next** through the optional configuration items until you get to **Workload Groups**.  

**Workload Group**s, aka *upstream* in NGINX, are server pools that you can proxy request to.  They are commonly used for defining either a web server cluster for load balancing, or an app server cluster for URI routing/load balancing.

21. In **Workload Groups**, in *Workload Group Name* enter **time1**.

**Backend Workload URIs** are the servers that comprise the Workload Group, aka upstream server, (ie. pool member)

22. Add the backend workload URI: <http://time1.nginx.rocks:81>
23. Be sure to hit **Done** in the *URI* box after adding the URI.
24. Hit **Submit**.
25. Wait for the *green* **Status:** Configured next to time1 in the app Overview page or Components section.  If it spins for more than a couple of minutes just hit refresh to see if it finished.

#### Check your work

26. You can either go back the shell and run a curl or use the browser.
    1. Shell option: **curl localhost/time1** (or if you prefer to test the certificate: **curl --resolve www.nginx.rocks:443:127.0.0.1 https://www.nginx.rocks/time1**)
       1. This should return the timestamp page for API_SERVER: API1
    2. If you prefer a browser: back in the UDF web console Go to UDF Deployment page and under the nginx-plus VM, go to the **Access** drop-down and open the **Web Shell**. You will get the default NGINX 404 error because nothing is configured for the "/" URI.  In the URI bar, add **/time1** to the link.  This should retrieve timestamp for API1. Note that the UDF environment masks the upstream URL, but https still works.
27. View the changes made to /etc/nginx/nginx.conf on your host.
    1. **sudo nginx -T**
       1. You should see an **upstream time1_http...** section with your **server** 3.20.89.115:81 in it.
28. Add another component to your app named **time2**, with a URI of **/time2** using the server <http://time2.nginx.rocks:82> by repeating steps 24-35.
    1. Go back to the **Web Console**
       1. **curl localhost/time2**
          1. 1. This should return the timestamp page for API_SERVER: API**2**
       2. **sudo nginx -T**
          1. You should see a new upstream section for time2

#### Create a load balanced component

29. Add another component and name it **both**.
30. Select your gateway.
31. In the URI section add: **/both**
32. Click **Done**.
33. Click on **Workload Groups** and add a workload group called **both** with a uri of **/both**.
34. Add both of our backend workoad URIs:
    1. <http://time1.nginx.rocks:81>
    2. <http://time2.nginx.rocks:82>
35. Test the new configuration with a few curl commands on your SSH session:
    1. **curl localhost/time1**
    2. **curl localhost/time2**
    3. **curl localhost/both**
       1. Run it several times to see the round robin functionality
    4. **curl --insecure <https://localhost/both>**
       1. Showing that https is listening/working.

## Configure API Management

API management is an add-on module to NGINX controller.  In this section you will configure API definitions which will automatically add components to your app.

1. Navigate to **Services** > **APIs** and view the workload group named **ergast** (ergast.com:80).
2. Now select **API Definitions** and click **Create an API Definition**.
   1. Name you API **F1 *Yourname* API**
   2. Set the *Base path* to **/api/f1**
   3. Hit **Save**
3. Now in the **URIs** box click **Add a URI**
   1. In the **Path** enter **/seasons** and check the **Enable Documentation** box.
   2. Click the **Add Response** button, enter a response code 200 and enter a description OK
   3. In the box below add

```
{
         "season": "1978",
         "url": "https://en.wikipedia.org/wiki/1978_Formula_One_season"
}
```

4. This detail is added to the developer portal which is accessed at the end of this workshop. These responses are not required, but will be beneficial to the developers looking for how to access these published APIs in the future.
5. Add an additional URI **/drivers** and add the response:

```
{
          "driverId": "arundell",
          "url": "http://en.wikipedia.org/wiki/Peter_Arundell",
          "givenName": "Peter",
          "familyName": "Arundell",
          "dateOfBirth": "1933-11-08",
          "nationality": "British"
}
```

6. Click **Add A Published API**
      1. The *Published API Name* is **f1_*yourname*_api**
      2. The *Environment is **production**
      3. The *Application* is your application you created earlier.
      4. The *Gateway* is the gateway you created earlier.
      5. Click on **Save**
7. Scroll to the bottom to **Routes**, click **Add a route** and add routes to the resources we created.
   1. URI **/drivers (GET)** Workload Group **ergast**
   2. URI **/seasons (GET)** Workload Group **ergast**
8. **Publish** and wait for the success message.
9.  curl a few of these examples:

   ```
      curl -k http://localhost/api/f1/seasons
      curl -k http://localhost/api/f1/drivers
      curl -k http://localhost/api/f1/drivers.json
      curl -k http://localhost/api/f1/drivers/arnold.json
   ```

10. Edit your published API and add a rate limit policy.
   1. Select **Add a policy** in the **Policies** section
      1. Set your **Policy Type** to **Rate Limit** and key off of URI with a limit of 10 requests per minute.
      2. Go to the UDF Web Shell and run the following command numerous times:
         ```
         curl -i http://localhost/api/f1/drivers/arnold.json
         ```
         1. After approximately 10 request you should get the 429 "Too Many Requests" response
         2. **NOTE:** If you rate limit didn't seem to take effect make sure you **Published** the new changes.  If you did not you will see *Edited since last publish* in your API box on the API Definitions overview page.
11. Once you are done testing you can set your rate limit policy to a higher limit or remove it.

### Adding authentication to your APIs

12. Review the JWT **Identity Provider** under the **API Management** Section. A JWT has been pre-configured. It is in this GitHub repo and named **auth_jwt_key_file.jwk**.
13. Go to your API Definition, edit your published API and add a policy to require authentication using the JWT Identity Provider.  APIs will present their credentials using **Bearer Token**
14. Publish and test a curl commands (below). For the command using the authorization token you could also rung the following script.
    1. **sh 3-run-jwt-curl.sh**.

   ```
      curl -i http://localhost/api/f1/drivers/arnold.json
      curl -H "Authorization: Bearer   eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiIsImtpZCI6IjAwMDEifQ.eyJuYW1lIjoiUXVvdGF0aW9uIFN5c3RlbSIsInN1YiI6InF1b3RlcyIsImV4cCI6IjE2MDk0NTkxOTkiLCJpc3MiOiJNeSBBUEkgR2F0ZXdheSJ9.lJfCn7b_0mfKHKGk56Iu6CPGdJElG2UhFL64X47vu2M" localhost/api/f1/seasons
   ```

15.  The first command should return a *401 Authorization Required* response.  The second command should authenticate and return the page successfully.

#### Building your own JWT/JWKs tokens

1. If you want to check out the format of the JWT above, use <https://jwt.io> The secret is: **fantasticjwt**
2. To build your own:
   1. See the guide on NGINX.com to create your jwk: <https://docs.nginx.com/nginx/admin-guide/security-controls/configuring-jwt-authentication/>
   2. And  to build your jwt go to: <https://www.nginx.com/blog/authenticating-api-clients-jwt-nginx-plus>

## Extra credit

If you have time and are so inclined.

1. Add an alert for too many 500 errors.
2. Create a dashboard that you think might be useful in a NOC.
3. Access the Developer API Management Portal: <http://18.222.224.129:8090/docs>

***Feel free to browse around the GUI to see other functionality.***
