## Setting Up the Development Environment

### About This Lab

**Objective:** To set up the virtual machine for use with the labs in this course

**Successful outcome:** You will have a one-node Hadoop cluster running inside a virtual machine


---
Steps
---------


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>1. Start the Virtual Machine</h2>

1\.  Start your ds virtual machine by selecting it and clicking "Play virtual machine", or connect to your AWS IP address.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>2. Start an HDP Cluster</h2>

1\. Open a Terminal window, either by clicking on the Terminal icon in the top toolbar, or by the **Application-\>System Tools** pull-down:

![setting-up-the-development-environment-1](https://user-images.githubusercontent.com/21102559/40942892-7547fb00-681d-11e8-99b2-280723939f9b.png)

2\.  Type the following:

```console
ssh sandbox
```

![setting-up-the-development-environment-2](https://user-images.githubusercontent.com/21102559/40942893-755c573a-681d-11e8-8d53-4e9290a10cf9.png)

3\. Enter: `ambari-server start`

![setting-up-the-development-environment-3](https://user-images.githubusercontent.com/21102559/40942894-756b1a22-681d-11e8-8e30-f0fcbb043d9a.png)


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>3. Verify the Cluster is Running</h2>

1\.  Open the Firefox Web browser in your VM. There is a shortcut to Firefox in the upper toolbar.

2\.  To launch Ambari, open a new tab and go to <a>http://sandbox:8080</a>. The login is admin and the password is `admin`

![setting-up-the-development-environment-4](https://user-images.githubusercontent.com/21102559/40942895-757beeb0-681d-11e8-9020-2adc468b2b84.png)

3\.  Notice the green/red health status of various Hadoop services. Click on the **HDFS** service. Find the pull-down menu **Quick Links** and choose **Namenode UI**.

![setting-up-the-development-environment-5](https://user-images.githubusercontent.com/21102559/40942896-758ae226-681d-11e8-9b5d-8945b1414201.png)

4\.  Click on the **DataNodes** tab and you should see one DataNode in your cluster

![setting-up-the-development-environment-6](https://user-images.githubusercontent.com/21102559/40942897-759bb556-681d-11e8-9f98-9b5f84a6cdd3.png)

5\.  Go back to Ambari and click **YARN -\> Quick Links -\> ResourceManager UI**

![setting-up-the-development-environment-7](https://user-images.githubusercontent.com/21102559/40942898-75aa9b52-681d-11e8-9cc0-8b41ba02401b.png)

### Result

You now have a virtual machine running on your local machine, and inside that virtual machine is a CentOS machine running an HDP cluster. Your environment is now setup and ready to complete the labs in the HDPANALYST: Data Science course.

You are finished!
