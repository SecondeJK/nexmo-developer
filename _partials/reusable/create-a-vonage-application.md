## Create a Vonage API Application

There are two alternative methods for creating a Messages and Dispatch application:

1. Using the Vonage CLI
2. Using the Dashboard

Each of these methods is described in the following sections.

### How to create a Messages and Dispatch application using the Vonage CLI

To create your application using the Vonage CLI, enter the following command into the shell:

``` shell
vonage apps:create "My Messages App" --messages_inbound_url=https://example.com/webhooks/inbound-message --messages_status_url=https://example.com/webhooks/message-status
```

This creates a Vonage API application with a messages [capability](/application/overview#capabilities), with the webhook URLs configured as specified, and generate a private key file `my_messages_app.key` and creates or updates the `vonage_app.json` file. 

### How to create a Messages and Dispatch application using the Dashboard

You can create Messages and Dispatch applications in the [Dashboard](https://dashboard.nexmo.com/applications).

To create your application using the Dashboard:

1. Under [Applications](https://dashboard.nexmo.com/applications) in the Dashboard, click the **Create a new application** button.

2. Under **Name**, enter the Application name. Choose a name for ease of future reference.

3. Click the button **Generate public and private key**. This will create a public/private key pair and the private key will be downloaded by your browser.

4. Under **Capabilities** select the **Messages** button.

5. In the **Inbound URL** box, enter the URL for your inbound message webhook, for example, `https://example.com/webhooks/inbound-message`.

6. In the **Status URL** box, enter the URL for your message status webhook, for example, `https://example.com/webhooks/message-status`.

7. Click the **Generate new application** button. You are now taken to the next step of the Create Application procedure where you can link a Vonage API number to the application, and link external accounts such as Facebook to this application.

8. If there is an external account you want to link this application to, click the **Linked external accounts** tab, and then click the corresponding **Link** button for the account you want to link to.

You have now created your application.

> **NOTE:** Before testing your application ensure that your webhooks are configured and your webhook server is running.
