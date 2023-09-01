
1.  Place a script inside the protected directory. PHP example script such as the following is good enough:
    
    ```
    <?php print_r($_SERVER) ?>
    ```
    
2.  Access the protected directory/script ([http://your.server/secure](http://your.server/secure)) from your browser, this should trigger a complete SSO cycle where you can authenticate on your IdP
3.  Upon successful authentication, the page should display all received attributes. Make sure you have non empty **Shib-Application-ID** amongst other attributes (if your IdP release them).
4.  Check your **shibd.log** to see if there are attributes received or errors encountered.

