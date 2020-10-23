Red Hat Decision Manager Loan Demo
=============================================
This demo project will provide you with an example of creating, deploying and leveraging a set of rules
(defined in a decision table) in a Decision Server. You will be given examples of calling the rules as if
using it from an application through the REST API that is exposed by the server.

There are three options for you to install this project: local, OpenShift and Docker

## Software

The following software is required to run this demo:
- [JBoss EAP 7.2 zip](https://developers.redhat.com/download-manager/file/jboss-eap-7.3.0.zip)
- [Red Hat Decision Manager 7.8.0.GA Decision Central deployable for EAP 7](https://developers.redhat.com/download-manager/file/rhdm-7.7.0-decision-central-eap7-deployable.zip)
- [Red Hat Decision Manager: KIE-Server 7.8.0.GA deployable for EE8](https://developers.redhat.com/download-manager/file/rhdm-7.8.0-kie-server-ee8.zip)
- [7-Zip](http://www.7-zip.org/download.html) (Windows only): to overcome the Windows 260 character path length limit, we need 7-Zip to unzip the Decision Manager deployable.

## Install on your machine

2. 1. [Download and unzip.](https://github.com/jbossdemocentral/rhdm7-loan-demo/archive/master.zip) or [clone this repo](https://github.com/jbossdemocentral/rhdm7-loan-demo.git).

2. Add the product ZIP files to the installs directory.

3. Run the `init.sh` (Linux/macOS) or `init.ps1` (Windows) file.

4. Start Red Hat Decision Manager by running `./target/jboss-eap-7.1/bin/standalone.sh` (Linux/macOS) or `.\targer\jboss-eap-7.1\bin\standalone.ps1` (Windows).

5. Login to http://localhost:8080/decision-central

    ```
    - login for admin and analyst roles (u:dmAdmin / p:redhatdm1!)
    ```

## Running the demo

6. You should already see the loan-application. If not, you can import it from https://github.com/jbossdemocentral/rhdm7-loan-demo-repo. 

7. Click on the "loan-application" project to open the Loan Application Demo project.

8. The project has simple some data model (Loan, Applicant, Reservation), a single decision table (loan-application) which contains the loan approval rule set and a DMN model (reservation) that provides a recomendation based on the credit score.

9. Build and deploy version 1.2.0 of the project. Click on the "Build and Deploy" in the upper right corner.

10. Go to "Menu -> Deploy -> Execution Servers" repository to see the loan-application_1.2.0 KIE Container deployed on the Decision Server.

11. The Decision Server provides a Swagger UI that documents the full RESTful interface exposed by the server at: http://localhost:8080/kie-server/docs

12. In the Swagger UI:
   - navigate to "KIE Server and KIE containers"
   - expand the "GET" operation for resource "/server/containers"
   - click on "Try it out"
   - leave the parameters blank and click on "Execute"
   - when asked for credentials use: Username: kieserver, Password: kieserver1!
   - observe the response, which lists the KIE Containers deployed on the server and their status (STARTED, STOPPED).

12. We can use the Swagger UI to test our Loan Approval Decision Service. In the Swagger UI.

### Consuming the decision services via REST:

#### **Testing the decisions implemented with DMN:**

To test the rule execution on the kieserver:

1. Open the Swagger UI : `http://localhost:8080/kie-server/docs`
2. In the KIE Server Swagger UI, navigate to the "DMN Assets" section and locate the POST "/server/containers/{containerId}/dmn" item.
   1. ContainerID: loan-application_1.2.0
   2. Change the "Parameter content type" and the "response content type" to "application/json"
   3. Body:
```
{ 
  "model-namespace" : "https://kiegroup.org/dmn/_C159F266-40FF-49CB-B4D9-447DFACDDC16", 
  "model-name" : "recommendation", 
  "decision-name" : [ ], 
  "decision-id" : [ ], 
  "dmn-context" : {"Credit Score": 250}
}
```

#### **Testing the business rules implemented with the guided decision table:**

To test the rule execution on the kieserver:

1. Open Swagger : `http://localhost:8080/kie-server/docs`
2. To send a request to the Decision Service, navigate to "KIE session assets", and search for POST "/server/containers/instances/{containerId}". 	 
   3. Insert for container **id**: `loan-application_1.2.0`
   2. Change the "Parameter content-type" and "Response content type" to "application/json";
   3. And use the following payload for **body**:
```
{
	"lookup": "default-stateless-ksession",
	"commands": [
		{
			"insert": {
				"object": {
					"com.redhat.demos.dm.loan.model.Applicant": {
						"creditScore": 120,
						"name": "Jim Whitehurst"
					}
				},
				"out-identifier": "applicant"
			}
		},
		{
			"insert": {
				"object": {
					"com.redhat.demos.dm.loan.model.Loan": {
						"amount": 2500,
						"approved": false,
						"duration": 24,
						"interestRate": 1.5
					}
				},
				"out-identifier": "loan"
			}
		},
		{
			"fire-all-rules": {}
		},
		{
			"get-objects": {
				"out-identifier": "objects"
			}
		},
		{
			"dispose": {}
		}
	]
}
```

--------
### Expanding:

In both cases observe the results. The Loan Application rules have fired and determined that, based on the credit score of the application, and the amount of the loan, the loan can be approved. The `approved` attribute of the `Loan` has been set to `true`.

Exercise by changing the decision table as desired, change the version of the project, and redeploy a new version to a new KIE Container (allowing you to serve multiple versions of the same rule set at the same time on the same Decision Server). You can also build a new version of the project and use the Version Configuration tab of the container definition (in the Execution Servers screen) to manage the container using the UPGRADE button to pull the new version.

----------

# Extra installation **options**

**NOTE**: The installation options below are currently set to install Red Hat DM 7.7.

Option 2 - Run on OpenShift 
-----------------------------------------
This demo can be installed on Red Hat OpenShift in various ways. We'll explain the different options provided.

All installation options require an `oc` client installation that is connected to a running OpenShift instance. More information on OpenShift and how to setup a local OpenShift development environment based on the Red Hat Container Development Kit can be found [here](https://developers.redhat.com/products/cdk/overview/).

---
**NOTE**

The Red Hat Decision Manager 7 - Decision Central image requires a [Persistent Volume](https://docs.openshift.com/container-platform/3.7/architecture/additional_concepts/storage.html) which has both `ReadWriteOnce` (RWO) *and* `ReadWriteMany` (RWX) Access Types. If no PVs matching this description are available, deployment of that image will fail until a PV of that type is available.

---

### Automated installation, manual project import
This installation option will install the Decision Manager 7 and Decision Service in OpenShift using a single script, after which the demo project needs to be manually imported.

1. [Download and unzip.](https://github.com/jbossdemocentral/rhdm7-loan-demo/archive/master.zip) or [clone this repo](https://github.com/jbossdemocentral/rhdm7-loan-demo.git).

2. Run the `init-openshift.sh` (Linux/macOS) or `init-openshift.ps1` (Windows) file. This will create a new project and application in OpenShift.

3. Login to your OpenShift console. For a local OpenShift installation this is usually: https://{host}:8443/console

4. Open the project "RHDM7 Loan Demo". Open the "Overview". Wait until the 2 pods, "rhdm7-loan-rhdmcentr" and "rhdm7-loan-kieserver" have been deployed.

5. Open the "Applications -> Routes" screen. Click on the "Hostname" value next to "rhdm7-loan-rhdmcentr". This opens the Decision Central console.

6. Login to Decision Central:

    ```
    - login for admin and analyst roles (u:dmAdmin / p:redhatdm1!)
    ```
7. Click on "Design" to open the design perspective.

8. Click on "Import project". Enter the following as the repository URL: https://github.com/jbossdemocentral/rhdm7-loan-demo-repo.git , and click on "Import".

9. Select "loan-application" and click on the "Ok" button on the right-hand side of the screen.

10. The project has simple data model (Loan & Applicant) and single decision table (loan-application) which contains the loan approval rule set.

11. Build and deploy version 1.1.0 of the project. Click on the "Build and Deploy" in the upper right corner.

12. Go to "Menu -> Deploy -> Execution Servers" repository to see the loan-application_1.1.0 KIE Container deployed on the Decision Server.

13. The Decision Server provides a Swagger UI that documents the full RESTful interface exposed by the server at. To open the Swagger UI, go back to
the OpenShift console, and go to the "Applications - Routes" screen. Copy the "Hostname" value next to "rhdm7-loan-kieserver". Paste the URL in a browser tab
and add "/docs" to the URL. This will show the Swagger UI.

14. Follow instructions from above "Option 1- Install on your machine", starting at step 11.

### Scripted installation
This installation option will install the Decision Manager 7 and Decision Service in OpenShift using a the provided `provision.sh` script, which gives
the user a bit more control how to provision to OpenShift.

1. [Download and unzip.](https://github.com/jbossdemocentral/rhdm7-loan-demo/archive/master.zip) or [clone this repo](https://github.com/jbossdemocentral/rhdm7-loan-demo.git).

2. In the demo directory, go to `./support/openshift`. In that directory you will find a `provision.sh` script. (Windows support will be introduced at a later time).

3. Run `./provision.sh -h` to inspect the installation options.

4. To provision the demo, with the OpenShift ImageStreams in the project's namespace, run `./provision.sh setup rhdm7-loan --with-imagestreams`.

    ---
    **NOTE**

    The `--with-imagestreams` parameter installs the Decision Manager 7 image streams and templates into the project namespace instead of the `openshift` namespace (for which you need admin rights). If you already have the required image-streams and templates installed in your OpenShift environment in the `openshift` namespace, you can omit the `--with-imagestreams` from the setup command.

    ---

5. A second useful option is the `--pv-capacity` option, which allows you to set the capacity of the _Persistent Volume_ used by the Decision Central component. This is for example required when installing this demo in OpenShift Online, as the _Persistent Volume Claim_ needs to be set to `1Gi` instead of the default `512Mi`. So, to install this demo in OpenShift Online, you can use the following command: `./provision.sh setup rhdm7-loan --pv-capacity 1Gi --with-imagestreams`

6. To delete an already provisioned demo, run `./provision.sh delete rhdm7-loan`.

7. After provisioning, follow the instructions from above "Option 2 - Automated installation, manual project import", starting at step 2.

Option 3 - Run in Docker
-----------------------------------------
The following steps can be used to configure and run the demo in a container

1. [Download and unzip.](https://github.com/jbossdemocentral/rhdm7-loan-demo/archive/master.zip) or [clone this repo](https://github.com/jbossdemocentral/rhdm7-loan-demo.git).

2. Add the product ZIP files to installs directory.

3. Run the 'init-docker.sh' (Linux/macOS) or 'init-docker.ps1' (Windows) file.

4. Start the container: `docker run -it -p 8080:8080 -p 9990:9990 jbossdemocentral/rhdm7-loan-demo`

5. Follow instructions from above "Option 1- Install on your machine", starting at step 5 replacing *localhost* with *&lt;CONTAINER_HOST&gt;* when applicable.

Additional information can be found in the jbossdemocentral container [developer repository](https://github.com/jbossdemocentral/docker-developer)


Supporting Articles & Videos
----------------------------
- [Your first Decision Services on Red Hat Decision Manager 7](https://upload.wikimedia.org/wikipedia/commons/6/67/Learning_Curve_--_Coming_Soon_Placeholder.png)

- [Getting Started with Red Hat Decision Manager 7](https://upload.wikimedia.org/wikipedia/commons/6/67/Learning_Curve_--_Coming_Soon_Placeholder.png)


Released versions
-----------------
See the tagged releases for the following versions of the product:

- v1.3 Red Hat Decision Manager 7.8.0 GA
- v1.2 Red Hat Decision Manager 7.7.0 GA
- v1.1 Red Hat Decision Manager 7.5.0.GA
- v1.0 Red Hat Decision Manager 7.0.0.GA

![Red Hat Decision Manager 7](./docs/demo-images/rhdm7.png)

![Loan Project](./docs/demo-images/loan-prj-overview.png)

![Decision Table](./docs/demo-images/decision-table.png)

![Execution Server View](./docs/demo-images/execution-server-view.png)

![Swagger UI](./docs/demo-images/kie-server-swagger-ui.png)

![Swagger UI Containers Overview](./docs/demo-images/kie-server-swagger-ui-containers-overview.png)

![Swagger UI Rules Request](./docs/demo-images/kie-server-swagger-ui-rules-request.png)

![Swagger UI Rules Response](./docs/demo-images/kie-server-swagger-ui-rules-response.png)
