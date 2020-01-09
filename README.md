<h1 align="center">
Corvid by Wix DB Connectors: External Database Adapter for Google Cloud Firestore with Cloud Run or AppEngine
</h1>

This project is a Node.js based adapter for [Google Cloud Firestore](https://cloud.google.com/firestore/), that lets you integrate an external database with your [Corvid](https://www.wix.com/corvid) enabled Wix site.

You can use this project as a basis for deploying your own adapter to either [Google Cloud Run](https://cloud.google.com/run/) or [AppEngine](https://cloud.google.com/appengine/). This project is implementation of the external database collection SPI that has filtering, authorization and error handling support.

You can [learn more about working with external database collections in Corvid](https://support.wix.com/en/corvid-by-wix/external-database-collections-1023416).

[See our SPI documentation](https://www.wix.com/corvid/reference/external-database-collections.html).

# Getting started

This project assumes you have a Google Cloud Project with billing enabled. If you don't have one, follow this [free trial guide](https://cloud.google.com/free/).

## Enable Firestore in Native Mode
Go to https://console.cloud.google.com/firestore/welcome and select **Native Mode**

## Deploying to Cloud Run using Cloud Console

Follow the instructions in the Google Cloud Run [Quickstart Guide](https://cloud.google.com/run/docs/quickstarts/prebuilt-deploy).

In the **Create service form** use the following settings:

1. **Container Image**: Use **gcr.io/corvid-api/firestore-connector-node**.
2. **Deployment Platform**: Select **Cloud Run (fully managed)**.
3. **Location**: Select **us-east1** region.
4. **Authentication**: Select **Allow unauthenticated invocations**. This enables access to the connector from Corvid.
5. **Show Optional Settings**: Add an **Environment variable** named "SECRET_KEY" with your secret key as the value. This value is used for connecting to your Corvid enabled site.
6. Click **Create** to deploy the image to Google Cloud Run and wait for the deployment to finish.

Click the displayed URL link to test the deployed connector.
In the browser you should see the following, which indicates that the connector is running:
> {"message":"Missing request context"}

Copy the service URL. You will need it to [connect Firestore to your Corvid enabled Wix site](#connecting-firestore-to-corvid-site).

## Deploying to Google Cloud Run using the gcloud CLI

1. Install the [Google Cloud SDK](https://cloud.google.com/sdk/docs/quickstarts) for your operating system.
2. [Acquire new user credentials to use for Application Default Credentials](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login):

    ```bash
    gcloud auth application-default login
    ```

3. [Set your project as the default](https://cloud.google.com/sdk/gcloud/reference/config/set):

    ```bash
    gcloud config set project PROJECTID
    ```

4. [Deploy the Corvid connector container](https://cloud.google.com/sdk/gcloud/reference/run/deploy):

    ```bash
    gcloud run deploy --image gcr.io/corvid-api/firestore-connector-node --platform managed --region us-east1 --set-env-vars SECRET_KEY=[YOUR SECRET KEY]
    ```

    a. At the prompt: **Allow unauthenticated invocations to [firestore-connector-node] (Y/N)?** Choose "Y".

    b. When deployment is successful, the command line output will contain something similar to this:

    > Service [firestore-connector-node] revision [firestore-connector-node-00001-nep] has been deployed and is serving 100 percent of traffic at <https://firestore-connector-node-[autogenerated].run.app>

Copy the service URL. You will need it to [connect Firestore to your Corvid enabled Wix site](#connecting-firestore-to-corvid-site).

## Deploying to Google App Engine using gcloud CLI

1.  Check out the source code:

  ```bash
   git clone https://github.com/wix-incubator/firestore-connector-node.git
   cd firestore-connector-node
  ```

2. [Deploy the local code and/or configuration of your app to App Engine](https://cloud.google.com/sdk/gcloud/reference/app/deploy):

  ```bash
  gcloud app deploy
  ```

3. After deployment, access your service at `https://<project id>.appspot.com/`

## Connecting Firestore to your Corvid enabled Wix site

Follow the [instructions here](https://support.wix.com/en/article/corvid-adding-and-deleting-an-external-database-collection).

In the connection Dialog settings use the following:

* **Add an endpoint URL**: Use the **connector service URL** from steps above.
* **Configuration**: Use: {"secretKey":<**Your secret key from the deployment step**>}

You should now see Firestore collections in the Databases sections of the sidebar in your site.

# Developing the adapter

1. Check out the source code:

  ```bash
   git clone https://github.com/wix-private/firestore-connector-node.git
   cd firestore-connector-node
  ```

2. Create a [GCP Service account](https://cloud.google.com/iam/docs/service-accounts):

  ```bash
  gcloud iam service-accounts create corvid --description Corvid Dev account --display-name corvid-dev
  ```

3. Run the following command and copy its output, which you will use in step 4:

  ```bash
  gcloud iam service-accounts list
  ```

4. Generate the Service Account key and save it in gcp-sa-key.json:

  ```bash
  gcloud iam service-accounts keys create gcp-sa-key.json --iam-account <Service account name from Step 3>
  ```

5. Set an environment variable for GCP client libraries:

  ```bash
  export GOOGLE_APPLICATION_CREDENTIALS=$PWD/gcp-sa-key.json
  ```

6. Use the following command to start the Connector. It is a NodeJS express application:

  ```bash
  npm install

  npm start
  ```
