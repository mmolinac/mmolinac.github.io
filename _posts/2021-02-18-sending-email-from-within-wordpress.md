---
layout: post
title: Sending email from within WordPress
author: Manuel Molina
about_author: "/us/manuel/readme.html"
status: publish
published: true
author_email: mmc@pocosmhz.org
tags:
- wordpress
- gmail
comments: true
---
In my recent this-thing-is-not-working spree, I've fortunately found [WP Mail SMTP](https://wordpress.org/plugins/wp-mail-smtp/). The thing that made me happy the most is that *it works*.

![Weird mascot logo](/content/images/2021-02-18-sending-email-from-within-wordpress/icon-256x256.png){: width="135" }

Let's say that last time I was in charge of managing a self-hosted WordPress installation you didn't have to bother with email being authenticated or digitally signed or whatever.

The plugin just allows you to configure one in a range of available mail delivery options for you to send comments notifications (or anything) from within WordPress.

The one I've chosen is to use <a href="https://developers.google.com/gmail/api">GMail API</a> and through <a href="https://wpmailsmtp.com/docs/how-to-set-up-the-gmail-mailer-in-wp-mail-smtp/#create-app">their detailed directions,</a> I managed to create OAUTH authentication credentials for my notification's sender account. Some concepts and steps are slightly different now than those in their documentation, but fair enough. Steps are something like this:</p>

1. Go to GMail's [Application Registration page](https://console.developers.google.com/flows/enableapi?apiid=gmail&pli=1) and log in with email account you wanted to create credentials for:

    ![Application to be registered](/content/images/2021-02-18-sending-email-from-within-wordpress/application_to_be_registered.png)

2. Once you click the button, you'll see a confirmation message:

    ![API is enabled](/content/images/2021-02-18-sending-email-from-within-wordpress/api_is_enabled.png)

3. On the next page, you'll have to answer a couple questions about the kind of credentials you need. Click the choices below and then hit the button:

    ![Add credentials to your project](/content/images/2021-02-18-sending-email-from-within-wordpress/kind_of_credentials_you_need.png)

4. You'll be presented with this screen:

    ![Set up OAuth consent screen](/content/images/2021-02-18-sending-email-from-within-wordpress/set_up_consent_screen.png)

5. Now you'll configure a consent screen that is usually shown to people, but not in this application. However, you'll have to click on "Set up consent screen" and then will get this screen in a separate tab or window:

    ![OAuth consent screen](/content/images/2021-02-18-sending-email-from-within-wordpress/oauth_consent_screen.png)

6. Select "Internal" and click on "Create". You'll get to the following screen, where you have to select the email account for this consent screen and other details. Omit the logo. When done, click "Save and continue" and get rid of this tab:
    ![Edit app registration](/content/images/2021-02-18-sending-email-from-within-wordpress/app_registration-1.png)
    ![App domain](/content/images/2021-02-18-sending-email-from-within-wordpress/app_domain.png)
    ![Developer contact information](/content/images/2021-02-18-sending-email-from-within-wordpress/developer_contact_information.png)

7. Once you click on "Save and continue", you will reach the "Configure scopes" dialog. Please leave it as is and "Save and continue" will take you to the end of this part:
    ![Edit app registration](/content/images/2021-02-18-sending-email-from-within-wordpress/scopes1.png)
    ![Restricted scopes](/content/images/2021-02-18-sending-email-from-within-wordpress/scopes2.png)

8. Now you will be setting Up Your OAuth Client ID:
    ![Credentials](/content/images/2021-02-18-sending-email-from-within-wordpress/create_oauth2_1.png)
    ![Create OAuth client ID](/content/images/2021-02-18-sending-email-from-within-wordpress/create_oauth2_button.png)

9. Click "Create OAuth client ID" after filling in the three fields in the dialog. You might be required to click "Refresh" button after some entries.

10. Then, you will be asked to either "Download" the credentials or "I'll do this later". Click the latter.

11. Select your recently created OAuth2 credentials here:

    ![OAuth 2.0 Client IDs](/content/images/2021-02-18-sending-email-from-within-wordpress/oauth2_credential_list_pencil.png)

    Click the "pencil icon" and you'll be able to jot down the Client ID and Client Secret values you'll need to configure and test the plugin:
    ![Client ID for Web application](/content/images/2021-02-18-sending-email-from-within-wordpress/your_client_id_and_secret.png)

12. You should be able to test the plugin by sending a test email from the WP Mail SMTP configuration settings window.
